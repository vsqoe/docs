# eBPF-Documentation

## eBPF-Program

eBPF (extended Berkeley Packet Filter) programs are programs that are dynamically loaded into Linux kernel, allowing users to safely extend the kernel's functionality without modifying its core-code. Originally designed for packet-filtering, eBPF has evolved to support a wide range of use-cases, including performance-monitoring, security-enforcement, network-traffic-analysis, and system-observability. eBPF programs are executed in response to kernel events (e.g., network-packets, system-calls, tracepoints), providing a powerful mechanism for dynamic, real-time behavior analysis and control. They are designed to be safe and efficient, with strict validation mechanisms ensuring they don't compromise kernel stability.

We are using eBPF program to hook to TCP socket involving initiation of either active-TCP-connection or passive-TCP-connection. And, whenever the program activates it updates the congestion-window and slow-start-threshold value to the value is stored in the eBPF-map by the user-space-program, which is in our case done by the `httpsession.cpp` inside OpenLiteSpeed. This enables the transfer of congestion-window and slow-start-threshold to TCP upon falling back from QUIC.

## eBPF-Map

A special data-structure which enables communication between user-space-programs and eBPF-programs running in the kernel-space.

## Code description

```cpp
struct {
    __uint(type, BPF_MAP_TYPE_HASH);
    __type(key, __u32);
    __type(value, __u32);
    __uint(max_entries, 1);
} CwndMap SEC(".maps");

struct {
    __uint(type, BPF_MAP_TYPE_HASH);
    __type(key, __u32);
    __type(value, __u32);
    __uint(max_entries, 1);
} SSThreshMap SEC(".maps");

struct {
    __uint(type, BPF_MAP_TYPE_HASH);
    __type(key, __u32);
    __type(value, __u32);
    __uint(max_entries, 1);
} FallbackMap SEC(".maps");
```

> eBPF-map declarations: Currently map only support max entries of 1, this means there can only be one key-value pair.

```cpp
if (op != BPF_SOCK_OPS_PASSIVE_ESTABLISHED_CB && op != BPF_SOCK_OPS_ACTIVE_ESTABLISHED_CB) return 0;
```

First we check for the TCP-socket-operation being performed. If neither active nor passive-TCP-connection; skip (do not the updation of kernel-congestion-window and slow-start-threshold). Reasoning, is neither a new connection is established nor an old connection is reused, which means the fallback has not been initiated yet.

```cpp
userCwnd = bpf_map_lookup_elem(&CwndMap, &key);
if (!userCwnd) {
    // key not present in cwnd map
    ret = bpf_map_update_elem(&CwndMap, &key, &val, BPF_NOEXIST);
    if (ret != 0) bpf_printk("log: CWND value update to NOEXIST failed");
    return 0;
}
```

Test the presence of key inside the eBPF-map. If, the key is absent, this means the user-space-program never added the congestion-window; therefore, exit.

```cpp
// key present in cwnd map
if (!*userCwnd) return 0;
// *userCwnd = ceil(*userCwnd/MSS)
int newCwnd = (unsigned long long)(*userCwnd+MSS-1)/(unsigned long long)MSS;
bpf_printk("log: updating CWND to %d (%d MSS)", *userCwnd, newCwnd);
*userCwnd = newCwnd;
```

> Change the unit of congestion-window from bytes to MSS. This takes the ceil of `cwnd/MSS`.

```cpp
ret = bpf_setsockopt(skops, SOL_TCP, TCP_BPF_IW, userCwnd, sizeof(int));
if (ret != 0) bpf_printk("error %d: failed to set socket CWND to %d", ret, *userCwnd);
else bpf_printk("log: set socket CWND to %d", *userCwnd);
```

Update the value of the kernel-initial-congestion-window to the value extracted from the eBPF-map.

```cpp
userSSThresh = bpf_map_lookup_elem(&SSThreshMap, &key);
if (!userSSThresh) {
    // key not present in sshthresh map
    ret = bpf_map_update_elem(&SSThreshMap, &key, &val, BPF_NOEXIST);
    if (ret != 0) bpf_printk("log: SSThresh value update to NOEXIST failed");
    return 0;
}

// key present in ssthresh map
if (!*userSSThresh) return 0;
// *userSSThresh = ceil(*userSSThresh/MSS)
int newSSThresh = (unsigned long long)(*userSSThresh+MSS-1)/(unsigned long long)MSS;
bpf_printk("log: updating SSThresh to %d (%d MSS)", *userSSThresh, newSSThresh);
*userSSThresh = newSSThresh;

// setting the slow start threshold to be same as `*userSSThresh`
// Reference: https://netdevconf.info/2.2/papers/brakmo-tcpbpf-talk.pdf
ret = bpf_setsockopt(skops, SOL_TCP, TCP_BPF_SNDCWND_CLAMP, userSSThresh, sizeof(int));
if (ret != 0) bpf_printk("error %d: failed to set socket SSThresh to %d", ret, *userSSThresh);
else bpf_printk("log: set socket SSThresh to %d", *userSSThresh);
```

> The same is done for slow-start-threshold.
# Video-Streaming-Server

> _**NOTE**: If you are looking for the setup guide, please navigate to [OpenLiteSpeed-setup](/openlitespeed/)._

OpenLiteSpeed-web-server is being used as a video-streaming-server. It has support for `HTTP3/QUIC` out of the box. The server is hosted on Google Cloud Platform (GCP). 

## Patches

To support the updation of congestion-window-eBPF-map i.e., `CwndMap`; changes have been made to the openlitespeed-source-code:

1. Sets the default congestion-control-algorithm to `cubic`.
2. A shared-variable is maintained in the memory, this stores the current-state of the kernel-congestion-window set by the `lsquic` module specifically the `lsquic_cubic.c` file. As well as logs the current-state of kernel-congestion-window, which is picked up during the testing.
3. The shared-variable's value is used when a cubic-connection is reset. 
4. During each new HTTP-request, new logic is added to test the presence of the fallback-HTTP-header. If fallback-HTTP-header is present, then update the value of kernel-congestion-window inside the `CwndMap` using the value of the shared-variable. This logic is handled by the `httpsession.cpp` file.

> `lsquic_cubic.c` patch can be found here: [github.com/vsqoe/lsquic](https://github.com/litespeedtech/lsquic/compare/master...vsqoe:lsquic:master). <br>
> `httpsession.c` patch can be found here: [github.com/vsqoe/openlitespeed](https://github.com/litespeedtech/openlitespeed/compare/master...vsqoe:openlitespeed:master).

## Building the server

To build the server first run the [setup-script](https://github.com/vsqoe/server/blob/master/SETUP.sh), this sets up the project and the prerequisites ([cwndebpf](https://github.com/nos1dot618/cwndebpf.git), [libbpf](https://github.com/libbpf/libbpf.git), and [bpftool](https://github.com/libbpf/bpftool.git)). Afterwards, run the [build-script](https://github.com/vsqoe/server/blob/master/scripts/build_ols.sh), this builds and installs the lsws-service.
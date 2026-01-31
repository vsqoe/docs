# Client

## Components

1. [Dash-Player](https://dashjs.org/): A reference client implementation for the playback of MPEG DASH via Javascript and compliant browsers. Refer to [vsqoe/dash.js](https://github.com/vsqoe/dash.js), this patched-version adds a logging-callback which sends logs to a logging-server, additionally also changes the default streaming-URL to that of our custom-streaming-server-domain.
2. [Logging-Server](https://github.com/vsqoe/client/blob/master/logger.py): Receives the logs from the dash-player, writes to a CSV-file, which later is used for analysis.

## Getting Started

Follow the instructions mentioned in [vsqoe/client](https://github.com/vsqoe/client) for the client-side setup.

## Scripts Usage

1. [block_udp.sh](https://github.com/vsqoe/client/blob/master/scripts/block_udp.sh): Adds a new rule to IP-table for blocking all UDP connections on port 443.
2. [unblock_udp.sh](https://github.com/vsqoe/client/blob/master/scripts/unblock_udp.sh): Flushes all IP-table rules.
3. [run_udp.sh](https://github.com/vsqoe/client/blob/master/scripts/run_udp.sh): Kills all dash-instances, starts a new dash-instance, then starts chromium (in incognito mode, with QUIC enabled, and certificate-validation disabled) with url set to that of the dash-instance. After the experimentation kills both dash and chromium-instance. **User must click on "Load" themselves; for the streaming to begin.**
4. [run_tcp.sh](https://github.com/vsqoe/client/blob/master/scripts/run_tcp.sh): Similar to the UDP-script, with the addition of UDP-blocking and unblocking, before and after the experimentation.
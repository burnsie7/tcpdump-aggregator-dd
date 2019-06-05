# tcpdump-aggregator-dd

Code to aggregate tcpdump traffic and send to UDP log collector.

Forked from https://github.com/flarco/elk-tcpdump.git with a small amount of code added to determine which datadog service traffic is being sent to based off of IP ranges found here: https://ip-ranges.datadoghq.com/.

This allows one to capture a host's network traffic statistics:
  - Source IP/Host/Port to Target IP/Host/Port
  - Aggregate count of packets over time
  - Aggregate length of packets over time
  - Datadog service (agents/api/apm/logs/process)

# Instructions

The following packages are required:
- tcpdump
- host

The following python packages are required:
- requests

## Configure the datadog agent to listen for logs on UDP port and restart agent

```
logs:
  - type: udp
    port: 10518
    service: tcpdump
    source: agent-traffic
```

Then clone this repo:
```
git clone https://github.com/burnsie7/tcpdump-aggregator-dd.git
cd tcpdump-aggregator-dd
```

To start collecting tcpdump statistics, run the following on the host:
```
tcpdump -U -i eth0 -nn -tttt port not 10518 and port not 22 | python /path/to/tcpdump-aggregator-dd/tcpdump_aggregator_dd.py "127.0.0.1:10518"
 ```

To run as cron job at reboot:

```
@reboot /usr/sbin/tcpdump -U -i eth0 -nn -tttt port not 10518 and port not 22 | python /path/to/tcpdump-aggregator-dd/tcpdump_aggregator_dd.py "127.0.0.1:10518"
```

In the example above, the tcpdump aggregates will be sent over to host 127.0.0.1 / port 10518 via UDP every interval specified (in the script - default 10 sec).

The `source_HOST` / `target_HOST` fields are resolved by trial (using the linux `host` command), then cached. If no hostname is returned, the IP is stored instead.

# tcpdump-aggregator

Code to aggregate tcpdump traffic and send to UDP log collector.  This is built from https://github.com/flarco/elk-tcpdump.git with a small amount of code added to determine which datadog service traffic is being sent to.

This allows one to capture a host's network traffic statistics:
  - Source IP/Host/Port to Target IP/Host/Port
  - Aggregate count of packets over time
  - Aggregate length of packets over time

# Instructions

The following packages are required:
- tcpdump
- host

The following python packages are required:
- requests
- json

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
```shell
tcpdump -U -i eth0 -nn -tttt port not 10518 | python tcpdump-aggregator-dd/tcpdump_aggregator_dd.py "127.0.0.1:10518"`
 ```

To run as cron job at reboot:

`@reboot /usr/sbin/tcpdump -U -i eth0 -nn -tttt port not 10518 | python /path/to/tcpdump-aggregator-dd/tcpdump_aggregator_dd.py "127.0.0.1:10518"`

In the example above, the tcpdump aggregates will be sent over to host '192.168.2.3' / port 5141 via UDP every interval specified (in the script - default 20 sec).

Here is an example of the received data on 192.168.2.3:5141:
```shell
root@d54ea1457852:/tmp# netcat -ul 5141
{"source_IP": "172.17.0.3", "source_PORT": 22, "target_IP": "172.17.0.1", "target_PORT": 54686, "type": "TCP", "count": 1, "length": 212, "source_HOST": "172.17.0.3", "target_HOST": "172.17.0.1", "time": "2016-09-08 23:27:40.090202"}
{"source_IP": "172.17.0.1", "source_PORT": 54692, "target_IP": "172.17.0.3", "target_PORT": 22, "type": "TCP", "count": 24, "length": 0, "source_HOST": "NXDOMAIN", "target_HOST": "NXDOMAIN", "time": "2016-09-08 23:28:29.073292"}
{"source_IP": "172.17.0.1", "source_PORT": 54690, "target_IP": "172.17.0.3", "target_PORT": 22, "type": "TCP", "count": 1, "length": 52, "source_HOST": "172.17.0.1", "target_HOST": "172.17.0.3", "time": "2016-09-08 23:28:29.073292"}
{"source_IP": "172.17.0.3", "source_PORT": 22, "target_IP": "172.17.0.1", "target_PORT": 54690, "type": "TCP", "count": 1, "length": 0, "source_HOST": "172.17.0.3", "target_HOST": "172.17.0.1", "time": "2016-09-08 23:28:29.073292"}
{"source_IP": "172.17.0.3", "source_PORT": 22, "target_IP": "172.17.0.1", "target_PORT": 54692, "type": "TCP", "count": 24, "length": 3888, "source_HOST": "172.17.0.3", "target_HOST": "172.17.0.1", "time": "2016-09-08 23:28:29.073292"}
{"source_IP": "172.17.0.1", "source_PORT": 54686, "target_IP": "172.17.0.3", "target_PORT": 22, "type": "TCP", "count": 1, "length": 0, "source_HOST": "172.17.0.1", "target_HOST": "172.17.0.3", "time": "2016-09-08 23:28:29.073292"}
```

The `source_HOST` / `target_HOST` fields are resolved by trial (using the linux `host` command), then cached. If no hostname is returned, the IP is stored instead.

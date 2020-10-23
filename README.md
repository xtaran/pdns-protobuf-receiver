# PowerDNS protobuf receiver

![](https://github.com/dmachard/pdns-protobuf-receiver/workflows/Publish/badge.svg)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![PyPI - Python Version](https://img.shields.io/pypi/pyversions/pdns-protobuf-receiver)

The `pdns_protobuf_receiver` is a daemon in Python 3 that acts a protobuf server for PowerDNS's products. You can use it to collect DNS queries and responses and to log to syslog or a json remote tcp collector.

## Table of contents
* [Installation](#installation)
* [Execute receiver](#execute-receiver)
* [Startup options](#startup-options)
* [Output JSON format](#output-json-format)
* [PowerDNS configuration](#powerdns-configuration)
* [About](#about)

## Installation

### PyPI

From pypi, deploy the `pdns_protobuf_receiver` with the pip command.
Only Python3 is supported.

```python
pip install pdns-protobuf-receiver
```

After installation, you will have `pdns_protobuf_receiver` binary available

### Docker Hub

Pull the pdns-protobuf-receiver image from Docker Hub.

```bash
docker pull dmachard/pdns-protobuf-receiver:latest
```

Deploy the container

```bash
docker run -d -p 50001:50001 --name=pdns-pb01 dmachard/pdns-protobuf-receiver
```

Follow containers logs 

```bash
docker logs pdns-pb01 -f
```

## Execute receiver

The receiver is listening by default on the 0.0.0.0 interface and 50001 tcp port 

If you want to print DNS queries and responses to stdout in JSON format, then execute the `pdns_protobuf` receiver as below: 

```
# pdns_protobuf_receiver -v
2020-05-29 18:39:08,579 Start pdns protobuf receiver...
2020-05-29 18:39:08,580 Using selector: EpollSelector
```

If you want to resend protobuf message to your remote tcp collector
Start the pdns_protobuf receiver as below:

```
# pdns_protobuf_receiver -j 10.0.0.235:6000 -v
2020-05-29 18:39:08,579 Start pdns protobuf receiver...
2020-05-29 18:39:08,580 Using selector: EpollSelector
2020-05-29 18:39:08,580 Connecting to 10.0.0.235 6000
2020-05-29 18:39:08,585 Connected to 10.0.0.235 6000
```

## Startup options

Command line options are:

```
usage: -c [-h] [-l L] [-j J] [-v]

optional arguments:
  -h, --help  show this help message and exit
  -l L        listen protobuf dns message on tcp/ip address <ip:port>
  -j J        write JSON payload to tcp/ip address <ip:port>
  -v          verbose mode
```

## JSON log format

Each events generated by the `pdns_protbuf` receiver will have the following format:

```json
{
    "dns_message": "AUTH_QUERY",
    "socket_family": "IPv6",
    "socket protocol": "UDP",
    "from_address": "0.0.0.0",
    "to_address": "184.26.161.130",
    "query_time": "2020-05-29 13:46:23.322",
    "response_time": "1970-01-01 01:00:00.000",
    "latency": 0,
    "query_type": "A",
    "query_name": "a13-130.akagtm.org.",
    "return_code": "NOERROR",
    "bytes": 4
}
```

Keys description:
 - dns_message: PDNS message type (CLIENT_QUERY, CLIENT_RESPONSE, ...)
 - socket_family: IP protocol used (IPv4 or IPv6)
 - socket_protocol: transport protocol used (UDP or TCP)
 - from_address: the querier IP address
 - to_address: the destination IP address
 - query_time: time of query reception
 - response_time: time of response reception
 - latency: difference between query and response time
 - query_type: the query type (A, AAAA, NS, ...)
 - query_name: the query name
 - return_code: the response code sent back to the client (NXDOMAIN, NOERROR, ...)
 - bytes: size in bytes of the query or response

## PowerDNS configuration

You need to configure dnsdist or pdns-recursor to active remote logging.
 
### dnsdist

Configure the dnsdist `/etc/dnsdist/dnsdist.conf` and add the following lines
Set the newRemoteLogger function with the address of your pdns_protobuf_receiver
instance.

```
rl = newRemoteLogger("10.0.0.97:50001")
addAction(AllRule(),RemoteLogAction(rl))
addResponseAction(AllRule(),RemoteLogResponseAction(rl))
```

Restart dnsdist.

### pdns-recursor

Configure the powerdns recursor `/etc/pdns-recursor/recursor.conf` and add the following line

```
lua-config-file=/etc/pdns-recursor/recursor.lua
```

Create the LUA file `/etc/pdns-recursor/recursor.lua`
Set the protobufServer or outgoingProtobufServer functions with the address of your pdns_protobuf receiver instance.

```
protobufServer("10.0.0.97:50001", {logQueries=true,
                                   logResponses=true,
                                   exportTypes={'A', 'AAAA',
                                                'CNAME', 'MX', 
                                                'PTR', 'NS',
                                                'SPF', 'SRV',
                                                'TXT'}} )
outgoingProtobufServer("10.0.0.97:50001",  {logQueries=true,
                                            logResponses=true,
                                            exportTypes={'A', 'AAAA',
                                                         'CNAME', 'MX',
                                                         'PTR', 'NS',
                                                         'SPF', 'SRV',
                                                         'TXT'}})
```

Restart the recursor.

## About

| | |
| ------------- | ------------- |
| Author |  Denis Machard <d.machard@gmail.com> |
| PyPI |  https://pypi.org/project/pdns-protobuf-receiver/ |
| Github | https://github.com/dmachard/pdns-protobuf-receiver |
| DockerHub | https://hub.docker.com/r/dmachard/pdns-protobuf-receiver  |
| | |


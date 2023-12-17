### When not to use

- Encrypted traffic : HTTPS, SSH, IPSec
- Packet drops at the NIC - the network card is dropping packets
- Packet drops en route - when intermediate components like a firewall or network filter are dropping packets

## DNS

```A client sends a DNS query to a DNS server typically asking for an IP address in exchange for a host name. The DNS server either responds directly with information it possesses or it asks other DNS servers on behalf of the clients (recursive queries).
``````
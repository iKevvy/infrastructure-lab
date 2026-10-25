# Network Access Matrix

## Environment

| Component | Address | Purpose |
|---|---|---|
| CT104 | 10.77.105.104 | Ansible management controller |
| CT105 | 10.77.105.105 | Isolated Debian 13 LXC |
| vmbr1 | 10.77.105.0/24 | Private management subnet |

## Baseline Observations

- CT105 uses a private virtual `eth0` interface.
- CT105 has no IPv4 default gateway.
- CT105 has IPv4 forwarding enabled in the observed configuration.
- Docker is active; no Docker containers were running during the initial audit.
- Nginx listens on TCP/80.
- SSH listens on TCP/22.
- Existing firewall configuration includes Docker-managed tables.
- The experimental firewall was applied as a separate table.

## Experimental IPv4 Input Policy

| Source | Destination | Service | Policy | Evidence |
|---|---|---|---|---|
| CT104 | CT105 | SSH TCP/22 | ALLOW | Connection succeeded during enforcement |
| CT104 | CT105 | HTTP TCP/80 | ALLOW | HTTP 200 during enforcement |
| CT104 | CT105 | ICMP | ALLOW | Ping succeeded in an earlier live test |
| CT104 | CT105 | TCP/8088 | DROP | Timed out during enforcement |
| Other IPv4 sources | CT105 | Unmatched input | DROP | Configured rule; not separately tested |
| CT105 | Other networks | IPv4 routing | No default route | Routing configuration observation |

## Firewall Rule Order

1. Allow loopback input.
2. Allow established and related traffic.
3. Allow SSH from CT104.
4. Allow HTTP from CT104.
5. Allow ICMP from CT104.
6. Drop unmatched IPv4 input traffic.

## Scope Notes

The default-deny chain applies to IPv4 input traffic.

It does not represent a complete host-wide policy for IPv6, forwarding, NAT, or Docker-managed chains.

The absence of a default route is a routing property, not a substitute for firewall enforcement.

## Outcome

The tested access rules behaved as intended during temporary enforcement.

The experimental firewall was removed after testing.

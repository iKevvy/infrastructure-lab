# Project 002 — Validation Evidence

## Environment

| Property | Value |
|---|---|
| Management controller | CT104 — 10.77.105.104 |
| Managed target | CT105 — 10.77.105.105 |
| Operating system | Debian 13 LXC |
| Hypervisor | Proxmox VE |
| Automation | Ansible |
| Firewall | nftables |
| Recovery | systemd transient timer |

## 1. Firewall Syntax Validation

Ansible reported:

```text
Firewall syntax validation passed.

PLAY RECAP
10.77.105.105 : ok=3 changed=1 unreachable=0 failed=0
```

## 2. Baseline Connectivity

Before experimental firewall enforcement:

```text
SSH TCP/22: OPEN
HTTP TCP/80: HTTP 200
TCP/8088: OPEN
```

TCP/8088 was served by a temporary Python HTTP listener started through Ansible.

## 3. Enforcement Deployment

```text
PLAY RECAP
10.77.105.105 : ok=11 changed=2 unreachable=0 failed=0
```

## 4. Connectivity During Enforcement

```text
SSH TCP/22: OPEN
HTTP TCP/80: HTTP 200
TCP/8088: Connection timed out
```

The TCP/8088 timeout is meaningful because the same service was confirmed reachable before firewall activation.

## 5. Post-Rollback Verification

After the scheduled rollback window:

```text
Experimental table: No such file or directory
TCP/8088: OPEN
```

The timer had previously been observed active.

The transient service's execution history was not retained. Therefore, the exact execution sequence was not independently captured.

## 6. Listener Cleanup

After stopping the temporary listener through Ansible:

```text
TCP/8088: Connection refused
```

This confirms the test service was no longer accepting connections.

## Results Summary

| Check | Expected | Observed | Result |
|---|---|---|---|
| Firewall syntax | Valid | Valid | PASS |
| Ansible enforcement | No failures | 0 failed | PASS |
| SSH during enforcement | Allowed | Open | PASS |
| HTTP during enforcement | Allowed | HTTP 200 | PASS |
| TCP/8088 before enforcement | Reachable | Open | PASS |
| TCP/8088 during enforcement | Blocked | Timeout | PASS |
| Experimental table after rollback | Absent | Absent | PASS |
| TCP/8088 after rollback | Reachable | Open | PASS |
| Temporary listener cleanup | Closed | Refused | PASS |

## Limitations

- Tests were conducted from CT104.
- IPv6 was not tested.
- The firewall was temporary rather than persistent.
- Exact rollback service execution logs were not retained.
- The experiment does not establish comprehensive network isolation from every source.

## Conclusion

The experiment demonstrated the intended IPv4 input filtering behavior.

Authorized management traffic remained reachable while a previously accessible test service was blocked.

After the scheduled rollback window, the experimental firewall was absent and the test service was reachable again.

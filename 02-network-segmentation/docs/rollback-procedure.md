# Firewall Rollback and Recovery

## Purpose

A restrictive firewall can accidentally block the management connection used to deploy it.

This experiment used an independent systemd timer to remove the experimental policy automatically after a fixed interval.

## Experimental Scope

The rollback removes only:

`table ip ct105_lab`

It does not flush the complete nftables ruleset or intentionally modify Docker-managed tables.

## Rollback Script

Installed on CT105 through Ansible:

```sh
#!/bin/sh
/usr/sbin/nft delete table ip ct105_lab 2>/dev/null || true
```

The current script suppresses deletion errors. This is a limitation: a successful process exit does not by itself prove the table was removed.

## Deployment Sequence

1. Confirm the experimental table does not exist.
2. Validate the firewall configuration.
3. Verify the rollback script targets the correct table.
4. Schedule a transient systemd timer.
5. Confirm the timer is active.
6. Apply the experimental firewall.
7. Test management and unauthorized traffic.
8. Verify the table is absent after the rollback window.

## Manual Recovery

Run from CT104 using Ansible:

```bash
ansible all -i '10.77.105.105,' \
  -u labadmin -k -b -K \
  -m command \
  -a '/usr/sbin/nft delete table ip ct105_lab'
```

This command should only be used when the experimental table is present and manual recovery is needed.

## Observed Recovery

During the enforcement experiment:

- The rollback timer was observed active.
- The experimental firewall was active.
- TCP/8088 timed out during enforcement.
- After the rollback window, the firewall table was absent.
- TCP/8088 became reachable again.
- The transient systemd units were no longer listed.

The exact rollback service execution history was not captured because the transient unit was no longer available for inspection.

## Failure Scenarios

| Scenario | Expected response |
|---|---|
| SSH becomes inaccessible | Allow independent rollback timer to expire |
| Firewall syntax invalid | Reject configuration before activation |
| Experimental table already exists | Abort rather than overwrite unknown state |
| Rollback timer not active | Do not activate restrictive firewall |
| Rollback unit name conflicts | Inspect state; do not blindly retry |
| Table remains after deadline | Investigate and perform controlled recovery |

## Lessons Learned

- Schedule rollback before activating restrictive rules.
- Keep rollback independent from the management SSH session.
- Verify actual network state after recovery.
- Do not rely exclusively on systemd exit codes.
- Use unique timer identifiers for repeated experiments.
- Persist rollback logs for better auditability.

## Production Considerations

The current implementation is experimental.

A production design should include idempotent deployment, persistent logging, explicit failure handling, tested out-of-band recovery, and well-defined health checks.

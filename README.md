# Ansible Homelab Baseline

An Ansible playbook that configures two Ubuntu Server 24.04 lab servers the same way every time: baseline packages, groups and users, SSH hardening, and a least-privilege sudo rule. It was written for a VirtualBox homelab (see the homelab-docs repo).

## What the playbook does
- Installs baseline packages (htop, curl, net-tools)
- Ensures the helpdesk and webadmins groups and two lab users exist
- Applies an SSH hardening drop-in (key-only login, no root login, MaxAuthTries 3) and restarts ssh only when the file changes
- Installs a help desk sudo rule (is-active and restart for ssh only), validated with visudo before it is written

## Files
- baseline.yml: the playbook
- inventory.ini: the two lab servers (ksserver is the control node)
- ansible.cfg: points Ansible at the inventory

## Run it
Dry run:

    ansible-playbook baseline.yml --check --diff -K

Apply:

    ansible-playbook baseline.yml -K

## Results
| Run | ksserver | ksserver2 | What happened |
|---|---|---|---|
| 1. Dry run (--check --diff) | ok=6, changed=1 | ok=7, changed=3 | Showed what would change; nothing applied |
| 2. Real run | ok=6, changed=1 | ok=7, changed=3 | Installed net-tools on both, created the SSH hardening file on ksserver2, restarted ssh there |
| 3. Repeat run | ok=6, changed=0 | ok=6, changed=0 | Idempotent: nothing left to change |
| 4. After deleting the help desk sudo rule on ksserver2 | ok=6, changed=0 | ok=6, changed=1 | Drift detected and the rule restored |

## Notes
- The dry run showed the SSH hardening file was missing on ksserver2 before the first apply; the playbook created it.
- Sudo passwords are asked for with -K and are not stored anywhere in the repo.
- This is a small lab. A production setup would add roles, a vault for secrets, and testing.

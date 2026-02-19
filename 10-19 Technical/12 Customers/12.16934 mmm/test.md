---
tags:
  - Documentation/Testing
  - Documentation/Git
status: Rough Draft
last-modified: '2026-02-19T12:09:07-05:00'
rich-text: This is where you would put your content!!!
___mb_schema: /.mattrbld/schemas/documentation.json
---
\## Issue

When attempting to clone the template VM, it fails with something like:

\`\`\`bash

create full clone of drive scsi0 (iscsi-lvm:vm-105-disk-0) device-mapper: create ioctl on iscsi-vm--122--disk--0 failed: Device or resource busy

TASK ERROR: clone failed: lvcreate 'iscsi/vm-122-disk-0' error: Failed to activate new LV iscsi/vm-122-disk-0.

\`\`\`

\## Cause

This is because when a previous VM using that vmid was deleted, a bug occured and the lvm was not properly purged from the iscsi datastore. This can be verified by running the following on one of the Proxmox nodes, where \`$VMID\` is the vmid you are trying to use:

\`\`\`bash

sudo dmsetup table | grep $VMID

\`\`\`

\`\`\`bash

iscsi-vm--122--disk--0: 0 41943040 linear 8:16 9776930816

\`\`\`

\> Most likely there was some error in the past that led to the volume not being deactivated correctly, thus having the old device mapper entry being still present on the current target node. Please try \[@mir\](https://forum.proxmox.com/members/20028/) 's solution.

Quote from one of the Proxmox staff members about there likely being a bug that cause this in the past \[\[11.22.02 Proxmox Cant Clone VM#^3e18aa|Found Here\]\]

\## Solution

\*\*FIRST\*\* Verify the disk isnt in use by any other vms:

\`\`\`bash

sudo grep -R "vm-$VMID-disk-0" /etc/pve

\`\`\`

This should return empty

Then, the LV can be removed by just running:

\`\`\`bash

sudo dmsetup remove iscsi-vm--$VMID--disk--0

\`\`\`

Verify it got removed:

\`\`\`bash

sudo dmsetup table | grep $VMID

\`\`\`

\## Resources

\- https://forum.proxmox.com/threads/lvm-over-iscsi-volume-stuck-and-cannot-be-created.67740/

\- https://forum.proxmox.com/threads/cannot-migrate-device-mapper-create-ioctl-on-cluster-failed.12221/post-371000 ^3e18aa

\- https://forum.proxmox.com/threads/logical-volume-pve-vm-101-disk-0-in-use.120999/post-525819

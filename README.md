efi_migrate
=========

This Role can change a Server from BIOS to EFI Boot.  
I tested it with RHEL 8 and works so far.  

Please note: this is neither supported by RedHat or by myself !! USE WITH CAUTION !! I recommend making Snapshots beforehand!


Role Variables
--------------

| Variable | Purpose |
| -------- | ------- |
| `efi_migrate_reboot_msg` | You can Alter the Reboot Message on the Change to GPT. |
| `efi_migrate_reboot_timeout` | Timeout for the GPT-Change Reboot (Default: 120 seconds)

Dependencies
------------

You will need the `community.general`  and `ansible.posix` Collections for this role to work.


Example Playbook
----------------

    - hosts: servers
      roles:
         - efi_migrate

License
-------

GPT-v3.0-only


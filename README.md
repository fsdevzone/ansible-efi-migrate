<!-- DOCSIBLE START -->

# 📃 Role overview

## ansible-efi-migrate



Description: This Role can change a Server from BIOS to EFI Boot.  
I tested it with RHEL 8 and works so far.  
Please note: this is neither supported by RedHat or by myself !! USE WITH CAUTION !! I recommend making Snapshots beforehand!

The role will check if the system is running in BIOS mode and if there are mount points for /boot and /boot/efi.  
It will then check if the disk containing /boot/efi is using a GPT partition table and if not, it will convert the disk to GPT and create a BIOS compatibility partition if needed.  
Finally, it will install the EFI bootloader and migrate the system to EFI boot mode.








<details>
<summary><b>🧩 Argument Specifications in meta/argument_specs</b></summary>

#### Key: main
**Description**: This is the main entry point for the role.




#### Key: cleanup
**Description**: This task file handles the migration from BIOS to EFI bootloader.


  - **efi_migration_bios_disk_part_id**
    - **Required**: True
    - **Type**: str
    - **Default**: none
    - **Description**: The partition ID of the BIOS boot partition.
  
  
  

  - **efi_migration_boot_disk_part_id**
    - **Required**: True
    - **Type**: str
    - **Default**: none
    - **Description**: The partition ID of the EFI boot partition.
  
  
  

  - **efi_migration_boot_disk**
    - **Required**: True
    - **Type**: str
    - **Default**: none
    - **Description**: The boot disk where the EFI migration will take place.
  
  
  



</details>




### Defaults

**These are static variables with lower priority**

#### File: defaults/main.yml

| Var          | Type         | Value       |
|--------------|--------------|-------------|
| [efi_migrate_reboot_msg](defaults/main.yml#L4)   | str | `{{ efi_migration_boot_disk }} changed to GPT` |    
| [efi_migrate_reboot_timeout](defaults/main.yml#L5)   | int | `120` |    





### Tasks


#### File: tasks/cleanup.yml

| Name | Module | Has Conditions |
| ---- | ------ | -------------- |
| [Read device information](tasks/cleanup.yml#L31) | community.general.parted | False |
| [Remove Bios Compat Part](tasks/cleanup.yml#L22) | block | True |
| [Unmont EFI & Boot](tasks/cleanup.yml#L12) | ansible.posix.mount | False |
| [Remove Bios Compat Part](tasks/cleanup.yml#L22) | community.general.parted | True |
| [Read device information](tasks/cleanup.yml#L31) | community.general.parted | False |
| [Extend last partition to fill all available space](tasks/cleanup.yml#L37) | community.general.parted | False |
| [Mount BIOS & EFI again after cleanup](tasks/cleanup.yml#L50) | ansible.posix.mount | False |

#### File: tasks/efi.yml

| Name | Module | Has Conditions |
| ---- | ------ | -------------- |
| [Pakete installieren](tasks/efi.yml#L3) | ansible.builtin.dnf | False |
| [Set Partition Flags](tasks/efi.yml#L14) | ansible.builtin.shell | False |
| [Install EFI Bootloader](tasks/efi.yml#L25) | ansible.builtin.shell | False |
| [Config BIOS Bootloader](tasks/efi.yml#L29) | ansible.builtin.shell | False |
| [Config EFI Bootloader](tasks/efi.yml#L33) | ansible.builtin.shell | False |
| [Shutdown Server](tasks/efi.yml#L37) | community.general.shutdown | False |
| [Change Bios Settings](tasks/efi.yml#L40) | ansible.builtin.pause | False |

#### File: tasks/gpt.yml

| Name | Module | Has Conditions | Comments |
| ---- | ------ | -------------- | -------- |
| [Unmont EFI & Boot](tasks/gpt.yml#L3) | ansible.posix.mount | False |  |
| [Resize EFI Part](tasks/gpt.yml#L11) | ansible.builtin.shell | False |  |
| [Change Partition Table to GPT](tasks/gpt.yml#L20) | ansible.builtin.shell | False |  |
| [Read device information](tasks/gpt.yml#L27) | community.general.parted | False |  |
| [Create Compat Partition](tasks/gpt.yml#L37) | block | True | You'll need this in order to boot in BIOS mode from a GPT disk |
| [Create Compat Partition](tasks/gpt.yml#L37) | ansible.builtin.shell | False | You'll need this in order to boot in BIOS mode from a GPT disk |
| [Set Partition Flags](tasks/gpt.yml#L45) | ansible.builtin.shell | False |  |
| [Mount BIOS and EFI again](tasks/gpt.yml#L57) | ansible.posix.mount | False |  |
| [Install BIOS Bootloader](tasks/gpt.yml#L70) | ansible.builtin.shell | False |  |
| [Config BIOS Bootloader](tasks/gpt.yml#L74) | ansible.builtin.shell | False |  |
| [Reboot Server](tasks/gpt.yml#L78) | ansible.builtin.reboot | False |  |

#### File: tasks/main.yml

| Name | Module | Has Conditions |
| ---- | ------ | -------------- |
| [Migrate to EFI](tasks/main.yml#L5) | block | False |
| [Check EFI](tasks/main.yml#L8) | ansible.builtin.stat | False |
| [Check Readiness](tasks/main.yml#L13) | ansible.builtin.assert | False |
| [Get EFI Boot](tasks/main.yml#L20) | ansible.builtin.set_fact | False |
| [Get BIOS Boot Info](tasks/main.yml#L32) | ansible.builtin.set_fact | False |
| [Read device information](tasks/main.yml#L44) | community.general.parted | False |
| [Change Disk to GPT Table](tasks/main.yml#L50) | ansible.builtin.include_tasks | True |
| [Change to EFI Boot](tasks/main.yml#L54) | ansible.builtin.include_tasks | False |
| [Cleanup](tasks/main.yml#L57) | ansible.builtin.include_tasks | False |


## Task Flow Graphs



### Graph for cleanup.yml

```mermaid
flowchart TD
Start
classDef block stroke:#3498db,stroke-width:2px;
classDef task stroke:#4b76bb,stroke-width:2px;
classDef includeTasks stroke:#16a085,stroke-width:2px;
classDef importTasks stroke:#34495e,stroke-width:2px;
classDef includeRole stroke:#2980b9,stroke-width:2px;
classDef importRole stroke:#699ba7,stroke-width:2px;
classDef includeVars stroke:#8e44ad,stroke-width:2px;
classDef rescue stroke:#665352,stroke-width:2px;

  Start-->|Task| Read_device_information0[read device information]:::task
  Read_device_information0-->|Block Start| Remove_Bios_Compat_Part1_block_start_0[[remove bios compat part<br>When: **efi migration disk info partitions   selectattr <br>flags    contains    bios grub     length     1**]]:::block
  Remove_Bios_Compat_Part1_block_start_0-->|Task| Unmont_EFI___Boot0[unmont efi   boot]:::task
  Unmont_EFI___Boot0-->|Task| Remove_Bios_Compat_Part1[remove bios compat part<br>When: **efi migration disk info partitions   selectattr <br>flags    contains    bios grub     length     1**]:::task
  Remove_Bios_Compat_Part1-->|Task| Read_device_information2[read device information]:::task
  Read_device_information2-->|Task| Extend_last_partition_to_fill_all_available_space3[extend last partition to fill all available space]:::task
  Extend_last_partition_to_fill_all_available_space3-->|Task| Mount_BIOS___EFI_again_after_cleanup4[mount bios   efi again after cleanup]:::task
  Mount_BIOS___EFI_again_after_cleanup4-.->|End of Block| Remove_Bios_Compat_Part1_block_start_0
  Mount_BIOS___EFI_again_after_cleanup4-->End
```


### Graph for gpt.yml

```mermaid
flowchart TD
Start
classDef block stroke:#3498db,stroke-width:2px;
classDef task stroke:#4b76bb,stroke-width:2px;
classDef includeTasks stroke:#16a085,stroke-width:2px;
classDef importTasks stroke:#34495e,stroke-width:2px;
classDef includeRole stroke:#2980b9,stroke-width:2px;
classDef importRole stroke:#699ba7,stroke-width:2px;
classDef includeVars stroke:#8e44ad,stroke-width:2px;
classDef rescue stroke:#665352,stroke-width:2px;

  Start-->|Task| Unmont_EFI___Boot0[unmont efi   boot]:::task
  Unmont_EFI___Boot0-->|Task| Resize_EFI_Part1[resize efi part]:::task
  Resize_EFI_Part1-->|Task| Change_Partition_Table_to_GPT2[change partition table to gpt]:::task
  Change_Partition_Table_to_GPT2-->|Task| Read_device_information3[read device information]:::task
  Read_device_information3-->|Block Start| Create_Compat_Partition4_block_start_0[[create compat partition<br>When: **efi migration disk s info partitions   length    <br>4**]]:::block
  Create_Compat_Partition4_block_start_0-->|Task| Create_Compat_Partition0[create compat partition]:::task
  Create_Compat_Partition0-->|Task| Set_Partition_Flags1[set partition flags]:::task
  Set_Partition_Flags1-.->|End of Block| Create_Compat_Partition4_block_start_0
  Set_Partition_Flags1-->|Task| Mount_BIOS_and_EFI_again5[mount bios and efi again]:::task
  Mount_BIOS_and_EFI_again5-->|Task| Install_BIOS_Bootloader6[install bios bootloader]:::task
  Install_BIOS_Bootloader6-->|Task| Config_BIOS_Bootloader7[config bios bootloader]:::task
  Config_BIOS_Bootloader7-->|Task| Reboot_Server8[reboot server]:::task
  Reboot_Server8-->End
```


### Graph for efi.yml

```mermaid
flowchart TD
Start
classDef block stroke:#3498db,stroke-width:2px;
classDef task stroke:#4b76bb,stroke-width:2px;
classDef includeTasks stroke:#16a085,stroke-width:2px;
classDef importTasks stroke:#34495e,stroke-width:2px;
classDef includeRole stroke:#2980b9,stroke-width:2px;
classDef importRole stroke:#699ba7,stroke-width:2px;
classDef includeVars stroke:#8e44ad,stroke-width:2px;
classDef rescue stroke:#665352,stroke-width:2px;

  Start-->|Task| Pakete_installieren0[pakete installieren]:::task
  Pakete_installieren0-->|Task| Set_Partition_Flags1[set partition flags]:::task
  Set_Partition_Flags1-->|Task| Install_EFI_Bootloader2[install efi bootloader]:::task
  Install_EFI_Bootloader2-->|Task| Config_BIOS_Bootloader3[config bios bootloader]:::task
  Config_BIOS_Bootloader3-->|Task| Config_EFI_Bootloader4[config efi bootloader]:::task
  Config_EFI_Bootloader4-->|Task| Shutdown_Server5[shutdown server]:::task
  Shutdown_Server5-->|Task| Change_Bios_Settings6[change bios settings]:::task
  Change_Bios_Settings6-->End
```


### Graph for main.yml

```mermaid
flowchart TD
Start
classDef block stroke:#3498db,stroke-width:2px;
classDef task stroke:#4b76bb,stroke-width:2px;
classDef includeTasks stroke:#16a085,stroke-width:2px;
classDef importTasks stroke:#34495e,stroke-width:2px;
classDef includeRole stroke:#2980b9,stroke-width:2px;
classDef importRole stroke:#699ba7,stroke-width:2px;
classDef includeVars stroke:#8e44ad,stroke-width:2px;
classDef rescue stroke:#665352,stroke-width:2px;

  Start-->|Block Start| Migrate_to_EFI0_block_start_0[[migrate to efi]]:::block
  Migrate_to_EFI0_block_start_0-->|Task| Check_EFI0[check efi]:::task
  Check_EFI0-->|Task| Check_Readiness1[check readiness]:::task
  Check_Readiness1-->|Task| Get_EFI_Boot2[get efi boot]:::task
  Get_EFI_Boot2-->|Task| Get_BIOS_Boot_Info3[get bios boot info]:::task
  Get_BIOS_Boot_Info3-->|Task| Read_device_information4[read device information]:::task
  Read_device_information4-->|Include task| Change_Disk_to_GPT_Table_gpt_yml_5[change disk to gpt table<br>When: **efi migrate disk info disk table     gpt**<br>include_task: gpt yml]:::includeTasks
  Change_Disk_to_GPT_Table_gpt_yml_5-->|Include task| Change_to_EFI_Boot_efi_yml_6[change to efi boot<br>include_task: efi yml]:::includeTasks
  Change_to_EFI_Boot_efi_yml_6-->|Include task| Cleanup_cleanup_yml_7[cleanup<br>include_task: cleanup yml]:::includeTasks
  Cleanup_cleanup_yml_7-.->|End of Block| Migrate_to_EFI0_block_start_0
  Cleanup_cleanup_yml_7-->End
```





## Author Information
Fabian Seelbach

#### License

GPL-3.0-only

#### Minimum Ansible Version

2.12

#### Platforms

- **EL**: [8]


#### Dependencies

No dependencies specified.
<!-- DOCSIBLE END -->

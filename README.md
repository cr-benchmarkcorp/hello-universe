# Packer build for RHEL 7/8 on VMware
This build handles the creation and configuration of RHEL 7/8 based images in VMware.

The build handles the following components: 
- Creation of VMWare Template
- Hardening Based on CIS Framework

## Compatibility
Packer files are built with HCL2 and are meant for use with Packer 1.7 and above.

## Usage
Usage is as follows:  
Build File:

```hcl
build {
  name        = "rhel"
  description = <<EOF
//This build creates images for the following versions :
* RHEL 7
* RHEL 8
For the following builders :
* vsphere-iso
EOF

  source "source.vsphere-iso.base-rhel" {
    name = "7"
    http_content = {
      "/ks7.cfg" = templatefile("kickstart/ks7.pkrtpl.hcl", { rhelusername = var.rhelusername, rhelpassword = var.rhelpassword, rhel7_hostname = var.rhel7hostname })
    }
    boot_command = [
      "<tab><wait>",
      " ks=http://{{ .HTTPIP }}:{{ .HTTPPort }}/ks7.cfg<enter>"
    ]
    guest_os_type = "rhel7_64Guest"
    iso_paths     = ["[${var.vsphere_datastore}] isos/${var.isoname_rhel7}"]
    vm_name       = var.rhel7_template_name
  }

  source "source.vsphere-iso.base-rhel" {
    name = "8"
    http_content = {
      "/ks8.cfg" = templatefile("kickstart/ks8.pkrtpl.hcl", { rhelusername = var.rhelusername, rhelpassword = var.rhelpassword, rhel8_hostname = var.rhel8hostname })
    }
    boot_command = [
      "<tab><wait>",
      " ks=http://{{ .HTTPIP }}:{{ .HTTPPort }}/ks8.cfg<enter>"
    ]
    guest_os_type = "rhel8_64Guest"
    iso_paths     = ["[${var.vsphere_datastore}] isos/${var.isoname_rhel8}"]
    vm_name       = var.rhel8_template_name
  }
}
```
Source File
```hcl
source "vsphere-iso" "base-rhel" {
  CPUs                 = var.vm_cpu_num
  RAM                  = var.vm_mem_size
  RAM_reserve_all      = false
  disk_controller_type = ["pvscsi"]
  cluster              = var.vsphere_cluster
  datastore            = var.vsphere_datastore
  folder               = var.vsphere_folder
  convert_to_template  = var.create_template

  network_adapters {
    network      = var.vsphere_network
    network_card = "vmxnet3"
  }

  storage {
    disk_size             = var.vm_disk_size
    disk_thin_provisioned = true
  }

  ssh_password = var.ssh_password
  ssh_username = var.ssh_username
  notes        = "Built via Packer-${timeadd(timestamp(), "-7h")}"

  vcenter_server      = var.vsphere_server
  username            = var.vsphere_user
  password            = var.vsphere_password
  insecure_connection = "true"
}
```
<!-- Command Section -->
Then perform the following commands on the root folder:
- `packer validate .` to lint the template
- `packer build .` to start the template creation

<!-- BEGINNING OF Packer DOCS -->
## Common Variables & Inputs
| Name | Description | Type | Default | Sensitive | Assigned by |
|------|-------------|------|---------|:--------:|----------|
| vm-cpu-num | Number of VCPU used to build the template.| `string` | `2` | no | variables.common.pkr.hcl |
| vm-mem-size | The Amount of Memory to builf the template. | `string` | `4096` | no | variables.common.pkr.hcl |
| vm-disk-size | The Default Disk size when building the template. | `string` | `86016` | no | variables.common.pkr.hcl |
| vsphere-server |  The DNS or IP address of the VSphere Server. | `string` | `" "` | no | runtime.pkrvars.hcl |
| vsphere-cluster | The VSphere Cluster to Launch the template on. | `string` | `" "` | no | runtime.pkrvars.hcl |
| vsphere-folder | The Folder in VCenter to store the template in. | `string` | `" "` | no | runtime.pkrvars.hcl |
| vsphere-network | The VLAN Network to launch the VM on to create the template. | `string` | `" "` | no | runtime.pkrvars.hcl |
| vsphere-datastore | The DataStore to store the VMware Template on. | `string` | `" "` | no | runtime.pkrvars.hcl |
| vsphere-user | The user name to use to authenticate against the vCenter Server. | `string` | `" "` | yes | GitHub Secrets->Action Env |
| vsphere-password | The Password for authenticating against VSphere. | `string` | `" "` | yes | GitHub Secrets->Action Env |
| ssh-password | The Password used by the provisioner to connect to the VM. | `string` | `" "`| yes | GitHub Secrets->Action Env |
| ssh-username | The Username used by SSH to connect to the VM. | `string` | `" "` | yes | GitHub Secrets->Action Env |
| rhelusername | The Username used by RHEL Subscription Manager. | `string` | `" "` | no | runtime.pkrvars.hcl |
| rhelpassword | The Password used by RHEL Subscription Manager. | `string` | `" "` | yes | GitHub Secrets->Action Env |
| create_template | The variable used by the vSphere builder for template creation. | `string` | `null` | no | runtime.pkrvars.hcl |


## Version Specific
| Name | Description | Type | Default | Sensitive | Assigned by |
|------|-------------|------|---------|:--------:|----------|
| isoname-rhel7 | OS Major Version of RHEL 7. | `string` | `rhel-server-7.9-x86_64-dvd.iso` | no | variables.rhel7.pkr.hcl |
| isoname-rhel8 | OS Major Version of RHEL 8. | `string` | `rhel-8.4-x86_64-dvd.iso` | no | variables.rhel8.pkr.hcl |
| rhel7_template_name | Name Packer should use to create the VM template (Rhel8). | `string` | `" "` | no | runtime.pkrvars.hcl |
| rhel8_template_name | Name Packer should use to create the VM template (Rhel7). | `string` | `" "` | no | runtime.pkrvars.hcl |
| rhel7hostname | Hostname that Rhel7 Kickstart file uses for OS hostname. | `string` | `" "` | no | runtime.pkrvars.hcl |
| rhel8hostname | Hostname that Rhel7 Kickstart file uses for OS hostname. | `string` | `" "` | no | runtime.pkrvars.hcl |


## Requirements
Before this module can be used on a project, you must ensure that the following pre-requisites are fulfilled:

1. Packer and mkisofs are [installed](#software-dependencies) on the machine where apcker is executed.

2. The Service Account you execute the build with has the right [permissions](#configure-a-service-account).
### Software Dependencies
#### Packer
- [Packer](https://www.packer.io/downloads) 1.7.4
### VCenter Service Account Requirements 
In order to execute this build you must have a Service Account using a Role with the
following VCenter permissions:

System.Anonymous
System.View
System.Read
Global.CancelTask
Folder.Create
Folder.Delete
Folder.Rename
Folder.Move
Datastore.Browse
Datastore.DeleteFile
Datastore.FileManagement
Datastore.AllocateSpace
Datastore.UpdateVirtualMachineFiles
Datastore.UpdateVirtualMachineMetadata
Network.Assign
Host.Config.SystemManagement
Host.Config.AdvancedConfig
VirtualMachine.Inventory.Create
VirtualMachine.Inventory.CreateFromExisting
VirtualMachine.Inventory.Register
VirtualMachine.Inventory.Delete
VirtualMachine.Inventory.Unregister
VirtualMachine.Inventory.Move
VirtualMachine.Interact.PowerOn
VirtualMachine.Interact.PowerOff
VirtualMachine.Interact.Suspend
VirtualMachine.Interact.Reset
VirtualMachine.Interact.AnswerQuestion
VirtualMachine.Interact.ConsoleInteract
VirtualMachine.Interact.DeviceConnection
VirtualMachine.Interact.SetCDMedia
VirtualMachine.Interact.SetFloppyMedia
VirtualMachine.Interact.ToolsInstall
VirtualMachine.Interact.GuestControl
VirtualMachine.Interact.Backup
VirtualMachine.Interact.PutUsbScanCodes
VirtualMachine.Config.Rename
VirtualMachine.Config.AddExistingDisk
VirtualMachine.Config.AddNewDisk
VirtualMachine.Config.RemoveDisk
VirtualMachine.Config.CPUCount
VirtualMachine.Config.Memory
VirtualMachine.Config.AddRemoveDevice
VirtualMachine.Config.EditDevice
VirtualMachine.Config.Settings
VirtualMachine.Config.Resource
VirtualMachine.Config.UpgradeVirtualHardware
VirtualMachine.Config.ResetGuestInfo
VirtualMachine.Config.AdvancedConfig
VirtualMachine.Config.DiskLease
VirtualMachine.Config.DiskExtend
VirtualMachine.Config.ReloadFromPath
VirtualMachine.State.CreateSnapshot
VirtualMachine.State.RevertToSnapshot
VirtualMachine.State.RemoveSnapshot
VirtualMachine.State.RenameSnapshot
VirtualMachine.Provisioning.Customize
VirtualMachine.Provisioning.Clone
VirtualMachine.Provisioning.PromoteDisks
VirtualMachine.Provisioning.CreateTemplateFromVM
VirtualMachine.Provisioning.DeployTemplate
VirtualMachine.Provisioning.CloneTemplate
VirtualMachine.Provisioning.MarkAsTemplate
VirtualMachine.Provisioning.MarkAsVM
VirtualMachine.Provisioning.ReadCustSpecs
VirtualMachine.Provisioning.ModifyCustSpecs
VirtualMachine.Provisioning.DiskRandomAccess
VirtualMachine.Provisioning.DiskRandomRead
VirtualMachine.Provisioning.GetVmFiles
VirtualMachine.Provisioning.PutVmFiles
Resource.AssignVMToPool
Resource.AssignVAppToPool
Resource.ApplyRecommendation
Resource.CreatePool
Resource.RenamePool
Resource.EditPool
Resource.MovePool
Resource.DeletePool
Resource.HotMigrate
Resource.ColdMigrate
Resource.QueryVMotion
ScheduledTask.Create
ScheduledTask.Delete
ScheduledTask.Run
ScheduledTask.Edit
StorageProfile.View


# Using the Kickstart file
Kickstart helps to automate the installation process, without need for any intervention from the user. The Kickstart file contains answers to all questions normally asked by the installation program.

Storing the Kickstart file in git allows it to fall under version control and makes it subject to approvals if required.

This Kickstart file was obtained from NIST as part of the National Checklist Program and is part of a SCAP security guide authored by Red Hat.

[NIST National Checklist for Red Hat Enterprise Linux 7.x content](https://nvd.nist.gov/ncp/checklist/811)

## Usage
The Kickstart file is mounted as a CD-ROM Drive during the Packer build and provides the answers to proceed with an unattended RHEL installation.  The file is commented to provide clarity around the sections and chosen values.

## Key Considerations
### Passwords

#### NEVER SET A PLAINTEXT PASSWORD IN THE KICKSTART FILE

Encrpyted passwords are set in the Kickstart file for:

* root
* bootloader
* a user that can login and escalate prvileges (eg: admin)

These passwords are set with the --iscrypted flag within the kickstart file. To create a new encrypted password, sign into an existing RHEL server and use the following command to generate a hashed password.

### Generating new encrypted passwords

Replace "My Password" and "My Salt" with the values you wish to use.  Leave $6$ as it sets the algorithm to SHA-512.

python -c 'import crypt; print(crypt.crypt("My Password", "$6$My Salt"))'

The output of the file will be the encrypted password.  This value will follow the pattern *$algorithm$salt$hashed_password$*

For example, hashing the plaintext password *Luke$kywalk3r* with SHA-512 and the salt *R2D2* will produce the following:

```python
python -c 'import crypt; print(crypt.crypt("Luke$kywalk3r", "$6$R2D2"))'

$6$R2D2$HbGlv21UkpSaPMi/S.wPQKGCAWChClqfjvSOlpfnX2R67OGkSUf7LtyQn7AX4b.UyRZ5x/bsSxrYfGLuwXO8A/
```

### Hardening

OS hardening occurs on RHEL servers against the CIS profile via the OpenSCAP installer add-on (%addon org_fedora_oscap), integrating the capabilities of OpenSCAP ecosystem utilities directly into the operating system installation process.


```
%addon org_fedora_oscap
        content-type = scap-security-guide
        profile = xccdf_org.ssgproject.content_profile_cis
%end
```

This closes a typical gap where an operating system is installed, but not configured with expected policies. The security policy is fed into the installation process ensuring systems are compliant from the very first boot. After the first boot, audit results are stored in the /root directory.


## Version Specific File Names
| Name           | Description                      | Runtime (emphemeral)           |
|----------------|----------------------------------|--------------------------------|
| ks7.pkrtpl.hcl | Kickstart HCL template for RHEL7 | ks7.cfg                        |
| ks8.pkrtpl.hcl | Kickstart HCL template for RHEL7 | ks8.cfg                        |            


## HCL Kickstart Templating
Normally kickstart files would end in <filename>.cfg.   As you can see from above, the kick start files are HCL templates in the format <filename>.pkrtpl.hcl.    Packer will automatically use these files and substitute build-specific and sensitive values in during the build.

### Template Format
Inside the <filename>.pkrtpl.hcl files you will notice the occurance of ${variable} in the configuration.

Eg: 
```
subscription-manager register --auto-attach --username=${rhelusername} --password=${rhelpassword}
```
The templated variables ${rhelusername} and ${rhelpassword} are not stored in the kick start template (and hence not in your Git repo).
During build time, Hashicorp's Packer will find and replace these values using the following substitution line in the build HCL:
```
"/ks7.cfg" = templatefile("kickstart/ks7.pkrtpl.hcl", { rhelusername = var.rhelusername, rhelpassword = var.rhelpassword, rhel7_hostname = var.rhel7hostname })
```

The templated variables ${rhelusername} and ${rhelpassword} are then replaced by the Packer variables, and the resulting kickstart file in this example will be named "ks7.cfg".

## Helpful Links

[Pykickstart's Documentation](https://pykickstart.readthedocs.io/en/latest/)

[Red Hat RHEL 7 Kickstart Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-howto)

[Red Hat RHEL 7 Kickstart Syntax Reference](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-syntax)

[Red Hat RHEL 8 Kickstart Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/performing_an_advanced_rhel_installation/performing_an_automated_installation_using_kickstart)

[Red Hat RHEL 8 Kickstart Syntax Reference](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/performing_an_advanced_rhel_installation/kickstart-commands-and-options-reference_installing-rhel-as-an-experienced-user)

[OSCAP Anaconda Addon](https://www.open-scap.org/tools/oscap-anaconda-addon/)

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

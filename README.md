# Installation Notes
Installation notes for setting up Windows VM Lab in VirtualBox.

## Summary
1. `packer`: Use packer to build the Windows artifacts.
2. `vagrant`: Use vagrant to create the VMs
3. `metasploitable3`: Use packer and vagrant to setup the metasploitable3 VM.

## System Requirements
#### Total
* VT-x/AMD-V supported processor (recommended)
* 65 GB storage space (61440 MB)
	* Each VM will be provisioned with this size, but will have their disks resized after provisioning.
* 8 GB ram

#### RAM and Storage Space
| Component       | RAM (GB) | Storage (GB) |
| --------------- | -------- | ------------ |
| Base OS         | 1        | N/A          |
| Kali VM         | 2        | 30           |
| ADFS2 - dc      | 3/4      | 65           |
| ADFS2 - adfs2   | 3/4      | 65           |
| ADFS2 - web     | 1        | 65           |
| ADFS2 - ps      | 3/4      | 65           |
| ADFS2 - ts      | 3/4      | 65           |
| Metasploitable3 | 1        | 65           |

## Requirements
* [Packer](https://www.packer.io/intro/getting-started/setup.html)
* [Vagrant](https://www.vagrantup.com/docs/installation/)
* [Vagrant Reload Plugin](https://github.com/aidanns/vagrant-reload#installation)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

## Packer
Get the packer-windows repository
```bash
git clone http://github.com/diaclectic/packer-windows
cd packer-windows
```

Then build the Windows 2012 R2 artifact
```bash
cmd.exe /c packer build windows_2012_r2.json
```

## Vagrant
Get the adfs2 repository
```bash
git clone http://github.com/diaclectic/adfs2 metasploitlab
cd metasploitlab
```

Then create the VMs and fill in some printers via powershell scripts
```
vagrant up dc --provider virtualbox
vagrant up adfs2 --provider virtualbox
vagrant up web --provider virtualbox
vagrant up win7 --provider virtualbox
vagrant up nd --provider virtualbox
powershell -file c:\vagrant\scripts\create-queues.ps1
vagrant up nd --provider virtualbox
powershell -file c:\vagrant\scripts\import-ep.ps1
```

To resize the hard disks:
https://gist.github.com/christopher-hopper/9755310
https://tuhrig.de/resizing-vagrant-box-disk-space/

#### Test Single Sign On
Single Sign On should work out of the box with the provisioning scripts.
But you can install the JBoss Negotiation Toolkit for further tests

1. Go to the `ep` box and open the Ocon Shell
2. `jb`
3. `install-jboss-negotiation-toolkit.pl`
4. Go to the `win7` box and login as `mike.hammer`
5. Open IE with URL [http://ep:8080/jboss-negotiation-toolkit/](http://ep:8080/jboss-negotiation-toolkit/)

## Normal Use
After setting up all boxes, you simply can start and stop the boxes, but the
Domain Controller should be started first and stopped last.

```bash
vagrant up dc
vagrant up web
vagrant up win7
vagrant halt win7
vagrant halt web
vagrant halt dc
```

## Metasploitable3
Get the metasploitable3 repository
```bash
git clone http://github.com/diaclectic/metasploitable3
cd metasploitable3
```

Build the base VM image using packer
```
cmd.exe /c packer build windows_2008_r2.json
```
Then add the base vagrant box to your vagrant environment
```bash
vagrant box add windows_2008_r2_virtualbox.box --name=metasploitable3
```
Install the reload vagrant provisioner
```bash
vagrant plugin install vagrant-reload
```
Start the vm
```bash
vagrant up
```

Once this is setup, you can use the default credentials U:vagrant/P:vagrant

# References
For more information, please use the following resources.

#### Resizing disk space with VirtualBox
[tuhrig.de Blog](https://tuhrig.de/resizing-vagrant-box-disk-space/)

#### packer-windows
This was ultimately used as the basis for all the repos reference here.
[GitHub](https://github.com/joefitzgerald/packer-windows)

#### packer-windows (2)
This is a modified version of the repo above.
[GitHub](https://github.com/StefanScherer/packer-windows)

#### adfs2
This is a multi vagrant infrastructure with AD for testing ADFS 2.
[GitHub](https://github.com/StefanScherer/adfs2)

#### Metasploitable3
This is an intentionally vulnerable VM used for pen testing.
[GitHub](https://github.com/rapid7/metasploitable3)
[Wiki page](https://github.com/rapid7/metasploitable3/wiki/Vulnerabilities)
[Blog post](https://community.rapid7.com/community/metasploit/blog/2016/11/15/test-your-might-with-the-shiny-new-metasploitable3)


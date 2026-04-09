09/04/2026

[home assistant install for linux](https://www.home-assistant.io/installation/linux/)

KVM option available install container using
* vrt-install
* vrt-manager

## install tools

[stackoverflow what's the difference vrsh, vrt-install, vrt-manager](https://unix.stackexchange.com/questions/163519/what-are-the-diffence-between-virsh-virt-install-and-virt-manager)
* virsh is vm management (create, destroy, stop, start)
* vrt-install is simple create vm
* vrt-manager is GUI

[vrt-install man page](https://linux.die.net/man/1/virt-install)

uses libvirt
    - [libvirt arch man](https://wiki.archlinux.org/title/Libvirt)
    - [libvirt wikipedia](https://en.wikipedia.org/wiki/Libvirt)
        * API for 
        * support many hypervisor and used by many management tools
        
[virt manager desktop application website](https://virt-manager.org/)

# installation with vrt

on home server smelly-server-host

wget https://github.com/home-assistant/operating-system/releases/download/17.2/haos_ova-17.2.qcow2.xz

downloads: -rw-rw-r--  1 mcadmin mcadmin 554921332 Apr  7 10:47 haos_ova-17.2.qcow2.xz

virt-install --name haos --description "Home Assistant OS" --os-variant=generic --ram=4096 --vcpus=2 --disk ./haos_ova-17.2.qcow2.xz,bus=scsi --controller type=scsi,model=virtio-scsi --import --graphics none --boot uefi

mm now I'm here:

UEFI Interactive Shell v2.2
EDK II
UEFI v2.70 (EDK II, 0x00010000)
Mapping table
     BLK0: Alias(s):
          PciRoot(0x0)/Pci(0x3,0x0)/Scsi(0x0,0x0)
Press ESC in 1 seconds to skip startup.nsh or any other key to continue.
Shell> exit

## what happened with the vm

[virsh man page](https://www.libvirt.org/manpages/virsh.html#synopsis)

> sudo virsh list
> [sudo] password for mcadmin: 
> error: failed to connect to the hypervisor
> error: Failed to connect socket to '/var/run/libvirt/libvirt-sock': No such file or directory

well that doesn't look great

[googling error stackoverflow says add user to kvm groups](https://stackoverflow.com/questions/52642739/kvm-failed-to-connect-to-the-hypervisor-error)
* kvm
* libvrtd

[what is qemu its the cpu emulator](https://www.reddit.com/r/linuxquestions/comments/1ixj4o8/can_someone_explain_qemu_to_me_really_fast/)
* used/configured by hypervisor


try reboot and now this:

> virt-install --name haos --description "Home Assistant OS" --os-variant=generic --ram=4096 --vcpus=2 --disk ./haos_ova-17.2.qcow2.xz,bus=scsi --controller type=scsi,model=virtio-scsi --import --graphics none --boot uefi
> ERROR    Failed to connect socket to '/var/run/libvirt/libvirt-sock': No such file or directory

... wondering if above was due to running the command or just running jvm or something I'm not sure at all

I am too newb I don't even know what I need to install

[this kvm for ubuntu tutorial looks promising](https://medium.com/codemonday/setup-virt-manager-qemu-libvert-and-kvm-on-ubuntu-20-04-fa448bdecde3)

aha I had not installed like anything for libvirt, kvm yet, now I get:

> mcadmin@smelly-server-host:~$ virsh list --all
>  Id   Name   State
> --------------------

# try vm install again


virt-install --name haos --description "Home Assistant OS" --os-variant=generic --ram=4096 --vcpus=2 --disk ./haos_ova-17.2.qcow2.xz,bus=scsi --controller type=scsi,model=virtio-scsi --import --graphics none --boot uefi

> mcadmin@smelly-server-host:~/home-assistant$ virt-install --name haos --description "Home Assistant OS" --os-variant=generic --ram=4096 --vcpus=2 --disk ./haos_ova-17.2.qcow2.xz,bus=scsi --controller type=scsi,model=virtio-scsi --import --graphics none --boot uefi
> WARNING  /home/mcadmin/home-assistant/haos_ova-17.2.qcow2.xz may not be accessible by the hypervisor. You will need to grant the 'libvirt-qemu' user search permissions for the following directories: ['/home/mcadmin']
> WARNING  Using --osinfo generic, VM performance may suffer. Specify an accurate OS for optimal results.

> Starting install...
> ERROR    internal error: process exited while connecting to monitor: 2026-04-08T23:13:31.869685Z qemu-system-x86_64: -blockdev {"driver":"file","filename":"/home/mcadmin/home-assistant/haos_ova-17.2.qcow2.xz","node-name":"libvirt-1-storage","auto-read-only":true,"discard":"unmap"}: Could not open '/home/mcadmin/home-assistant/haos_ova-17.2.qcow2.xz': Permission denied
> Domain installation does not appear to have been successful.
> If it was, you can restart your domain by running:
>   virsh --connect qemu:///system start haos
> otherwise, please restart your installation.

* tried sudo
* tried virsh --connect qemu:///system start haos

look at warning "You will need to grant the 'libvirt-qemu' user search permissions for the following directories: ['/home/mcadmin'] "

[askubuntu bottom answer](https://askubuntu.com/questions/722034/permission-error-in-virtual-machine-manager)
* setfacl (file access control list)
* add libvirt-qemu to acl with read access for iso files
* add libvirt-qemu to acl with read and write for disk images /usually/var/lib/libvirt/images

[this reddit post has nice descriptions of giving libvirt read of iso](https://www.reddit.com/r/Fedora/comments/yh3h38/permission_error_on_virt_manager_qemukvm/)
* one person uses acl to give qemu rwx access to iso dir
* one person has /media/{isos,qemu} owned by libvirt and stores them there

do acl for now I don't mind but I like the long term /media dir solution

sudo setfacl -m u:libvirt-qemu:r /home/mcadmin/home-assistant

try virt-install again

> WARNING  /home/mcadmin/home-assistant/haos_ova-17.2.qcow2.xz may not be accessible by the hypervisor. You will need to grant the 'libvirt-qemu' user search permissions for the following directories: ['/home/mcadmin']

ok needs to search the whole user

mcadmin@smelly-server-host:~/home-assistant$ sudo setfacl -m u:libvirt-qemu:r /home/mcadmin/

still same warning now, try the /media dir solution

mcadmin@smelly-server-host:~/home-assistant$ sudo mkdir /media/qemu
mcadmin@smelly-server-host:~/home-assistant$ sudo setfacl -m u:libvirt-qemu:r /home/mcadmin/


mkdir, chmod libvirt-qemu.libvirt-qemu

sudo mv qcow2.xz

try virt-install again

virt-install --name haos --description "Home Assistant OS" --os-variant=generic --ram=4096 --vcpus=2 --disk /media/qemu/haos_ova-17.2.qcow2.xz,bus=scsi --controller type=scsi,model=virtio-scsi --import --graphics none --boot uefi

ok that was fast now I'm here again: 

> UEFI Interactive Shell v2.2
> EDK II
> UEFI v2.70 (EDK II, 0x00010000)
> Mapping table
>      BLK0: Alias(s):
>           PciRoot(0x0)/Pci(0x3,0x0)/Scsi(0x0,0x0)
> Press ESC in 1 seconds to skip startup.nsh or any other key to continue.
> Shell> 

## new newb mistake extracting file

I was trying tar -xvf on the .qcow2.xz confused why it wouldn't work, gpt says tar works for .tar.xz files but just .xz files are not tars - fair

a couple google results down in how to extract .xz file is this lovely link

[how to extract .xz file with xz-utils](https://linux-tips.com/t/how-to-extract-xz-files/265)
*unxz [file]

thank fuck: mcadmin@smelly-server-host:~/home-assistant$ xz -d haos_ova-17.2.qcow2.xz 


try virt install from /home/mcadmin/home-assistant again

permissions error: eventually try "sudo setfacl -m u:libvirt-qemu:rwx /home/mcadmin

then find out I was overlooking this error (must be from when I made a vm out of the .qcow2.xz file)

> Starting install...
> ERROR    Guest name 'haos' is already in use.
> Domain installation does not appear to have been successful.
> If it was, you can restart your domain by running:
>   virsh --connect qemu:///system start haos
> otherwise, please restart your installation.
> mcadmin@smelly-server-host:~/home-assistant$ virsh --connect qemu:///system start haos
> error: Domain is already active


woopsie

how to virsh kill

[back to virsh man page](https://www.libvirt.org/manpages/virsh.html#synopsis)

[why are they called domains](https://unix.stackexchange.com/questions/408308/why-are-virtual-machines-in-kvm-qemu-called-domains)
* historic thing, first system initialised on kernel is dom0 (hypervisor) and other later domains domU are guests or VMs
* also domain means area owned or area of autonomy so it's like the space and resources allocated to the VM

[redhat remove domain toot](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-virsh-delete)
* # virsh undefine [domain] --remove-all-storage

[also shutdown options on redhat toot](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-managing_guest_virtual_machines_with_virsh-shutting_down_rebooting_and_force_shutdown_of_a_guest_virtual_machine)
* # virsh shutdown

mcadmin@smelly-server-host:~/home-assistant$ sudo virsh shutdown haos

stuck in: 

> Domain 'haos' is being shutdown

so getting:

> mcadmin@smelly-server-host:~/home-assistant$ sudo virsh undefine haos --remove-all-storage
> error: Storage volume deletion is supported only on stopped domains

sudo destroy! `sudo virsh destroy haos`

wait I missed we have a new problem

> error: Requested operation is not valid: cannot undefine domain with nvram

[random forum answer to undefine vram error](https://forums.unraid.net/topic/51102-way-to-manually-remove-or-delete-vms/)
* easy as virsh undefine --nvram haos

> Welcome to Home Assistant
> homeassistant login: 

yaaaay

but also rude? what login? move on to [home assistant onboarding tutorial](https://www.home-assistant.io/getting-started/onboarding/)

but first try http://homeassistant.local:8123/ -> nope
http://homeassistant:8123/ -> nope
http://smelly-server-host:8123/ -> nope

[troubleshooting says it should have worked](https://www.home-assistant.io/installation/troubleshooting/)

I suspect this login sitch

try just pressing enter for empty login -> nope

[these guys had same issue but connected to link via browser to create a login](https://community.home-assistant.io/t/is-there-a-default-username-and-pwd-to-get-new-ha-installation-started/280394/11)

so I couldn't connect I tried:
* wifi network of router that server is connected to
* smelly-server-host:port
* IPs listed in mcadmin@smelly-server-host:~$ hostname -I

so then I tried mcadmin@smelly-server-host:~$ sudo netstat -plnt [that I got from here](https://docs.rackspace.com/docs/checking-listening-ports-with-netstat)

and I didn't see my service (although not on that machine is it, it's on the VM)

I get frustrated and close the terminal in the VM

I try connect to the domain from a new terminal session

> virsh connect haos
> error: failed to connect to the hypervisor
> error: no connection driver available for URI 'haos' does not include a driver name

[libvrt forum about can't connect to domain driver something](https://wiki.libvirt.org/Failed_to_connect_to_the_hypervisor.html)
* sometimes happens when libvirt compiled from sources -> oh I didn't do that though so probs not this

probably I'll destroy it and start again another time

thanks that was great



# Notes and snippets for __Virtualization with KVM and QEMU__ on LinkedIn Learning

This file contains brief notes to accompany the course. See the course for more details and context. Specific implementation on your system will vary.

## 01_04 - QEMU command-line tools

To install QEMU on Ubuntu:

```
apt install qemu-kvm
```

QEMU commands can be lengthy and we should compose them in text editors instead of the terminal. The following commands are functionally the same but are presented in different ways. When we are satisfied with our command, we can copy it and paste it in the terminal using Ctrl-Shift-V.

Many options presented on the same line:

```
qemu-system-x86_64 -enable-kvm -cpu host -smp 4 -m 8G -k en-us -vnc :0 -usbdevice tablet -drive file=disk1.qcow2,if=virtio -cdrom ubuntu-22.04-desktop-amd64.iso -boot d 
```

The same options presented with line-continuation characters ( \\ ) to make the command more readable:

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -vnc :0 \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -cdrom ubuntu-22.04-desktop-amd64.iso \
  -boot d 
```

## 02_01 - Creating disk images

To create a qcow2 disk image with a 50GB capacity:

```
qemu-img create -f qcow2 disk1.qcow2 50G
```

## 02_02 - Creating a Virtual Machine

Create a basic QEMU guest using KVM, 4 vCPUs, and 8GB of RAM, with a VNC display at port 5900 on the host:

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -vnc :0 \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -cdrom ubuntu-22.04-desktop-amd64.iso \
  -boot d 
```

*Remember: be sure to substitute your disk image paths and names*

## 02_03 - Installing a guest operating system

Make a copy of our installed disk image for later:

```
cp disk1.qcow2 disk2.qcow
```

## 02_04 - Control and debug a guest with QEMU Monitor

Start a guest with the QEMU Monitor available for control and troubleshooting:

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -vnc :0 \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -monitor stdio
```

Launching a guest with a VNC password for macOS Screen Sharing compatibility (note `password=on` below):

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -vnc :0,password=on \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -monitor stdio
```

Set or change the VNC password from the QEMU Monitor:

```
change vnc password
```

Send the shutdown signal to a guest in QEMU Monitor:

```
system_powerdown
```

## 02_05 - Managing and modifying disk images

### Modifying a disk image

If desired, create an overlay file to store changes:

```
qemu-img create -o backing_file=file.qcow2,backing_fmt=qcow2 -f overlay.qcow2
```

If necessary, rebase an overlay file:

```
qemu-img rebase -b file.qcow2 overlay.qcow2
```

Resize an image file, either with relative or absolute size:

```
qemu-img resize +50G file.qcow2
qemu-img resize 100G file.qcow2
```

### Inspecting a disk image

Load the NBD kernel module:

```
sudo modprobe nbd
```

Attach the disk image as an NBD device:

```
sudo qemu-nbd --connect=/dev/nbd0 /home/scott/disk.qcow2
```

Find the appropriate partition to mount:

```
lsblk -f /dev/nbd*
```

Create a mount point for the partition:

```
sudo mkdir /mnt/mydisk
```

Mount the partition:

```
sudo mount /dev/nbd0p3 /mnt/mydisk
```

Use the mounted partition:

```
ls /mnt/mydisk
```

Unmount the partition:

```
sudo umount /mnt/mydisk
```

## 02_06 - Guest graphics and display options

Start a guest with a GTK display window and the Standard VGA graphics card:

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display gtk \
  -vga std \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio
```

Start a guest with an SDL display window and the VirtIO graphics card:

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio
```

## 02_07 - Sharing files between host and guest

Create a 9P file share to use with the guest, with the host directory `/home/scott/shared` (change this to suit your system):

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -virtfs local,path=/home/scott/shared,mount_tag=shared,security_model=mapped
```

Mount the file share in the guest:

```
# From https://wiki.qemu.org/Documentation/9psetup#Mounting_the_shared_path
mount -t 9p -o trans=virtio shared /mnt/shared -oversion=9p2000.L
```

## 02_08 - Using host USB devices in a guest

Pass a specific USB device to the guest by creating a USB 3 host adapter and plugging the device into it:

```
sudo qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -device qemu-xhci,id=xhci \
  -device usb-host,bus=xhci.0,vendorid=0x046d,productid=0x0843
```

*Remember: your device's Vendor ID and Product ID will be different. Find these values with the command `lsusb` on the host.*

Pass a specific USB port to the guest by creating a USB 3 host adapter and attaching it to the host port:

```
sudo qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -device qemu-xhci,id=xhci \
  -device usb-host,bus=xhci.0,hostbus=1,hostport=5
```

*Remember: your port's Bus ID and Port ID will be different. Find these values with the command `lsusb` on the host.*

## 03_02 - User-mode networking

Provide the guest an emulated RTL8139 network interface connected to a User Mode networking backend `net0`:

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -netdev user,ipv6=off,id=net0 \
  -device rtl8139,netdev=net0
```

Provide the guest a VirtIO network interface using the `-nic` option:

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -nic user,model=virtio-net-pci
```

## 03_03 - Port forwarding

Forward host port 2222 to guest port 22, and host port 8080 to guest port 80:

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -nic user,model=virtio-net-pci \
  -nic user,hostfwd=::2222-:22,hostfwd=::8080-:80
```

## 03_04 - Removing network connectivity

Remove the NIC from the guest:

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -nic none
```

## 03_05 - Bridged networking

Create a network bridge `br0` on the host:

```
sudo ip link add br0 type bridge
sudo ip link set br0 up
```

Manually start two guests on a bridged network (for illustrative purposes only):

```
# create tap interfaces tap0 and tap1
ip tuntap add tap0 mode tap && ip tuntap add tap1 mode tap

# bring up tap interfaces tap0 and tap1
ip link set tap0 up promisc on && ip link set tap1 up promisc on

# connect tap interfaces tap0 and tap1 to the bridge br0
ip link set tap0 master br0 && ip link set tap1 master br0

# start two guests using tap0 and tap1 (remember to use two different disk images)
qemu-system-x86_64 ... -netdev tap,id=t0,ifname=tap0,script=no,downscript=no -device e1000,netdev=t0,id=nic0

qemu-system-x86_64 ... -netdev tap,id=t1,ifname=tap1,script=no,downscript=no -device e1000,netdev=t1,id=nic1
```

Set up the QEMU Bridge Helper:

```
sudo chmod u+s /usr/lib/qemu/qemu-bridge-helper
sudo echo "allow br0" > /etc/qemu/bridge.conf
sudo chmod o+r /etc/qemu/bridge.conf
```

## 03_06 - Creating a private network

Bring up a guest on a bridged network using the QEMU Bridge Helper and the `-netdev` and `-device` options:

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -netdev bridge,br=br0,id=net1 \
  -device virtio-net,netdev=net1
```

Bring up a guest on a bridged network using the QEMU Bridge Helper and the `-nic` option, specifying a different MAC address than the default:

```
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -k en-us \
  -display sdl \
  -vga virtio \
  -usbdevice tablet \
  -drive file=disk1.qcow2,if=virtio \
  -nic bridge,br=br0,mac=52:54:00:12:34:57 \
  -device virtio-net
```

## 03_07 - Creating a host-only network

Add the host to the bridge `br0`

```
sudo ip addr add 10.10.10.1/24 dev br0
```

Start a DHCP server using `dnsmasq` on the host to provide addresses within the bridged network:

```
sudo dnsmasq --interface=br0 --bind-interfaces --dhcp-range=10.10.10.2,10.10.10.100
```

## 03_08 - Creating a NAT network

Configure the host to provide packet forwarding and NAT through the bridge `br0` at `10.10.10.1`:

```
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -A FORWARD -i br0 -j ACCEPT
sudo iptables -t nat -I POSTROUTING -s 10.10.10.1/24 -j MASQUERADE
```

## 03_09 - Public virtual bridge

Set up the bridge on the host again:

```
sudo ip link add br0 type bridge
sudo ip link set br0 up
```

Attach the host's network interface `enp4s0f2` to the bridge:

```
sudo ip link set enp4s0f2 master br0
```

*Remember: your network interface may be different. You can find the identifiers for your host network interfaces by running `ip a` on the host.*

Remove the host's network interface `enp4s0f2` from the bridge:

```
sudo ip link set dev enp4s0f2 nomaster
```

## 04_01 - Exploring libvirt

Install libvirt and clients on the host:

```
sudo apt install libvirt-daemon-system libvirt-clients
```

## 04_02 - Exploring Virtual Machine Manager

Install virt-manager on the host:

```
sudo apt install virt-manager
```

## 04_03 - Exploring Virsh

Start virsh in interactive mode on the host:

```
virsh
```

In the virsh shell, list running domains:

```
list
```

In the virsh shell, list all domains:

```
list --all
```

In the virsh shell, edit the domain `ubuntu22.04`

```
edit ubuntu22.04
```

In the virsh shell, show the help:

```
help
```

Quit the virsh shell:

```
quit
```

On the host, start the domain `ubuntu22.04` from the shell:

```
virsh start ubuntu22.04
```

Start virt-viewer on the host:

```
virt-viewer
```

If virt-viewer complains about not being able to open the display, try this:

```
export DISPLAY=:0
```

On the host, safely shut down the domain `ubuntu22.04` from the shell:

```
virsh shutdown ubuntu22.04
```

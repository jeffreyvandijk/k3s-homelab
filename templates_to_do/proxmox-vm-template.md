vm template proxmox:

wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
mv noble-server-cloudimg-amd64.img ubuntu-2404-server.qcow2
qm set 999 --serial0 socket --vga serial0
qemu-img resize ubuntu-2404-server.qcow2 150G
qm importdisk 999 ubuntu-2404-server.qcow2 local-lvm
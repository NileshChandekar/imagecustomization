#!/bin/bash

clear

declare LIBVIRT_BASE_DIRECTORY=/tmp
declare LIBVIRT_BASE_UBUNTU_QCOW2=jammy-server-cloudimg-amd64.qcow2
declare QEMU_IMG=$(which qemu-img)
declare VIRT_CT=$(which virt-customize)
declare QEMU_NBD=$(which qemu-nbd)
declare MOD_PROBE=$(which modprobe)
declare GRPART=$(which growpart)
declare E2FSK=$(which e2fsck)
declare RESI2FS=$(which resize2fs)
declare GUEST_MNT=$(which guestmount)
declare GUEST_UNMT=$(which guestunmount)



  # load the nbd module
  sudo $MOD_PROBE nbd max_part=5

  sleep 2

  # resize the image first
  sudo $QEMU_IMG resize "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2" 60G

  sleep 2

  # mount the qcow2 image
  sudo $QEMU_NBD -c /dev/nbd0 "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2"

  sleep 2

  # grow the partition size on the qcow2 image
  sudo $GRPART /dev/nbd0 1

  sleep 2

  # check the filesystem on the partition
  sudo $E2FSK -y -f /dev/nbd0p1

  sleep 2

  # resize the filesystem on the partition
  sudo $RESI2FS /dev/nbd0p1

  sleep 2

  # unmount the qcow2 image
  sudo $QEMU_NBD -d /dev/nbd0


  sleep 2

  # remove the module
  sudo $MOD_PROBE -r nbd


  sleep 2

  # set the root password on the image
  sudo $VIRT_CT -a "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2" \
  --root-password password:0

  sleep 2
  # enable multi-user target on the base image
  sudo $VIRT_CT -a "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2" \
  --upload /tmp/interface:/etc/network/interfaces

  sleep 2
  # enable multi-user target on the base image
  sudo $VIRT_CT -a "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2" \
  --upload /tmp/ssh:/etc/ssh/sshd_config 

  sleep 2

  # install the network-manager package on the base image
  sudo $VIRT_CT -a "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2" \
  --install network-manager

  # install the ifupdown package on the base image
  sudo $VIRT_CT -a "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2" \
  --install ifupdown

  sleep 2

  # ssh host keys on the base image
  sudo $VIRT_CT -a "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2" \
  --run-command "ssh-keygen -A"

  sleep 2

  # enable multi-user target on the base image
  sudo $VIRT_CT -a "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2" \
  --run-command "systemctl set-default multi-user.target"

  sleep 2

  # enable multi-user target on the base image
  sudo $VIRT_CT -a "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2" \
  --run-command "systemctl restart ssh"


  sleep 2

  # enable multi-user target on the base image
  sudo $VIRT_CT -a "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2" \
  --run-command "systemctl restart sshd"

#!/bin/bash

clear

declare LIBVIRT_BASE_DIRECTORY=/tmp
declare LIBVIRT_BASE_UBUNTU_QCOW2=jammy-server-cloudimg-amd64.qcow2
declare QEMU_IMG=$(which qemu-img)
declare VIRT_CT=$(which virt-customize)

  # resize the image first
  $QEMU_IMG resize "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2" 60G

  sleep 2

  # set the root password on the image
  sudo $VIRT_CT -a "$LIBVIRT_BASE_DIRECTORY/$LIBVIRT_BASE_UBUNTU_QCOW2" \
  --root-password password:rack1234

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

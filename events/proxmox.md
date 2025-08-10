# Proxmox

[VM serial access](https://chatgpt.com/share/6865228d-3c68-8005-b2ca-d8d74f4d0580)

# TODO: WHAT IS PROXMOX?

## [YouTube Tutorial](https://www.youtube.com/watch?v=5j0Zb6x_hOk&list=PLT98CRl2KxKHnlbYhtABg6cF50bYa8Ulo)
- Login
  - Username: `root`
  - Password: `password`

- Chrome Reader Mode extension somehow interferes with web proxy
- You can ssh into your proxmox
- First thing, Updates > Refresh, _Upgrade
  - Ignore errors
  - Turn off Proxmox repositories by going into Updates > Repositories and disabling anything with `enterprise.proxmox`
  - Adding a Test repository means being an early adopter
  - You will see your action in the Task History section
- Disks
  - Although the LVM section might suggest there's no free space, the used space is assigned to LVM, which you can see in the LVM Thin section
- VMs migrate between Proxmox servers on a cluster quickly and stay live. Containers are the opposite.
- Predownload:
  - Proxmox ISO
  - Ubuntu server ISO
- networking for Ubuntu Server
  - Proxmox ip: 172.24.201.156
  - Subnet: 172.24.192.0/20
  - Address: 172.24.201.200
  - Gateway: 172.24.192.1
  - Name servers: 8.8.8.8, 1.1.1.1

- When starting from a VM template, you will need to run:
  ```
  sudo ssh-keygen -A
  sudo systemctl enable ssh
  sudo systemctl start ssh
  ```
  Modify /etc/hostname and /etc/hosts so the hostname is different, then `sudo reboot`

- Agenda:
  - [Create a VM](https://www.youtube.com/watch?v=xBUnV2rQ7do&list=PLT98CRl2KxKHnlbYhtABg6cF50bYa8Ulo&index=6)
    - Add an ISO
      - Datacenter > orlando-2 > local > ISO Images > Download from URL
    - Create a VM button
      - Name: webserver
      - OS > ISO Image
      - Hard Disk
        - SCSI Controller > check discard
        - 16GB
      - CPU > pick a Type with AES (x86-64-v2), apparently some of the PCs allow this and some don't, so I can't move a VM
      - Memory > 1024MB
    - Note in Options, there is the Start At Boot option and the Start/Shutdown order option
    - Go into the Console and accept all the defaults EXCEPT install the OpenSSH server
    - Once rebooting and logging in, note that you can SSH into the VM
    - `sudo apt update && sudo apt dist-upgrade`
    - `cat /proc/cpuinfo | grep "model name"` to show this is a virtual CPU
    - No need
      - `sudo apt install qemu-guest-agent` - what does this do???
      - `sudo systemctl start qemu-guest-agent.service` and notice it hangs
      - Proxmox > VM > Options > QEMU Guest Agent > Enable
      - restart vm and then you can enable it
    - `sudo apt install apache2`
  - VM template
    - In `/etc/ssh`, move all the `ssh_host_*` files to another directory
    - Note that `ln -l /var/lib/dbus/machine-id` is a symbolic link to `/etc/machine-id`
    - `sudo truncate -s 0 /etc/machine-id`
    - `sudo apt clean && sudo apt autoremove` What do these do???
    - Can't get this to work
      - `apt search cloud-init` - Help with VM templates. It is supposed to reset the ssh host keys, but it doesn't???
      - `sudo cloud-init clean`
    - `sudo poweroff`
    - Right-click VM > Convert To Template
    - VM Template > Hardware
      - Remove CD drive
      - Can't get to work:
        - Add Cloud Init drive > local-lvm
        - Cloud Init
          - set user to mbuchoff
          - Set password
    - Right-click template, clone
      - Name: webserver-1
      - full clone
    - log in, `ip a`, access it with a web browser
    - Adjust /etc/hostname and /etc/hosts with a unique host
    - sudo reboot
    - ```
      sudo ssh-keygen -A
      sudo systemctl enable ssh
      sudo systemctl start ssh
      ```
  - Connect a cluster
    - In case it's already connected, `pvecm list nodes` lists nodes and `pvecm delnode #` deletes a node. That way, you can reconnect nodes.
    - It is a bit of a rigamarole to disconnect the cluster from the node, so I just reformatted ProxMox on that computer.
    - From host: Cluster name on left > Cluster > Join Information
    - From client: Cluster > Join > paste, enter password
  - Transfer a VM between nodes
    - Right-click > Migrate
  - Create a container template
    - Local > Container Templates > Templates > Search for Ubuntu > pick one
  - Create a container
    - Create CT
      - Template from above
      - Network > DHCP
    - In Options, note the boot options and the Unprivileged Container option
    - In the Console, log in as username root
    - `ip a` to get ip
    - Try ssh-ing. Fail
    - ```
      adduser mbuchoff
      usermod -aG sudo mbuchoff
      ```
    - ssh again as mbuchoff
    - ```
      apt update
      apt install apache2
      ```
  - Transfer a container between nodes

# Provisioning Compute Resources

Note: You must have VirtualBox and Vagrant configured at this point

Clone the repository:

```shell
$ git clone https://github.com/mmumshad/kubernetes-the-hard-way.git
```

Navigate into the _vagrant_ directory:

```shell
$ cd kubernetes-the-hard-way\vagrant
```

Start the master-, worker nodes and loadbalancer

```shell
$ vagrant up
```

This does the below:

- Deploys 5 VMs - 2 Master, 2 Worker and 1 Loadbalancer with the name 'kubernetes-ha-* '
    > This is the default settings. This can be changed at the top of the Vagrant file

- Set's IP addresses in the range 192.168.5

    | VM            |  VM Name               | Purpose       | IP           | Forwarded Port   |
    | ------------  | ---------------------- |:-------------:| ------------:| ----------------:|
    | master-1      | kubernetes-ha-master-1 | Master        | 192.168.5.11 |     2711         |
    | master-2      | kubernetes-ha-master-2 | Master        | 192.168.5.12 |     2712         |
    | worker-1      | kubernetes-ha-worker-1 | Worker        | 192.168.5.21 |     2721         |
    | worker-2      | kubernetes-ha-worker-2 | Worker        | 192.168.5.22 |     2722         |
    | loadbalancer  | kubernetes-ha-lb       | LoadBalancer  | 192.168.5.30 |     2730         |

    > These are the default settings. These can be changed in the Vagrant file

- Add's a DNS entry to each of the nodes to access internet
    > DNS: 8.8.8.8

- Install's Docker on Worker nodes

- Runs the below command on all nodes to allow for network forwarding in IP Tables.
  This is required for kubernetes networking to function correctly.
    > sysctl net.bridge.bridge-nf-call-iptables=1


## SSH to the nodes

There are two ways to SSH into the nodes:

### 1. SSH using Vagrant

  From the directory you ran the `vagrant up` command, run `vagrant ssh <vm>` for example `vagrant ssh master-1`.
  > Note: Use VM field from the above table and not the vm name itself.

### 2. SSH Using SSH Client Tools

Use your favourite SSH Terminal tool (putty).

Use the above IP addresses. Username and password based SSH is disabled by default.
Vagrant generates a private key for each of these VMs. It is placed under the .vagrant folder (in the directory you ran the `vagrant up` command from) at the below path for each VM:

**Private Key Path:** `.vagrant/machines/<machine name>/virtualbox/private_key`

**Username:** `vagrant`


## Verify Environment

- Ensure all VMs are up

```shell
$ vagrant global-status
```

Ouput example:

```
id       name         provider   state   directory                                                                    
---------------------------------------------------------------------------
2cf2393  master-1     virtualbox running ../kubernetes-the-hard-way/vagrant
b5d9c70  master-2     virtualbox running ../kubernetes-the-hard-way/vagrant
ea8d587  loadbalancer virtualbox running ../kubernetes-the-hard-way/vagrant
eaef426  worker-1     virtualbox running ../kubernetes-the-hard-way/vagrant
71f6280  worker-2     virtualbox running ../kubernetes-the-hard-way/vagrant
```

Ensure that `masters`, `workers` and `loadbalancer` are in _running_ state.

Next step is to:

- ensure the VMs are assigned the above IP provided addresses
- ensure SSH access into the VMs using the IP and private keys is enabled
- ensure the VMs can ping each other
- ensure the worker nodes have Docker installed on them. Version: 18.06
  > command `sudo docker version`

  Manually SSH into each node and determine it's assigned IP, and make sure that it matches the IP listed in earlier provided IP table.

  Start with the _master-1_ node:

  ```shell
  $ vagrant ssh master-1    # ssh into master-1 node
  $ ip a                    # should output an interface with linked ip: 192.168.5.11
  $ ping 192.168.5.12       # check if master-1 can ping master-2
  $ ping 192.168.5.21       # check if master-1 can ping worker-1
  $ ping 192.168.5.22       # check if master-1 can ping worker-2
  ```

  Output example:

  ```shell
  $ vagrant ssh master-1                                                # ssh into master-1 node

  Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-135-generic x86_64)
  ...

  vagrant@master-1:~$ ip a                                              # list ip's
      ...
  3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
      ...
      inet 192.168.5.11/24 brd 192.168.5.255 scope global enp0s8        # here you see the IP of master-1 node

        ...

  vagrant@master-1:~$ ping 192.168.5.12                                 # ping master-2     
  PING 192.168.5.12 (192.168.5.12) 56(84) bytes of data.
  64 bytes from 192.168.5.12: icmp_seq=1 ttl=64 time=0.460 ms
  64 bytes from 192.168.5.12: icmp_seq=2 ttl=64 time=0.344 ms
  ^C
  --- 192.168.5.12 ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1062ms
  rtt min/avg/max/mdev = 0.344/0.402/0.460/0.058 ms

  vagrant@master-1:~$ ping 192.168.5.21                                 # ping worker-1
  PING 192.168.5.21 (192.168.5.21) 56(84) bytes of data.
  64 bytes from 192.168.5.21: icmp_seq=1 ttl=64 time=0.449 ms
  64 bytes from 192.168.5.21: icmp_seq=2 ttl=64 time=0.320 ms
  ^C
  --- 192.168.5.21 ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1037ms
  rtt min/avg/max/mdev = 0.320/0.384/0.449/0.067 ms

  vagrant@master-1:~$ ping 192.168.5.22                                 # ping worker-2
  PING 192.168.5.22 (192.168.5.22) 56(84) bytes of data.
  64 bytes from 192.168.5.22: icmp_seq=1 ttl=64 time=0.465 ms
  64 bytes from 192.168.5.22: icmp_seq=2 ttl=64 time=0.316 ms
  ^C
  --- 192.168.5.22 ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1279ms
  rtt min/avg/max/mdev = 0.316/0.390/0.465/0.077 ms
  ```

## Troubleshooting Tips

If any of the VMs failed to provision, or is not configured correct, stop and or delete VM's using the following command:

Stop VM's:

```shell
$ vagrant halt
```

Delete a specific VM:

```shell
$ vagrant destroy <vm>
```

Then reprovision. Only the missing VMs will be re-provisioned:

```shell
$ vagrant up
```

Sometimes the delete does not delete the folder created for the vm and throws the below error.

VirtualBox error:

```shell
VBoxManage.exe: error: Could not rename the directory 'D:\VirtualBox VMs\ubuntu-bionic-18.04-cloudimg-20190122_1552891552601_76806' to 'D:\VirtualBox VMs\kubernetes-ha-worker-2' to save the settings file (VERR_ALREADY_EXISTS)
VBoxManage.exe: error: Details: code E_FAIL (0x80004005), component SessionMachine, interface IMachine, callee IUnknown
VBoxManage.exe: error: Context: "SaveSettings()" at line 3105 of file VBoxManageModifyVM.cpp
```

In such cases delete the VM, then delete the VM folder and then re-provision

```shell
$ vagrant destroy <vm>
$ rmdir "<path-to-vm-folder>\kubernetes-ha-worker-2"
$ vagrant up
```

Next: [Installing the Client Tools](03-client-tools.md)

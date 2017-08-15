#vdcm
To run this ansible playbook, follow these instructions

1. First, the vDCM has to have an Management IP and must be reachable through SSH2. To check this, navigate in the remote device to the folder

/etc/sysconfig/network-scripts

2. List down the interfaces by running the command

ifconfig

With this list, the administrator can check the MAC ADDRESS and see which interface corresponds to the management network

3. edit with vim the ifcfg-en* file associated to the Management interface. A File like the shown below will be displayed

TYPE=Ethernet
BOOTPROTO=dhcp
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens224
UUID=9f8250b0-49d0-8bbe-df19aab348be
DEVICE=ens224
ONBOOT=no

To configure an static IPV4 IP ADDRESS, the file must be set similar to the shown below

TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=yes 
IPV6INIT=no
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens224
UUID=9f8250b0-49d0-8bbe-df19aab348be
DEVICE=ens224
ONBOOT=yes
IPADDR=10.0.0.101
PREFIX=24
GATEWAY=10.0.0.1

4. After saving the edited interface configuration file, restart the network services to apply the changes with the command

services network restart

5. At this moment, the local PC must be able to reach the remote vDCM. Test the following command in the local PC to the remote Management IP. The administrator must have the root password or the user/password to access the machine

ssh2 root@10.0.0.101

In this case, it is used the root user to access the remote machine. This will request the password to access the remote vDCM

6. If successful, the administrator the must add this device in the Ansible's hosts file. By default, Ansible uses the file /etc/ansible/hosts to run all the ansible playbooks and commands. In this case the administrator will use another hosts files located in other folder.

The administrator must navigate to the folder where the playbooks are stored and create a new hosts file. in this case it will be the folder /etc/ansible/vDCM-Installation

After navigating, the user can run the command 

touch ./hosts

to create the file.

7. Edit the hosts file as is shown below, by creating the groups to work and the name of the devices, for example:
---
[vdcm-one]
vdcm1 ansible_ssh_host=10.0.0.101
vdcm2 ansible_ssh_host=10.0.0.102
vdcm3 ansible_ssh_host=10.0.0.103

[vdcm-two]
vdcm4 ansible_ssh_host=10.0.0.104
vdcm5 ansible_ssh_host=10.0.0.105

[vsm]
vsm1 ansible_ssh_host=10.0.0.21

[local]
macpc ansible_ssh_host=127.0.0.1

---

Then, save the file.

8. With the hosts file create, it us time to edit the YAML files (playbooks). To do this, edit each .yml file, and change the line

   hosts: ***

For the group or device to run the playbooks. For example, if the playbook will be run for all the vDCMs in the group vdcm-one, then replace the hosts: line for the foloowing line:

   hosts: vdcm-one

Also, the user can point directly to a device. For example, if the playbook will be run only for the vDCM1 (with IP 10.0.0.101), then replace the hosts: line for

    hosts: vdcm1

Finally, save the file

9. Do this for all the playbooks that will be run to trigger the playbooks only to the desired vDCMs. In this case, the first playbook that will be run is the if_settings.yml playbook. What this playbook does, is mainly:

  - name: Delete extra vNIC on vDCMs
  - name: List all interfaces in host    
  - name: Replace DHCP info on ifcfg-enp*
  - name: Replace IPV4 Failure Fatal on IPV4
  - name: Set IP as IPv4  
  - name: Look Up for Input_IP
  - name: Insert IPADDRESS_INPUT into file
  - name: Look Up for Prefix
  - name: Insert PREFIX_INPUT into file
  - name: Look Up for Output_IP
  - name: Insert IPADDRESS_OUPTUT into file
  - name: Look Up for Prefix
  - name: Insert PREFIX_OUTPUT into file
  - name: Change RP Filter for Input Traffic Port
  - name: Change RP Fier for Output Traffic Port
  - name: Restart Sysctl
  - name: Restart network service

To see the detail, go to the file and read the comments for each line

This Playbook is based mainly in the Lookup function, where the script will Look Up for the management IP configured in the vDCM. Then, will use an Ansible.csv file that contains the relationship between the Management IP and the IP Addresses and Masks for the traffic interfaces (input/output). This information is retrieved and added to the ifcfg-en*** file of the respective interface. Finally, the service is restarted, and sets the IP ADDRESSES and MASK into the interfaces.

The second playbook is the install_vdcm.yml playbook. This playbook manly does:

---
  - name: Creates /temps/ directory
  - name: Disable Firewall on Hosts
  - name: Stop Firewall on Hosts
  - name: Copy SW from localhost to vdcm group
  - name: Run vDCM installation
  - name: Enable to run all vDCM services
  - name: Stop vDCM software
  - name: Add IIOP user for VSM
  - name: Stop again vDCM software
  - name: Comment existing NTP servers
  - name: Update /etc/ntp.conf file
  - name: Restart ntpd service
  - name: Debug NTP State
  - name: Fix warnings
  - name: Change MaxCrashReportsSize /etc/abrt/abrt.conf to 8000
  - name: Comment RateLimitInterval on journald.conf
  - name: Comment RateLimitBurst on journald.conf
---

Some values (for example, NTP IP Address) is set inside the Playbook. For partÃicular IP ADDRESSES for NPT, the playbook must be modified with the respective values.

10. The next step is to save the SSH Key inside the local PC, to allow the ansible playbook to run freely. To do this run the command:

ssh-copy-id <USER>@<vDCM_IP_ADDRESS>

For the example, it would be

ssh-copy-id root@10.0.0.101

This command will request the password for the root user in this machine, by entering it, the key for the root user will be stored internally in the local PC.

11. Now, it is time to run the ansible-playbook. To do this, navigate to the folder with the playbooks and respective hosts file and run:

ansible-playbook -i hosts if_settings.yml 

12. There is another YAML file that appears as vdcm_upgrade.yml but it was only used for testing. No need to run this file.

Happy Scripting :)
Created By: ClassicApe 

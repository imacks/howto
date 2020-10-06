Rancher Cluster Setup on Openstack
==================================
This is a full walkthrough on manually setting up a Rancher cluster as a tenant on Openstack. 

Openstack network configuration
-------------------------------
1. Create a private net:
- Network > Networks > (+ Create Network)
  - Network:
    - Network Name: stagenet
  - Subnet:
    - Subnet Name: default
    - Network address: 192.168.1.0/24
  - Subnet details:
    - Allocation Pools: 192.168.1.100,192.168.1.200
    - DNS Name Servers: 8.8.8.8\n8.8.4.4
  - (Create)

2. Create a router to shuttle traffic between public net and private net:
- Network > Routers > (+Create Router)
  - Router Name: stagenet-rt
  - External Network: pub0net
  - (Create Router)
- Network > Routers > (stagenet-rt hyperlink)
  - Interfaces:
    - (+Add Interface)
    - Subnet: stagenet: 192.168.1.0/24 (default)
    - (Submit)

3. Create a network security group for worker nodes
- Network > Security Groups > (+Create Security Group)
  - Name: open
- Network > Security Groups > (open hyperlink) > (+Add Rule)
  - Rule: Other Protocol
  - IP Protocol: -1
  - (Add)
- Network > Security Groups > (open hyperlink) > (+Add Rule)
  - Rule: Other Protocol
  - IP Protocol: -1
  - CIDR: ::/0
  - (Add)
- Network > Security Groups > (open hyperlink) > (+Add Rule)
  - Rule: Other Protocol
  - IP Protocol: -1
  - Remote: Security Group
  - Security Group: open
  - (Add)

4. Create a network security group for controller node
- Network > Security Groups > (+Create Security Group)
  - Name: secure
- Network > Security Groups > (secure hyperlink) > (+Add Rule)
  - Rule: Other Protocol
  - IP Protocol: -1
  - Remote: Security Group
  - Security Group: secure
  - (Add)
- (Repeat last step but choose `open` for Security Group)

5. Allow `secure` to access `open` group:
- Network > Security Groups > (open hyperlink) > (+Add Rule)
  - Rule: Other Protocol
  - IP Protocol: -1
  - Remote: Security Group
  - Security Group: secure
  - (Add)


Openstack VM instances
----------------------
1. Create your SSH key
- Compute > Key Pairs > (+Create Key Pair)
  - Key Pair Name: default
  - Key Type: SSH Key
  - (+Create Key Pair)
- (Browser will automatically download the .pem key to your computer)

2. Create a VM group
- Compute > Server Groups > (+Create Server Group)
  - Name: ran1-ctls
  - Policy: Soft Anti Affinity
- Compute > Server Groups > (+Create Server Group)
  - Name: ran1-nodes
  - Policy: Soft Anti Affinity

3. Create the controller VM
- Compute > Instances > (+Launch Instance)
  - Details:
    - Instance Name: ran1ctl1
  - Source:
    - Available: vmi-ubuntu-1804lts2-amd64 (click up arrow)
    - Delete Volume on Instance Delete (yes)
  - Flavor: t4a.medium
  - Networks: stagenet
  - Security Groups:
    - default (click down arrow)
    - secure (click up arrow)
  - Key Pair:
    - default (click up arrow)
  - Server Groups:
    - ran1-ctls (click up arrow)
  - (Launch Instance)

4. Create worker node VM
- Compute > Instances > (+Launch Instance)
  - Details:
    - Instance Name: ran1node1
  - Source:
    - Available: vmi-ubuntu-1804lts2-amd64 (click up arrow)
    - Delete Volume on Instance Delete (yes)
  - Flavor: t4a.medium
  - Networks: stagenet
  - Security Groups:
    - default (click down arrow)
    - open (click up arrow)
  - Key Pair:
    - default (click up arrow)
  - Server Groups:
    - ran1-nodes (click up arrow)
  - (Launch Instance)


Openstack volumes
-----------------
1. Create data disk for controller VM and attach
- Volumes > Volumes > (+Create Volume)
  - Volume Name: ran1ctl1hdd1
  - Type: hd-store
  - Size (GB): 128
  - (Create Volume)
- Volumes > Volumes > (ran1ctl1hdd1) > (Actions column dropdown arrow next to `Edit Volume` button)
  - Manage Attachments > Attach To Instance > ran1ctl1 > (Attach Volume)

2. Create docker disk for worker node VM and attach
- Volumes > Volumes > (+Create Volume)
  - Volume Name: ran1node1ssd1
  - Type: ceph-store
  - Size (GB): 16
  - (Create Volume)
- Volumes > Volumes > (ran1node1ssd1) > (dropdown arrow next to `Edit Volume` button)
  - Manage Attachments > Attach To Instance > ran1node1 > (Attach Volume)


Configure controller VM
-----------------------
1. Temporarily enable access to `ran1ctl1`:
- Compute > Instances > (ran1ctl1) > (dropdown arrow next to `Create Snapshot` button) > Edit Security Groups
  - open(+)
  - (Save)
- Compute > Instances > (ran1ctl1) > (dropdown arrow next to `Create Snapshot` button) > Associate Floating IP
  - IP Address: (+)
  - Allocate Floating IP
    - Pool: pub0net
    - (Allocate IP)
  - IP Address: (allocated IP)
  - (Associate)

2. SSH into allocated public IP using your favorite program
  - username: administrator
  - password: (blank)
  - ssh key: (key pair created previously)

3. Setup docker, NFS server and rancher controller:

```bash
wget https://raw.githubusercontent.com/dockerguys/houseparty/master/ubuntu/prep_disk.sh
sudo mkdir /storage
sudo sh ./prep_disk.sh /dev/vdb /storage
wget https://raw.githubusercontent.com/dockerguys/houseparty/master/ubuntu/install_nfs_server.sh
sudo mkdir /storage/nfsvol
sudo sh ./install_nfs_server.sh -s /storage/nfsvol
wget https://raw.githubusercontent.com/dockerguys/houseparty/master/ubuntu/install-docker.sh
sudo sh ./install-docker.sh
sudo reboot
```

4. Verify setup is ok and install rancher controller

```bash
# verify /dev/vdb disk is mounted to /storage
df -h /storage
# verify docker is ok
docker info

# install rancher
docker run -d -v /var/lib/rancher:/var/lib/mysql --restart=unless-stopped -p 8080:8080 --name rancher-server rancher/server
docker logs -f rancher-server
```

5. Wait for a while and then visit http://<allocated_ip>:8080 to enter Rancher web console.

6. Secure your Rancher controller by setting up an admin password.
- admin > access control > (local)
  - Login Username: ranchadmin
  - Full Name: RanchAdmin
  - Password: ChangeMe
  - Confirm Password: ChangeMe
  - (Enable Local Auth)

Configure worker node VM
------------------------
1. Get SSH access to node:
- Compute > Instances > (ran1node1) > (dropdown arrow next to `Create Snapshot` button) > Associate Floating IP
  - IP Address: (+)
  - Allocate Floating IP
    - Pool: pub0net
    - (Allocate IP)
  - IP Address: (allocated IP)
  - (Associate)

2. SSH into allocated public IP using your favorite program
  - username: administrator
  - password: (blank)
  - ssh key: (key pair created previously)

3. Setup docker:

```bash
wget https://raw.githubusercontent.com/dockerguys/houseparty/master/ubuntu/prep_disk.sh
sudo mkdir /storage
sudo sh ./prep_disk.sh /dev/vdb /storage
wget https://raw.githubusercontent.com/dockerguys/houseparty/master/ubuntu/install-docker.sh
sudo mkdir /storage/docker
sudo sh ./install-docker.sh -d /storage/docker
sudo reboot
```

4. Verify setup is ok and install rancher controller

```bash
# verify /dev/vdb disk is mounted to /storage
df -h /storage
# verify docker is ok
docker info
```

Configure Rancher
-----------------
1. Customize settings:
- admin > settings
  - Host Registration URL
    - Something else: http://<private_ip_of_ran1ctl1>:8080
    - (save)
  - Catalog
    - Community Contributed: (disable)
    - (save)

2. Add worker node to cluster
- infrastructure > hosts > (add host) > (custom)
  - Specify the public IP...: <private ip of ran1node1>
  - (follow instruction 5 on screen and execute command on node1)
  - (close)
- (see host appear)

3. Run basic services
- catalog > library > Network Policy Manager
  - (Launch)
- default > edit default > Network Policy
  - Everything else: deny
- catalog > library > rancher nfs
  - nfs server: <private ip of ran1ctl1>
  - export base directory: /storage/nfsvol
- infrastructure > hosts > (ran1node1) > ... > edit
  - labels > (+add label) > io.role.lbs=yes

4. Install certificates
- infrastructure > certificates > (Add certificate)

5. Self reverse proxy
- stacks > user > (Add Stack)
  - Name: loadbalancer
  - Description: Primary loadbalancer
  - (Create)
  - (Add Service) > Add external service
    - Name: control1
    - Add target IP: <private ip of ran1ctl1>
    - (Create)
  - (Add Service) > Add Load balancer
    - Name: lbs
    - Always run one instance ...onevery host
    - Port Rules:
      - Public | HTTPS | rancher.example.com | 443 | loadbalancer/control1 | 8080
  - SSL Termination: (choose cert)
  - Scheduling > (+Add Scheduling Rule)
    - The host must have a host label of io.role.lbs = yes
  - (Create)

6. Integrate with external loadbalancer
Let's say you own example.com. Configure your nameserver to point rancher.example.com to the floating IP address 
assigned to ran1node1. Now you can get to the Rancher console by visiting https://rancher.example.com.


Protect controller
------------------
Disable temporarily enable access to `ran1ctl1`:
- Compute > Instances > (ran1ctl1) > (dropdown arrow next to `Create Snapshot` button) > Edit Security Groups
  - open(-)
  - (Save)
- Compute > Instances > (ran1ctl1) > (dropdown arrow next to `Create Snapshot` button) > Deassociate Floating IP


Adding more worker nodes
------------------------
1. Repeat step 4 of `Openstack VM instances`, substitute `ran1node1` for `ran1node2`, etc.
2. Repeat step 2 of `Openstack volumes`, substitute `ran1node1ssd1` for `ran1node2ssd1`, etc.
3. Repeat `Configure worker node VM`
4. Repeat step 2 of `Configure Rancher`.


Variations
----------
The following variables in the steps above may be substituted:
1. stagenet: alphanumeric. recommends suffix with `net` and not to lead with a number.
2. 192.168.1: any valid private ipv4 subnet. recommends `192.168.x`, where x is between 1 to 200.
3. 192.168.1.0/24: change with (2). you can substitute `/24` with a larger subnet if required.
4. 8.8.8.8\n8.8.4.4: any valid DNS server(s). You can also choose to use the internal DNS servers provided by the cloud platform.
5. pub0net: you are free to choose any other provisioned public network made available to you by the administrators.
6. ran1: this is just a prefix to document your cluster. Any name is fine.
7. 128gb disk on controller: increase or decrease as needed. this disk is shared via nfs by all the worker nodes.
8. 16gb disk on workers: this disk is used by docker for local volumes and images. increase or decrease as needed.
9. /dev/vdb: this should be the mount point for the first data disk, but you can verify using `ls /dev/vd*`.
10. ChangeMe: change this to your own password!!!
11. ranchadmin/RanchAdmin: change this to your desired admin username

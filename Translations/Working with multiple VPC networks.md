# LAB: AK8S-02 Working with Cloud Build

## OBJECTIVES
In this lab, you learn how to perform the following tasks:

1. Create custom mode VPC networks with firewall rules.

2. Create VM instances using Compute Engine.

3. Explore the connectivity for VM instances across VPC networks.

4. Create a VM instance with multiple network interfaces.

# Task 0: Sign in to the Google Cloud Platform (GCP) Console

For each lab, you get a new GCP project and set of resources for a fixed time at no cost.

1. Make sure you signed into Qwiklabs using an **incognito window**.

2. When ready, click ![start_lab](https://cdn.qwiklabs.com/XE8x7uvQokyubNwnYKKc%2BvBBNrMlo5iNZiDDzQQ3Ddo%3D).

2. Click **Open Google Console**.

3. Click **Use another account** and copy/paste credentials for this lab into the prompts.

4. Accept the terms and skip the recovery resource page.



# Task 1. Create custom mode VPC networks with firewall rules
Create two custom networks, managementnet and privatenet, along with firewall rules to allow SSH, ICMP, and RDP ingress traffic.

1. In GCP console, on the top right toolbar, click the Open Cloud Shell button.
![Cloud Shell icon](https://cdn.qwiklabs.com/vdY5e%2Fan9ZGXw5a%2FZMb1agpXhRGozsOadHURcR8thAQ%3D)


2. Click **Continue**. ![Continue icon](https://cdn.qwiklabs.com/lr3PBRjWIrJ%2BMQnE8kCkOnRQQVgJnWSg4UWk16f0s%2FA%3D)

## Create the managementnet network
1. Create the managementnet network using cloudshell in custom mode
```console
gcloud compute networks create managementnet --subnet-mode=custom
```

2. Create a subnet in us-central1
```console
gcloud compute networks subnets create managementsubnet-us --network=managementnet --region=us-central1 --range=10.130.0.0/20
```

## Create the privatenet network
Create the privatenet network using the gcloud command line.
1. Run the following command to create the privatenet network:

```console
gcloud compute networks create privatenet --subnet-mode=custom
```

2. Run the following command to create the privatesubnet-us subnet:

```console
gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24
```

3. Run the following command to create the privatesubnet-eu subnet:

```console
gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20
```

4. Run the following command to list the available VPC networks:

```console
gcloud compute networks list
```
5. Run the following command to list the available VPC subnets (sorted by VPC network):

```console
gcloud compute networks subnets list --sort-by=NETWORK
```

## Create the firewall rules for managementnet
Create firewall rules to allow SSH, ICMP, and RDP ingress traffic to VM instances on the managementnet network.

```console
gcloud compute firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
```
## Create the firewall rules for privatenet
1. Create the firewall rules for privatenet network using the gcloud command line.
```console
gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
```
2. Run the following command to list all the firewall rules (sorted by VPC network):

```console
gcloud compute firewall-rules list --sort-by=NETWORK
```
# Task 2. Create VM instances
Create two VM instances:

1. managementnet-us-vm in managementsubnet-us

2. privatenet-us-vm in privatesubnet-us

## Create the managementnet-us-vm instance
```console
gcloud compute instances create managementnet-us-vm --zone=us-central1-c --machine-type=f1-micro --subnet=managementsubnet-us
```
## Create the privatenet-us-vm instance
```console
gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=privatesubnet-us
```

# Task 3. Explore the connectivity between VM instances
Explore the connectivity between the VM instances. Specifically, determine the effect of having VM instances in the same zone versus having instances in the same VPC network.

## Ping the external IP addresses
Ping the external IP addresses of the VM instances to determine whether you can reach the instances from the public internet.
1. ssh into the mynet-us-vm
```console
gcloud compute ssh --zone=us-central1-c mynet-us-vm -q
```

2. To test connectivity to mynet-eu-vm's external IP, run the following command, replacing mynet-eu-vm's external IP:

```console
ping -c 3 <Enter mynet-eu-vm's external IP here>
```

3. To test connectivity to managementnet-us-vm's external IP, run the following command, replacing managementnet-us-vm's external IP:

```console
ping -c 3 <Enter managementnet-us-vm's external IP here>
```

4. To test connectivity to privatenet-us-vm's external IP, run the following command, replacing privatenet-us-vm's external IP:

```console
ping -c 3 <Enter privatenet-us-vm's external IP here>
```
## Ping the internal IP addresses
Ping the internal IP addresses of the VM instances to determine whether you can reach the instances from the public internet.
1. ssh into the mynet-us-vm
```console
gcloud compute ssh --zone=us-central1-c mynet-us-vm -q
```

2. To test connectivity to mynet-eu-vm's internal IP, run the following command, replacing mynet-eu-vm's internal IP:

```console
ping -c 3 <Enter mynet-eu-vm's internal IP here>
```

3. To test connectivity to managementnet-us-vm's internal IP, run the following command, replacing managementnet-us-vm's internal IP:

```console
ping -c 3 <Enter managementnet-us-vm's internal IP here>
```

4. To test connectivity to privatenet-us-vm's internal IP, run the following command, replacing privatenet-us-vm's internal IP:

```console
ping -c 3 <Enter privatenet-us-vm's internal IP here>
```

# Task 4. Create a VM instance with multiple network interfaces
Every instance in a VPC network has a default network interface. You can create additional network interfaces attached to your VMs. Multiple network interfaces enable you to create configurations in which an instance connects directly to several VPC networks (up to 8 interfaces, depending on the instance's type).

## Create the VM instance with multiple network interfaces
1. Create the vm-appliance instance with network interfaces in privatesubnet-us, managementsubnet-us, and mynetwork. The CIDR ranges of these subnets do not overlap, which is a requirement for creating a VM with multiple network interface controllers (NICs).
```console
gcloud compute instances create vm-appliance --zone=us-central1-c --machine-type=n1-standard-4 --network-interface subnet=privatesubnet-us --network-interface subnet=managementsubnet-us --network-interface subnet=mynetwork
```
2. ssh into the created vm-appliance and check its iterfaces using the command:
```console
sudo ifconfig
```

## Explore the network interface connectivity
Demonstrate that the vm-appliance instance is connected to privatesubnet-us, managementsubnet-us, and mynetwork by pinging VM instances on those subnets.

1. ssh into the vm-appliance VM

2. To test connectivity to privatenet-us-vm's internal IP, run the following command, replacing privatenet-us-vm's internal IP:

```console
ping -c 3 <Enter privatenet-us-vm's internal IP here>
```

3. Repeat the same test by running the following:

```console
ping -c 3 privatenet-us-vm
```
You can ping privatenet-us-vm by its name because VPC networks have an internal DNS service that allows you to address instances by their DNS names instead of their internal IP addresses. When an internal DNS query is made with the instance hostname, it resolves to the primary interface (nic0) of the instance. Therefore, this only works for privatenet-us-vm in this case.

4. To test connectivity to managementnet-us-vm's internal IP, run the following command, replacing managementnet-us-vm's internal IP:

```console
ping -c 3 <Enter managementnet-us-vm's internal IP here>
```
This works!

5. To test connectivity to mynet-us-vm's internal IP, run the following command, replacing mynet-us-vm's internal IP:

```console
ping -c 3 <Enter mynet-us-vm's internal IP here>
```
This works!

6. To test connectivity to mynet-eu-vm's internal IP, run the following command, replacing mynet-eu-vm's internal IP:

```console
ping -c 3 <Enter mynet-eu-vm's internal IP here>
```
This does not work! In a multiple interface instance, every interface gets a route for the subnet that it is in. In addition, the instance gets a single default route that is associated with the primary interface eth0. Unless manually configured otherwise, any traffic leaving an instance for any destination other than a directly connected subnet will leave the instance via the default route on eth0.

7. To list the routes for vm-appliance instance, run the following command:

```console
ip route
```
# End your lab
When you have completed your lab, click End Lab. Qwiklabs removes the resources you’ve used and cleans the account for you.

You will be given an opportunity to rate the lab experience. Select the applicable number of stars, type a comment, and then click Submit.
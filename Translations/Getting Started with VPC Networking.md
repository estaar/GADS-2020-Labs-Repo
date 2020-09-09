# LAB: Getting Started with VPC Networking
## OBJECTIVES: 
1. Explore the default VPC network
2. Create an auto mode network with firewall rules
3. Create VM instances using Compute Engine
4. Explore the connectivity for VM instances

# Task 1. Explore the default network
Start Cloud Shell
## View the subnets in the default network:
The default network has a subnet in each Google Cloud region. Enter the command below to see the subnets

``` console
gcloud compute networks subnets list --network=default
```
## View the routes
Routes tell VM instances and the VPC network how to send traffic from an instance to a destination, either inside the network or outside Google Cloud. Each VPC network comes with some default routes to route traffic among its subnets and send traffic from eligible instances to the internet.
``` console
gcloud compute routes list
```

## View the firewall rules
Each VPC network implements a distributed virtual firewall that you can configure. Firewall rules allow you to control which packets are allowed to travel to which destinations. Every VPC network has two implied firewall rules that block all incoming connections and allow all outgoing connections.
``` console
gcloud compute firewall-rules list
```
        
## Delete all the Firewall Rules listed.
Enter the command below to delete firewall rules. Use -q flag when you want to escape the flag to choose yes or no.

``` console
gcloud compute firewall-rules delete default-allow-icmp default-allow-rdp default-allow-ssh default-allow-internal -q
```
    
## Delete the default network
```console
gcloud compute networks delete default -q
```
    
## Try to create a VM- instance
Verify that you cannot create a VM instance without a VPC network.

``` console 
gcloud compute instances create mynet-eu-vm-test --zone=europe-west1-c --machine-type=f1-micro --image=debian-10-buster-v20200902 --image-project=debian-cloud
```
Creation fails when you do not have a network with:
> Invalid value for field 'resource.networkInterfaces[0].network'

# Task 2. Create a VPC network and VM instances
Create a VPC network so that you can create VM instances.

## Create an Auto mode VPC network called mynetwork and add all avalilable firewall rules:
``` console gcloud compute networks create mynetwork --subnet-mode=auto --bgp-routing-mode=global

gcloud compute firewall-rules create mynetwork-allow-icmp --network=mynetwork --direction=INGRESS --priority=65534 --action=ALLOW --rules=icmp

gcloud compute firewall-rules create mynetwork-allow-internal --network=mynetwork --direction=INGRESS --priority=65534 --action=ALLOW --rules=all

gcloud compute firewall-rules create mynetwork-allow-rdp --network=mynetwork --direction=INGRESS --priority=65534 --action=ALLOW --rules=tcp:3389

gcloud compute firewall-rules create mynetwork-allow-ssh --network=mynetwork --direction=INGRESS --priority=65534 --action=ALLOW --rules=tcp:22
```
    
## View subnets specifically for Europe-wes1 and us-central1 regions
```console
gcloud compute networks subnets list --filter="region:( us-central1 europe-west1 )" --network=mynetwork
```
    
## Create a VM instance in us-central1 region
``` console 
gcloud compute instances create mynet-us-vm --zone=us-central1-c --machine-type=f1-micro --image=debian-10-buster-v20200902 --image-project=debian-cloud --network=mynetwork
```
    
## Create a VM instance in europe-west1 region
``` console
gcloud compute instances create mynet-eu-vm --zone=europe-west1-c --machine-type=f1-micro --image=debian-10-buster-v20200902 --image-project=debian-cloud --network=mynetwork
```
    
# Task 3. Explore the connectivity for VM instances
Explore the connectivity for the VM instances. Specifically, try to SSH to your VM instances using tcp:22, and ping both the internal and external IP addresses of your VM instances using ICMP. Then explore the effects of the firewall rules on connectivity by removing the firewall rules individually.

## Verify connectivity for the VM instances
1. Save the external ip and internal ip address of mynet-eu-vm in a shell variable

``` console
MYNET_EU_VM_EXTERNAL_IP=$(gcloud compute instances describe mynet-eu-vm --zone=europe-west1-c --format='get(networkInterfaces[0].accessConfigs[0].natIP)')
MYNET_EU_VM_INTERNAL_IP=$(gcloud compute instances describe mynet-eu-vm --zone=europe-west1-c --format='get(networkInterfaces[0].networkIP)')
```
    
2. Echo to see the external and internal ips

``` console
echo $MYNET_EU_VM_INTERNAL_IP
echo $MYNET_EU_VM_EXTERNAL_IP
```

3. ssh into the mynet-us-vm via the console or via the cloudshell terminal.

``` console
gcloud compute ssh --zone=us-central1-c mynet-us-vm -q
```

4. Test connectivity to mynet-eu-vm internal ip address by typing the below commands in the vm ssh. Remember to replace <result of $MYNET_EU_VM_INTERNAL_IP> with the relevant data.

``` console
ping -c 3 <result of $MYNET_EU_VM_INTERNAL_IP>

ping -c 3 mynet-eu-vm

ping -c 3 <result of $MYNET_EU_VM_EXTERNAL_IP>
```
    
## Remove the allow-icmp firewall rules
Remove the allow-icmp firewall rule and try to ping the internal and external IP address of mynet-eu-vm.
1. Switch to the cloud console terminal  to delete the firewall.

``` console
gcloud compute firewall rules delete mynetwork-allow-icmp -q
```

2. Switch to the vm ssh window and type the commands remember to change  <result of $MYNET_EU_VM_INTERNAL_IP> to the VM in EU internal ip and same case to External IP.

``` console
ping -c 3 <result of $MYNET_EU_VM_INTERNAL_IP>
ping -c 3 <result of $MYNET_EU_VM_EXTERNAL_IP>
```
    
## Remove the allow-internal firewall rules
Remove the allow-internal firewall rule and try to ping the internal IP address of mynet-eu-vm.
1. Switch to the cloud console terminal and enter the following command
``` console
gcloud compute firewall rules delete mynetwork-allow-internal -q
```
2. Switch to the vm ssh window and type the commands remember to change  <result of $MYNET_EU_VM_INTERNAL_IP> to the VM in EU internal ip and same case to External IP
3. To test connectivity to mynet-eu-vm's internal IP, run the following command, replacing mynet-eu-vm's internal IP:

``` console
ping -c 3 <result of $MYNET_EU_VM_INTERNAL_IP>
```

3. Exit the ssh terminal:
``` console
exit
```

## Remove the allow-ssh firewall rules
Remove the allow-ssh firewall rule and try to SSH to mynet-us-vm.
1. Switch to the cloud console terminal
2. Remove the allow-ssh firewall rule and try to ssh into mynet-us-vm:

``` console
gcloud compute firewall rules delete mynetwork-allow-ssh -q
```

3. Try to ssh into  the europe-west1-vm
``` console
gcloud compute ssh --zone=europe-west1-c mynet-eu-vm
```
# Task 4. Review
In this lab, you explored the default network along with its subnets, routes, and firewall rules. You deleted the default network and determined that you cannot create any VM instances without a VPC network. Thus, you created a new auto mode VPC network with subnets, routes, firewall rules, and two VM instances. Then you tested the connectivity for the VM instances and explored the effects of the firewall rules on connectivity.

# End your lab
When you have completed your lab, click End Lab. Qwiklabs removes the resources you’ve used and cleans the account for you.

You will be given an opportunity to rate the lab experience. Select the applicable number of stars, type a comment, and then click Submit.

    


    


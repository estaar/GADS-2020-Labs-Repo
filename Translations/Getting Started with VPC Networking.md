# LAB: Getting Started with VPC Networking
# OBJECTIVES: 
#    1. Explore the default VPC network
#    2. Create an auto mode network with firewall rules
#    3. Create VM instances using Compute Engine
#    4. Explore the connectivity for VM instances

# View subnets in the default network:
# start cloud shell
gcloud compute networks subnets list --network=default
# View the routes
gcloud compute routes list

# View all the compute engine firewall rules
gcloud compute firewall-rules list
        
# Delete all the Firewall Rules listed. Use -q when you want to escape the flag to choose yes or no
gcloud compute firewall-rules delete default-allow-icmp default-allow-rdp default-allow-ssh default-allow-internal -q
    
# Delete the default network
gcloud compute networks delete default -q
    
# Try to create a VM- instance
gcloud compute instances create mynet-eu-vm-test --zone=europe-west1-c --machine-type=f1-micro --image=debian-10-buster-v20200902 --image-project=debian-cloud
# Fails when you do not have a network with
# Invalid value for field 'resource.networkInterfaces[0].network'

# Create an Auto mode VPC network called mynetwork and add all avalilable firewall rules:
gcloud compute networks create mynetwork --subnet-mode=auto --bgp-routing-mode=global
gcloud compute firewall-rules create mynetwork-allow-icmp --network=mynetwork --direction=INGRESS --priority=65534 --action=ALLOW --rules=icmp
gcloud compute firewall-rules create mynetwork-allow-internal --network=mynetwork --direction=INGRESS --priority=65534 --action=ALLOW --rules=all
gcloud compute firewall-rules create mynetwork-allow-rdp --network=mynetwork --direction=INGRESS --priority=65534 --action=ALLOW --rules=tcp:3389
gcloud compute firewall-rules create mynetwork-allow-ssh --network=mynetwork --direction=INGRESS --priority=65534 --action=ALLOW --rules=tcp:22
    
# View subnets specifically for Europe-wes1 and us-central1 regions
gcloud compute networks subnets list --filter="region:( us-central1 europe-west1 )" --network=mynetwork
    
# Create a VM instance in us-central1 region
gcloud compute instances create mynet-us-vm --zone=us-central1-c --machine-type=f1-micro --image=debian-10-buster-v20200902 --image-project=debian-cloud --network=mynetwork
    
# Create a VM instance in europe-west1 region
gcloud compute instances create mynet-eu-vm --zone=europe-west1-c --machine-type=f1-micro --image=debian-10-buster-v20200902 --image-project=debian-cloud --network=mynetwork
    
# Test connectivity between the Virtual Machines
# save the external ip and internal ip address of mynet-eu-vm in a shell variable
MYNET_EU_VM_EXTERNAL_IP=$(gcloud compute instances describe mynet-eu-vm --zone=europe-west1-c --format='get(networkInterfaces[0].accessConfigs[0].natIP)')
MYNET_EU_VM_INTERNAL_IP=$(gcloud compute instances describe mynet-eu-vm --zone=europe-west1-c --format='get(networkInterfaces[0].networkIP)')
    
# echo to see the external and internal ips
echo $MYNET_EU_VM_INTERNAL_IP
echo $MYNET_EU_VM_EXTERNAL_IP

# ssh into the mynet-us-vm via the console 
# gcloud compute ssh --zone=us-central1-c mynet-us-vm -q

# Test connectivity to mynet-eu-vm internal ip address by typing the below commands in the vm ssh
ping -c 3 <result of $MYNET_EU_VM_INTERNAL_IP>

ping -c 3 mynet-eu-vm

ping -c 3 <result of $MYNET_EU_VM_EXTERNAL_IP>
    
# Remove the allow-icmp firewall rule and try to ping the internal and external IP address of mynet-eu-vm
# switch to the cloud console terminal
gcloud compute firewall rules delete mynetwork-allow-icmp -q

# switch to the vm ssh window and type the commands remember to change  <result of $MYNET_EU_VM_INTERNAL_IP> to the VM in EU internal ip and same case to External IP
ping -c 3 <result of $MYNET_EU_VM_INTERNAL_IP>
ping -c 3 <result of $MYNET_EU_VM_EXTERNAL_IP>
    
# Remove the allow-internal firewall rule and try to ping the internal
# switch to the cloud console terminal
gcloud compute firewall rules delete mynetwork-allow-internal -q

# switch to the vm ssh window and type the commands remember to change  <result of $MYNET_EU_VM_INTERNAL_IP> to the VM in EU internal ip and same case to External IP
ping -c 3 <result of $MYNET_EU_VM_INTERNAL_IP>

exit ssh
    exit

# switch to the cloud console terminal
# Remove the allow-ssh firewall rule and try to ssh into mynet-us-vm
gcloud compute firewall rules delete mynetwork-allow-ssh -q

# Try to ssh into 
gcloud compute ssh --zone=europe-west1-c mynet-eu-vm

# Fails

    


    


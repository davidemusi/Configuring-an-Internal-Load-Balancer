arrow_back
Configuring an Internal Load Balancer
avatar image
—/40
Checkpoints
Configure internal traffic and health check firewall rules

Check my progress
/ 10

Create a NAT configuration using Cloud Router

Check my progress
/ 10

Configure instance templates and create instance groups

Check my progress
/ 10

Configure the internal load balancer

Check my progress
/ 10

01:30:00
Overview
Task 1. Configure internal traffic and health check firewall rules.
Task 2: Create a NAT configuration using Cloud Router
Task 3. Configure instance templates and create instance groups
Task 4. Configure the internal load balancer
Task 5. Test the internal load balancer
Task 6. Review
End your lab
Configuring an Internal Load Balancer
1 hour 30 minutes
Free
Rate Lab
Overview
Google Cloud offers Internal Load Balancing for your TCP/UDP-based traffic. Internal Load Balancing enables you to run and scale your services behind a private load balancing IP address that is accessible only to your internal virtual machine instances.

In this lab, you create two managed instance groups in the same region. Then you configure and test an internal load balancer with the instances groups as the backends, as shown in this network diagram:

network_diagram.png

Objectives
In this lab, you learn how to perform the following tasks:

Create internal traffic and health check firewall rules
Create a NAT configuration using Cloud Router
Configure two instance templates
Create two managed instance groups
Configure and test an internal load balancer
For each lab, you get a new GCP project and set of resources for a fixed time at no cost.

Make sure you signed into Qwiklabs using an incognito window.

Note the lab's access time (for example, img/time.png and make sure you can finish in that time block.

There is no pause feature. You can restart if needed, but you have to start at the beginning.

When ready, click img/start_lab.png.

Note your lab credentials. You will use them to sign in to Cloud Platform Console. img/open_google_console.png

Click Open Google Console.

Click Use another account and copy/paste credentials for this lab into the prompts.

If you use other credentials, you'll get errors or incur charges.

Accept the terms and skip the recovery resource page.
Do not click End Lab unless you are finished with the lab or want to restart it. This clears your work and removes the project.

Task 1. Configure internal traffic and health check firewall rules.
Configure firewall rules to allow internal traffic connectivity from sources in the 10.10.0.0/16 range. This rule allows incoming traffic from any client located in the subnet.

Health checks determine which instances of a load balancer can receive new connections. For HTTP load balancing, the health check probes to your load-balanced instances come from addresses in the ranges 130.211.0.0/22 and 35.191.0.0/16. Your firewall rules must allow these connections.

Explore the my-internal-app network
The network my-internal-app with subnet-a and subnet-b and firewall rules for RDP, SSH, and ICMP traffic have been configured for you.

In the Cloud Console, on the Navigation menu (Navigation menu), click VPC network > VPC networks. Notice the my-internal-app network with its subnets: subnet-a and subnet-b.

Each Google Cloud project starts with the default network. In addition, the my-internal-app network has been created for you as part of your network diagram.

You will create the managed instance groups in subnet-a and subnet-b. Both subnets are in the us-central1 region because an internal load balancer is a regional service. The managed instance groups will be in different zones, making your service immune to zonal failures.

Create the firewall rule to allow traffic from any sources in the 10.10.0.0/16 range
Create a firewall rule to allow traffic in the 10.10.0.0/16 subnet.

On the Navigation menu (Navigation menu), click VPC network > Firewall. Notice the app-allow-icmp and app-allow-ssh-rdp firewall rules.

These firewall rules have been created for you.

Click Create Firewall Rule.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	fw-allow-lb-access
Network	my-internal-app
Targets	Specified target tags
Target tags	backend-service
Source filter	IP Ranges
Source IP ranges	10.10.0.0/16
Protocols and ports	Allow all
Make sure to include the /16 in the Source IP ranges.

Click Create.

Create the health check rule
Create a firewall rule to allow health checks.

On the Navigation menu (Navigation menu), click VPC network > Firewall.

Click Create Firewall Rule.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	fw-allow-health-checks
Network	my-internal-app
Targets	Specified target tags
Target tags	backend-service
Source filter	IP Ranges
Source IP ranges	130.211.0.0/22 35.191.0.0/16
Protocols and ports	Specified protocols and ports
Make sure to include the /22 and /16 in the Source IP ranges.

For tcp, specify port 80.
Click Create.
Click Check my progress to verify the objective.
Configure internal traffic and health check firewall rules

Task 2: Create a NAT configuration using Cloud Router
The Google Cloud VM backend instances that you setup in Task 3 will not be configured with external IP addresses.

Instead, you will setup the Cloud NAT service to allow these VM instances to receive incoming traffic only through the Cloud NAT, and return traffic from the outbound NAT through the load balancer.

Create the Cloud Router instance
In the Cloud Console, on the Navigation menu (Navigation menu), click Network services > Cloud NAT.

Click Get started.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Gateway name	nat-config
VPC network	my-internal-app
Region	us-central1
Click Cloud Router, and select Create new router.

For Name, type nat-router-us-central1.

Click Create.

In Create a NAT gateway, click Create.

Wait until the NAT Gateway Status changes to Running before moving onto the next task.

Click Check my progress to verify the objective.
Create a NAT configuration using Cloud Router

Task 3. Configure instance templates and create instance groups
A managed instance group uses an instance template to create a group of identical instances. Use these to create the backends of the internal load balancer.

Configure the instance templates
An instance template is an API resource that you can use to create VM instances and managed instance groups. Instance templates define the machine type, boot disk image, subnet, labels, and other instance properties. Create an instance template for both subnets of the my-internal-app network.

On the Navigation menu (Navigation menu), click Compute Engine > Instance templates.

Click Create instance template.

For Name, type instance-template-1

Click Management, security, disks, networking, sole tenancy.

Click Management.

Under Metadata, specify the following:

Key	Value
startup-script-url	gs://cloud-training/gcpnet/ilb/startup.sh
The startup-script-url specifies a script that is executed when instances are started. This script installs Apache and changes the welcome page to include the client IP and the name, region, and zone of the VM instance. You can explore this script here.

Click Networking.

For Network interfaces, specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Network	my-internal-app
Subnetwork	subnet-a
Network tags	backend-service
External IP	None
The network tag backend-service ensures that the firewall rule to allow traffic from any sources in the 10.10.0.0/16 subnet and the Health Check firewall rule applies to these instances.

Click Create. Wait for the instance template to be created.

Create another instance template for subnet-b by copying instance-template-1:

Select the instance-template-1 and click Copy.

Click Management, security, disks, networking, sole tenancy.

Click Networking.

For Network interfaces, select subnet-b as the Subnetwork.

Click Create.

Create the managed instance groups
Create a managed instance group in subnet-a (us-central1-a) and subnet-b (us-central1-b).

On the Navigation menu (Navigation menu), click Compute Engine > Instance groups.

Click Create Instance group.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	instance-group-1
Location	Single-zone
Region	us-central1
Zone	us-central1-a
Instance template	instance-template-1
Autoscaling metrics > metrics type	CPU utilization
Target CPU utilization	80
Minimum number of instances	1
Maximum number of instances	5
Cool-down period	45
Managed instance groups offer autoscaling capabilities that allow you to automatically add or remove instances from a managed instance group based on increases or decreases in load. Autoscaling helps your applications gracefully handle increases in traffic and reduces cost when the need for resources is lower. Just define the autoscaling policy, and the autoscaler performs automatic scaling based on the measured load.

Click Create.

Repeat the same procedure for instance-group-2 in us-central1-b:

Click Create Instance group.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	instance-group-2
Location	Single-zone
Region	us-central1
Zone	us-central1-b
Instance template	instance-template-2
Autoscaling metrics > metric type	CPU utilization
Target CPU utilization	80
Minimum number of instances	1
Maximum number of instances	5
Cool-down period	45
Click Create.

Verify the backends
Verify that VM instances are being created in both subnets and create a utility VM to access the backends' HTTP sites.

On the Navigation menu, click Compute Engine > VM instances. Notice two instances that start with instance-group-1 and instance-group-2.

These instances are in separate zones, and their internal IP addresses are part of the subnet-a and subnet-b CIDR blocks.

Click Create Instance.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	utility-vm
Region	us-central1
Zone	us-central1-f
Machine type	f1-micro (1 vCPU)
Boot disk	Debian GNU/Linux 10 (buster)
Click Management, security, disks, networking, sole tenancy.

Click Networking.

For Network interfaces, click the pencil icon to edit.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Network	my-internal-app
Subnetwork	subnet-a
Primary internal IP	Ephemeral (Custom)
Custom ephemeral IP address	10.10.20.50
External IP	None
Click Done.

Click Create.

Note that the internal IP addresses for the backends are 10.10.20.2 and 10.10.30.2.

If these IP addresses are different, replace them in the two curl commands below.

Click Check my progress to verify the objective.
Configure instance templates and create instance groups

For utility-vm, click SSH to launch a terminal and connect. If you see the Connection via Cloud Identity-Aware Proxy Failed popup, click Retry.

To verify the welcome page for instance-group-1-xxxx, run the following command:

curl 10.10.20.2

The output should look like this (do not copy; this is example output):

<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.20.50<h2>Hostname</h2>Server Hostname:
 instance-group-1-1zn8<h2>Server Location</h2>Region and Zone: us-central1-a

To verify the welcome page for instance-group-2-xxxx, run the following command:

curl 10.10.30.2

The output should look like this (do not copy; this is example output):

<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.20.50<h2>Hostname</h2>Server Hostname:
 instance-group-2-q5wp<h2>Server Location</h2>Region and Zone: us-central1-b

Which of these fields identify the location of the backend?

Client IP

Server Location

Server Hostname

This will be useful when verifying that the internal load balancer sends traffic to both backends.

Close the SSH terminal to utility-vm:

exit

Task 4. Configure the internal load balancer
Configure the internal load balancer to balance traffic between the two backends (instance-group-1 in us-central1-a and instance-group-2 in us-central1-b), as illustrated in the network diagram:

network_diagram.png

Start the configuration
In the Cloud Console, on the Navigation menu (Navigation menu), click Network Services > Load balancing.
Click Create load balancer.
Under TCP Load Balancing, click Start configuration.
For Internet facing or internal only, select Only between my VMs.
Choosing Only between my VMs makes this load balancer internal. This choice requires the backends to be in a single region (us-central1) and does not allow offloading TCP processing to the load balancer.

Click Continue.

For Name, type my-ilb.

Configure the regional backend service
The backend service monitors instance groups and prevents them from exceeding configured usage.

Click Backend configuration.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (select option as specified)
Region	us-central1
Network	my-internal-app
Instance group	instance-group-1 (us-central1-a)
Click Done.

Click Add backend.

For Instance group, select instance-group-2 (us-central1-b).

Click Done.

For Health Check, select Create a health check.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (select option as specified)
Name	my-ilb-health-check
Protocol	TCP
Port	80
Health checks determine which instances can receive new connections. This HTTP health check polls instances every 5 seconds, waits up to 5 seconds for a response, and treats 2 successful or 2 failed attempts as healthy or unhealthy, respectively.

Click Save and Continue.

Verify that there is a blue check mark next to Backend configuration in the Cloud Console. If there isn't, double-check that you have completed all the steps above.

Configure the frontend
The frontend forwards traffic to the backend.

Click Frontend configuration.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Subnetwork	subnet-b
Internal IP	Reserve a static internal IP address
Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	my-ilb-ip
Static IP address	Let me choose
Custom IP address	10.10.30.5
Click Reserve.

For Ports, type 80.

Click Done.

Review and create the internal load balancer
Click Review and finalize.
Review the Backend and Frontend.
Click Create. Wait for the load balancer to be created before moving to the next task.
Click Check my progress to verify the objective.
Configure the Internal Load Balancer

Task 5. Test the internal load balancer
Verify that the my-ilb IP address forwards traffic to instance-group-1 in us-central1-a and instance-group-2 in us-central1-b.

Access the internal load balancer
On the Navigation menu, click Compute Engine > VM instances.

For utility-vm, click SSH to launch a terminal and connect.

To verify that the internal load balancer forwards traffic, run the following command:

curl 10.10.30.5

The output should look like this (do not copy; this is example output):

<h1>Internal Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.10.20.50<h2>Hostname</h2>Server Hostname:
 instance-group-1-1zn8<h2>Server Location</h2>Region and Zone: us-central1-a

As expected, traffic is forwarded from the internal load balancer (10.10.30.5) to the backend.

Run the same command a couple of times:

curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5

You should be able to see responses from instance-group-1 in us-central1-a and instance-group-2 in us-central1-b. If not, run the command again.

Task 6. Review
In this lab, you created two managed instance groups in the us-central1 region and a firewall rule to allow HTTP traffic to those instances and TCP traffic from the Google Cloud health checker. Then you configured and tested an internal load balancer for those instance groups.

End your lab
When you have completed your lab, click End Lab. Qwiklabs removes the resources you’ve used and cleans the account for you.

You will be given an opportunity to rate the lab experience. Select the applicable number of stars, type a comment, and then click Submit.

The number of stars indicates the following:

1 star = Very dissatisfied
2 stars = Dissatisfied
3 stars = Neutral
4 stars = Satisfied
5 stars = Very satisfied
You can close the dialog box if you don't want to provide feedback.

For feedback, suggestions, or corrections, please use the Support tab.

Copyright 2020 Google LLC All rights reserved. Google and the Google logo are trademarks of Google LLC. All other company and product names may be trademarks of the respective companies with which they are associated.



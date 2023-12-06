# CNE430Final
Setting up an autoscaling webserver using AWS.

## Description
In an attempt to make AWS “do something” Matt Bennett and Co. (it’s just me - I’m the Matt Bennett and the Co) will harness their technological powers to animate into existence cloned EC2 webservers under the umbrella of a VPC (virtual private cloud) using an ALB (application load balancer) to coordinate their efforts into rallying to the aid of their distressed brethren. After fulfilling their duties, these valiant EC2 instances shall return to their slumber, awaiting the next fateful call. 
The actual deliverables are as follows: 
1. [Create VPC](#1-creating-and-setting-up-the-vpc-and-additional-necessary-subgroups). Set up necessary routing and security groups.
2. [Create an EC2 instance within the VPC](#2-creating-an-ec2-instance-as-a-webserver). Set it up as a webserver (Wordpress most likely).
3. [Create an ASG (autoscaling group)](#3-creating-the-asg). Use AMI of instance created in deliverable #2. Define scaling policies.
4. [Create an ALB (application load balancer)](#4-creating-the-alb). Associated with the VPC and ASG.
5. [Devise a way to stress the webserver sufficiently to trigger autoscaling](#5-stressing-the-server).

## Walkthrough
### 1. Creating and setting up the VPC (and additional necessary subgroups):
We need a private environment where all the instances for the group will be housed. This is where the Virtual Private Cloud comes in.
1. Log in to your AWS account.
2. Navigate to the VPC console. Select "Create VPC"
3. Under "Resources to Create" select VPC and more.
4. Choose a name and CIDR block. 10.0.0.0/16 is standard and will provide you with all the addresses you could want (65536).
5. Choose your zones and number of available subnets. Note: You must have a minimum of two public subnets in two different zones to attach the load balancer.
6. Press "Create VPC"
This should create your own individual VPC with a routing table, internet gateway and two (or more) subnets within your CIDR block. If it did not, you can create them yourself, just make sure that you are associating them all with the VPC that you have just created.

### 2. Creating an EC2 instance as a webserver:
We need a webserver which will serve as the clone for when we create new instances.
1. Create an EC2 instance from the EC2 Dashboard. For my setup I used Amazon Linux 2023, a t2.micro instance, and 12gbs storage. The security group does not matter, but you must be able to SSH into the instance to set up the webserver. You will create a security group later that the template will use with the load balancer.
2. Follow this fabulous [guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hosting-wordpress-aml-2023.html) to set up your Wordpress server.
3. Once you are happy with your instance, you need to create an image of the instance to be used as a template. To do so, find and select your instance. In the top right select Actions -> Images and Templates -> Create Image. Give it a name and press "Create Image".
4. You can now create a template with the image of the instance. From the EC2 dashboard under Launch Templates, select Create Launch Template.
5. Make sure to select the image you just created as the AMI for the template under "My AMIs".
6. Under security group, you want to create your own security group. Make sure that it available to all the ports, protocols and IPs that you expect to need for inbound and outbound traffic. Also, assign it to the VPC that you created earlier.
7. Hit "Create Template".
We now have the template that we will use when setting up the ASG. Going forward if you decide to make any changes to the template, make sure that you choose the latest one and update the default to reflect changes.

### 3. Creating the ASG:
The auto scaling group itself. This will take our template and spin up new instances as necessary.
1. At the bottom of the EC2 dashboard under "Auto Scaling" is "Auto Scaling Groups". Here click on "Create Auto Scaling Group" in the top right.
2. Give the group a name and assign the template we created.
3. Assign it to the VPC we created in step one along with the subnets and zones that we chose.
4. For the moment, we are not going to assign a load balancer. Choose "No load balancer"
5. Choose your desired group size and the minimum and maximum sizes. Desired group size is what the balancer will default to. It will not go below the minimum or above the maximum when creating instances in response to stimulae.
6. I did not add any additional notifications, but now is when you would do it if desired.
7. After reviewing your choices, click "Create Auto Scaling Groups".
You now have the ASG but no way of directing traffic to it. You can also set up dynamic scaling options at this point which will scale up and down your instances based on triggers.

### 4. Creating the ALB:
The load balancer will direct traffic between the different instances as necessary within our VPC.
1. Before we set up the ALB we need to create a target group. This will let the load balancer know which instances are available for redirecting to. Near the bottom of the EC2 dashboard sidebar is "Load Balancing" and under that is "Target Groups". Go there and in the top right choose "Create Target Group".
2. Make sure you set it to target instances, your VPC and use HTTP port 80. Do not specify any instances, this will cause it direct to any instance under your VPC umbrella.
3. Create the target group.
4. We can now make the ALB. Above target group in the sidebar is "Load Balancers". From there, top right "Create Load Balancer".
5. We are using an Application Load Balancer. It should be internet facing with IPv4. Map to your VPC and the subnets in the ASG. Listening should happen on HTTP port 80 with the redirect pointed to the target group we created. The security group should be the same as the one created for the template.
6. Click "Create Load Balancer".
If we've done this correctly, we should be able to access the website we hosted through the ALB. Copy the DNS assigned the Load Balancer under the details page and paste it into your web browser. We should be able to see our website.

### 5. Stressing the Server:
To see if our ASG is working, we need a way to stress the server. For this I have chosen to use Apache JMeter. It provides us with a GUI which I appreciate for visualizing this sort of information.
1. Make sure you have Java installed. If you do not it can be accessed [here](https://www.oracle.com/java/technologies/downloads/).
2. Download JMeter [here](https://jmeter.apache.org/download_jmeter.cgi).
3. Run the jmeter.bat file (on Windows) or jmeter.sh file (on Linux/Mac) to start JMeter.
4. In JMeter, a test is organized into a "Test Plan." Right-click on the "Test Plan" in the left-hand panel. Choose "Add" -> "Threads (Users)" -> "Thread Group."
5. Right-click on the "Thread Group" you added and choose "Add" -> "Sampler" -> "HTTP Request". In the "HTTP Request" sampler, set the "Server Name or IP" to the DNS name or IP address of your ALB.
6. Right-click on the "Thread Group" again and choose "Add" -> "Listener" -> "View Results Tree". This listener allows you to view the results of your test.
7. Click the "Run" menu and choose "Start" or simply press the green "Play" button. JMeter will start sending requests to your web server based on the configured settings.

## Notes and Thank Yous
Note: This is a completely open and unsecured project. No SSL, no firewall, open ports everywhere. Proceed at your own peril.
Note: This is not a commercial or professional endevour. I am a networking student. Proceed at your own peril.
Note: I'm not kidding. I have no idea what I'm doing. Proceed at your own peril.
Thank you: Zak for encouraging me to undertake this project.
Thank you: AWS for having a bunch of easily accessible and free tech.
Thank you: Celine for listening to me complain.

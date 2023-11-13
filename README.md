# CNE430Final

Setting up autoscaling webserver using AWS

## Description

In an attempt to make AWS “do something” Matt Bennett and Co. (it’s just me - I’m the Matt Bennett and the Co) will harness their technological powers to animate into existence cloned EC2 webservers under the umbrella of a VPC (virtual private cloud) using an ALB (application load balancer) to coordinate their efforts into rallying to the aid of their distressed brethren. After fulfilling their duties, these valiant EC2 instances shall return to their slumber, awaiting the next fateful call. 
The actual deliverables are as follows: 
1. Create VPC. Set up necessary routing and security groups
2. Create an EC2 instance within the VPC. Set it up as a webserver. (Wordpress most likely)
3. Create an ASG (autoscaling group). Use AMI of instance created in deliverable #2. Define scaling policies.
4. Create an ALB associated with the VPC IP ranges.
5. Create Cloudwatch trigger for ASG. (This step may be unnecessary, update when reached)
6. Create way to stress the webserver sufficiently to trigger autoscaling.

## Walkthrough

### 1. Creating and setting up VPC:

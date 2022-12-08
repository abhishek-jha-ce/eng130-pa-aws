## Scalability
- It means that an applicatoin/sytem can handle greater loads by adapting.
- **Vertical Scalability** (Scale Up/Down) - Increase the size of the instance. There is a limit on how much we can scale (Hardware Limit).
  - Example: Upgrading from `t2.micro` to `t2.large`.
  - Use Case: Common for non distributed systems, such as database e.g. `RDS`, `ElastiCache`.
- **Horizontal Scalability** (Scale Out/In a.k.a `Elasticity`) - Increasing the number of instance/system for our application.
  - Achieved using `Auto Scaling Group` and `Load Balancer`.
  
## High Availability 
- We can run our application/system in at least 2 data centeres in different AZ's. The goal of high availability is to survive data center loss.
- The High Availability can both be passive (e.g. RDS multi AZ) or active (horizontal scaling in multiple AZ). It is achieved using:
  - Auto Scaling Group multi AZ.
  - Load Balancer multi AZ.

## Load Balancing
- Load Balances are servers that forward traffic to multiple servers (e.g. EC2 Instances) downstream. The more load we have, it will be evenly balanced between the EC2 Servers. 

<img align=right src="https://user-images.githubusercontent.com/110366380/206427484-41917628-b60c-46c4-ac2e-003f2ee4c7c1.png">

Advantages of Load Balancers
- Spread load across multiple downstream instances.
- Expose a single point of access (DNS) for our application.
- Handle failures of downstream instances.
- High availability across zones.
- Separate public traffic from private traffic.

# Elastic Load Balancing (ELB)

- It is a **managed load balancer** so AWS gurantees that it will be working and take care of upgrades, maintenance & high availability.
- It is integrated with many AWS services like `EC2`, `EC2 ASG`, `ECS`, `Route 53`, `CloudWatch`, `Global Accelerator` etc.

## Health Checks

Health Checks are crucial for Load Balancers. They enable the load balancer to know if instances it forwards traffic to are available.

<img align=right src="https://user-images.githubusercontent.com/110366380/206429612-ee1a96ff-1d43-4aba-9efa-52fa97278f00.png">

- The health check is done on a `port` and a `route`.
- If the response is not 200(OK), then the instance is unhealthy.

## Types of Load Balancer
### **Classic Load Balancer** (2009) 
- Supports `http`, `https`, `tcp`, `SSL`.
- **Deprecated** The Classic Load Balancer is deprecated and will soon not be available in the AWS Console.
  
### **Application Load Balancer (ALB)** (2016)

- **Layer 7** load balancer (HTTP).
- Load balancing to multiple HTTP applications across machines (target groups).
- Load balancing to multiple applications on the same machine (containers).
- Supports `http`, `https`, `WebSocket`.
- Supports redirects (e.g from HTTP to HTTPS).

<p align="center">
  <img src="https://user-images.githubusercontent.com/110366380/206438369-21ec66a1-e0f6-410c-9ad0-d87f1aa91776.png">
</p>

- We can have Routing tables to different target groups, for e.g.:
  - Route based on `path` in URL (abc.com/users or abc.com/posts) 
  - Route based on `hostname` in URL (one.abc.com or other.abc.com)
  - Route based on `Query String`, (abc.com/users?id=123&order=true)

<p align="center">
  <img src="https://user-images.githubusercontent.com/110366380/206437836-0bdb50d7-5e4f-4671-8b54-eea796a5e695.png">
</p>

- It is great fit for micro services & container-based application (example: Docker & Amazon ECS).
- It has a port mapping feature to redirect to a dynamic port in ECS.
  
### Network Load Balancer (2017) 
- Supports `tcp`, `tls`, `udp`.

### **Gateway Load Balancer** (2020)
- Operates at Layer 3 (Network Layer).

Load balancers can be setup as *internal*(private) or *external*(public). 

## Accessing Resources
To access the server (EC2 Instance) using load balancer:
- Allow traffic from anywhere to the `load balancer`.
- Allow traffic from **only** the `load balancer` to the `EC2 Instance`.
- This can be achieved by allowing `Port 80` access to the security group of the `load balancer` in the security group of the instance.

<p align="center">
  <img src="https://user-images.githubusercontent.com/110366380/206431846-efd8e185-7e4e-42a9-9807-df88317085f6.png">
</p>

## Sticky sessions

By default, an Application Load Balancer routes each request independently to a registered target based on the chosen load-balancing algorithm. However, we can use the sticky session feature (also known as session affinity) to enable the load balancer to bind a user's session to a specific target. 

<p align="center">
  <img src="https://user-images.githubusercontent.com/110366380/206467860-ae7467a1-d311-4bb1-9b5b-97742689b446.jpg">
</p>

- This ensures that all requests from the user during the session are sent to the same target.
- This feature is useful for servers that maintain state information in order to provide a continuous experience to clients.
- To use sticky sessions, the client must support cookies.
- The `cookie` used for stickiness has an expiration date, we can control that.
- **Note:** Enabling stickiness may bring imbalance to the load over the EC2 Instances.
- Sticky sessions are **not supported** if *cross-zone load balancing* is **disabled**. Attempting to enable sticky sessions while cross-zone load balancing is disabled will fail.

### Cookies

There are 2 types of Cookies:
#### Application-based Cookies
- Custom Cookie
  - The are generated by the target.
  - Can include any custom attributes required by the application.
  - Cookie name must be specified individually for each target group.
  - Cannot use reserved name like `AWSALB`, `AWSALBAPP` or `AWSALBTG`
- Application Cookie
  - Generated by load balancer.
  - Cookie name is `AWSALBAPP`.
#### Duration-based Cookies
- Generated by the load balancer.
- Cookie name is `AWSALB` for ALB OR `AWSELB` for CLB.

## Enabling Sticky Sessions

- Navigate to the `Target Groups`.
- Select the `Target Group`.
- Click on `Actions` and then `Edit Attributes`.
- Click on `Stickiness` at the bottom.
- We can see 2 types of `Stickiness`.

![image](https://user-images.githubusercontent.com/110366380/206473061-c8e8fc61-6905-4823-8961-06bb42915154.png)

For `Application-based cookie` we have to specify `App cookie name` as well.
- Save changes.

## Cross-Zone Load Balancing

The nodes of our load balancer distribute requests from clients to registered targets. When cross-zone load balancing is on, each load balancer node distributes traffic across the registered targets in all registered Availability Zones. When cross-zone load balancing is off, each load balancer node distributes traffic only across the registered targets in its Availability Zone. 

It is used when zonal failure domains are preferred over regional, ensuring that a healthy zone isn't impacted by an unhealthy zone, or for overall latency improvements.

- It is **enabled** by default for `Application Load Balancer`. **No charges** incurred for inter AZ data transfer.
![image](https://user-images.githubusercontent.com/110366380/206480092-3066d20d-5a03-4c6c-910f-1a6c582921c0.png)

We can **disable** it by clicking on `edit`.
![image](https://user-images.githubusercontent.com/110366380/206480490-b52962ea-0d81-401d-a22d-636dc7112a8d.png)

If it doesn't allow us to change, It is inherits the settings from load balancer. We have to go to the `target group` -> `attributes` -> `edit` and then turn off for that specific target group.
 ![image](https://user-images.githubusercontent.com/110366380/206482110-3208358c-1813-4514-859e-c769300fec41.png)

- It is **disabled** by default for `Network Load Balancer`. **Charges** will be incurred for inter AZ data transfer.
- It is **disabled** by default for `Gateway Load Balancer`. **Charges** will be incurred for inter AZ data transfer.
![image](https://user-images.githubusercontent.com/110366380/206479252-e6f7c982-75d9-4851-acaf-def0465b0fbd.png)

We can enable it by clicking on `edit` and enable it.
![image](https://user-images.githubusercontent.com/110366380/206479559-1e8f8afd-2456-48b4-961d-e17158c57d85.png)

- It is **disabled** by default for `Classic Load Balancer`. **No charges** incurred for inter AZ data transfer.

## SSL Certificates

- SSL refers to **Secure Sockets Layer**. It allows traffic between the user and load balancer to be encrypted in transit (aka in-flight encryption).
- It has a newer version called `TLS`, which stands for **Transport Layer Security**. It is mostly used nowadays.
- Public SSL certificates are issued by *Certificate Authority (CA)* like Comodo, Synamtec, GoDaddy, GlobalSign etc.
- SSL Certificates have an expiration date and must be renewed.

## Load Balancer - SSL Certificates

![image](https://user-images.githubusercontent.com/110366380/206484228-8f25f8f9-a5fd-4d6e-a7b8-df45ae9379ba.png)

- The load balancer uses an `X.509` certificate (SSL/TLS server certificate).
- We can manage certificates using ACM (AWS Certificate Manager).
- We can also create or upload our own certificates.

### HTTPS Listener:
A listener is a process that checks for connection requests, using the protocol and port that we configure. The rules that we define for a listener determine how the load balancer routes requests to its registered targets.
- We must specify a default certificate.
- We can alos add an optional list of certificates to support multiple domains.
- Users can use SNI (Server Name Indication) to specify the hostname they reach.
- We can also specify a security policy to support older versions of SSL/TLS for legacy clients.

### SNI - Server Name Indication
Server Name Indication (SNI) is an extension to the TLS protocol that is supported by browsers and clients released after 2010.

- It solves the problem of loading multiple SSL certificates in one web server, to server multiple websites.
- It requires the client to indicate the hostname of the target server in the initial SSL handshake.
- The server then find the correct certificate or return the default one.

![image](https://user-images.githubusercontent.com/110366380/206487121-043c6513-605e-4dfc-a410-aae7121ba8a5.png)

**Note**: It only works for `ALB` & `NLB` but not `CLB`. It also works with `CloudFront`.

### Enable SSL Certificates

To add/enable SSL Certificates:
- Go to `Load Balancers`.
- Click on `Add Listener`.

![image](https://user-images.githubusercontent.com/110366380/206500092-e297c868-7b3b-4dc9-af5d-2095d210726f.png)

- Select the `port`.
- Select the `target group`.
- Set the `security policy`.
- Choose the `ACM` or where the certificate is from. We can also choose to import the certificate.
- Click on `Add`

## Connection Draining

Connection draining is a process that ensures that existing, in-progress requests are given time to complete when a EC2 instance (VM) is removed. 

- It is called `Connection Draining` for **CLB**.
- It is called `Deregistration Delay` for **ALB** & **NLB**.
- It stops sending new requests to the EC2 instance which is de-registering.
- It gives some time for the active requests to be completed while the instance is de-registering or marked unhealthy.
- We can set the parameter for the de-registering between 1 to 3600 seconds (1 Hour), Default is 300 Seconds.
- We can disable it altogether, we can set the value to 0.
- It is a good idea to set a low value if the requests are short, else set it depending on the type of request.

![image](https://user-images.githubusercontent.com/110366380/206509384-263716c5-8efa-4fc5-a3dc-267bc5f7d6bf.png)

# Auto Scaling Group

An Auto Scaling group contains a collection of EC2 instances that are treated as a logical grouping for the purposes of automatic scaling and management. An Auto Scaling group also lets us use Amazon EC2 Auto Scaling features such as health check replacements and scaling policies.
<img align="right" src="https://user-images.githubusercontent.com/110366380/206510608-81ad9da1-9622-41d8-bbd7-211e93b517c8.png">
The main Goal of `Auto Scaling Group` is:
- Scale out to match the increased load.
- Scale in to match a decreased load.
- Ensure we have the minimum and a maximum number of EC2 instance.
- Automatically register new instances to a load balancer.
- Re-create an EC2 instance in case a previous one is terminated.
- ASG are free - We only pay for the underlying EC2 instance.

## Auto Scaling Group Attributes

To set up a Auto Scaling Group, We setup a `Launch template`. Note: `Launch Configurations` are deprecated now. 

The `Launch template` consists of (similar to creating an EC2 instance):
- AMI
- Instance Type
- SSH Key Pair
- Security Groups
- EBS Volumes
- EC2 User Data

Auto Scaling attributes:
- Network + Subnets Information

- Load Balancer Information, Health checks and more

- `Min Size`, `Max Size` and `Initial Capacity`.
- Scaling Policies.

- IAM Roles for EC2 Instances

- It is possible to scale an ASG based on CloudWatch alarms.
  - An alarm monitors a metric (Average CPU, or a custom metric).
  - Based on the alarm, we can create scale-in (decrease the no of instances) or scale-out (increase the number of instances) policies.

![image](https://user-images.githubusercontent.com/110366380/206514530-4241f6e4-abba-44c1-893c-6588f56315bc.png)

## Creating an Auto Scaling Group

- Click on `Auto Scaling Groups` then `New Auto Scaling Groups`.
- Give the Name for the ASG.
- Select the `Launch template`. If not already created one, click on `Create a launch template` and create one using the various attributes as metnioned above.
- Refresh and Select the `Launch template`.
- Click next for `instance launch options`. Select the VPC and Availability zones and subnets. 

![image](https://user-images.githubusercontent.com/110366380/206517463-f2c0262f-e3ac-4562-94ca-c8b04411b72a.png)

- Click next for advanced options like configuring `Load balancing` and `Health checks`
![image](https://user-images.githubusercontent.com/110366380/206519732-df2f948a-9998-40c7-8f9d-786e8652a043.png)
- Health check screenshot:
![image](https://user-images.githubusercontent.com/110366380/206518097-3eb5f6e4-acac-48fb-83b1-e395e17ded14.png)
- Click next to configure group size and scaling policies.

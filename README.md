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


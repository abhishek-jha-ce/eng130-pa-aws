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

## Elastic Load Balancing (ELB)

- It is a **managed load balancer** so AWS gurantees that it will be working and take care of upgrades, maintenance & high availability.
- It is integrated with many AWS services like `EC2`, `EC2 ASG`, `ECS`, `Route 53`, `CloudWatch`, `Global Accelerator` etc.

### Health Checks

Health Checks are crucial for Load Balancers. They enable the load balancer to know if instances it forwards traffic to are available.

<img align=right src="https://user-images.githubusercontent.com/110366380/206429612-ee1a96ff-1d43-4aba-9efa-52fa97278f00.png">

- The health check is done on a `port` and a `route`.
- If the response is not 200(OK), then the instance is unhealthy.

### Types of Load Balancer
#### **Classic Load Balancer** (2009) 
- Supports `http`, `https`, `tcp`, `SSL`.
- **Deprecated** The Classic Load Balancer is deprecated and will soon not be available in the AWS Console.
  
#### **Application Load Balancer (ALB)** (2016)

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
  
#### Network Load Balancer (2017) 
- Supports `tcp`, `tls`, `udp`.

#### **Gateway Load Balancer** (2020)
- Operates at Layer 3 (Network Layer).

Load balancers can be setup as *internal*(private) or *external*(public). 

### Accessing Resources
To access the server (EC2 Instance) using load balancer:
- Allow traffic from anywhere to the `load balancer`.
- Allow traffic from **only** the `load balancer` to the `EC2 Instance`.
- This can be achieved by allowing `Port 80` access to the security group of the `load balancer` in the security group of the instance.

<p align="center">
  <img src="https://user-images.githubusercontent.com/110366380/206431846-efd8e185-7e4e-42a9-9807-df88317085f6.png">
</p>


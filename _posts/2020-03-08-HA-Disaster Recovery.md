---
title: "HA"
excerpt: "AWS HA & Disaster Recovery"
toc: true
toc_sticky: true
categories:
  - AWS
modified_date: 2020-03-08 10:36:28 +0900
---
## HA Network Diagram
![ha-architecture](https://dasoldasol.github.io/assets/images/image/ha-1.png)

## Adding Resilience And Autoscaling 
Write Node & Read Node
![2nodes](https://dasoldasol.github.io/assets/images/image/ha-2.png)

## Scenario
- You are working for a University as their AWS Consultant. They want to have a disaster recovery strategy in AWS for mission-critical applications after suffering a disastrous outage wherein they lost student and employee records. They don't want this to happen again but at the same time want to minimize the monthly costs. You are instructed to **set up a minimal version** of the application that is **always available** in case of any outages. The **DR site** should **only run the most critical core elements** of your system in AWS **to save cost** which can be rapidly upgraded to a full-scale production environment in the event of system outages.    
Which of the following disaster recovery architectures is the most cost-effective type to use in this scenario?
  - **A) Pilot Light**
  - The term pilot light is often used to describe a DR scenario in which a minimal version of an environment is always running in the cloud.     
  For example, with AWS you can maintain a pilot light by configuring and running the most critical core elements of your system in AWS. When the time comes for recovery, you can rapidly provision a full-scale production environment around the critical core.
  - **Backup & Restore** : is incorrect because you are running mission-critical applications, and the speed of recovery from backup and restore solution might not meet your RTO and RPO.
  - **Warm Standby** : is incorrect. Warm standby is a method of redundancy in which the scaled-down secondary system runs in the background of the primary system. Doing so would not optimize your savings as much as running a pilot light recovery since some of your services are always running in the background.
  - **Multi Site** : is incorrect. it is expensive because of active/active configuration

- A start-up company has an EC2 instance that is hosting a web application. The volume of users is expected to grow in the coming months and hence, you need to add **more elasticity and scalability** in your AWS architecture to cope with the demand.     
Which of the following options can satisfy the above requirement for the given scenario? (Choose 2)
  - **A1) Set up two EC2 instances and then put them behind an Elastic Loadbalancer(ELB)**
  - **A2) Set up two EC2 instances and use Route53 to route traffic based on a Weighted Routing Policy**
  - Using an **Elastic Load Balancer** is an ideal solution for adding **elasticity** to your application. Alternatively, you can also create a policy in Route 53, such as a Weighted routing policy, to evenly distribute the traffic to 2 or more EC2 instances.
  - **AWS WAF** : is incorrect because AWS WAF is a web application firewall that helps protect your web applications from common **web exploits**. This service is more on providing **security** to your applications.

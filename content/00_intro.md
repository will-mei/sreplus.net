---
title: "00 intro"
date: 2021-05-31T02:51:07-04:00
draft: false
---

{{< columns >}}

https://www.redhat.com/en/topics/devops/what-is-sre

## What is SRE(site reliability engineering)?

Site reliability engineering is a software engineering approach to IT operations.

SRE teams use software as a tool to manage systems, solve problems, and automate operations tasks.

SRE takes the tasks that have historically been done by operations teams often manually, and instead gives them to engineers or ops teams
who use software and automation to solve problems and production systems.

SRE is a valuable practice when creating scalable and highly reliable software systems.
It helps you manange large systems through code, which is more scalable and sustainable for sysadmins managing thousands or hundreds of thousands of machines.

The concept of site reliability engineering comes from the Google engineering team and is credited to Ben Treynor Sloss. 
SRE helps teams find a balance between releasing new features and making sure that they are reliable for users.

Standardization and automation are 2 important components of the SRE model. Site reliability engineers should always be looking for ways to enhance and automate operations tasks.
In this way, SRE helps to improve the reliability of a system today, while also improving it as it grows over time. 
SRE supports teams who are moving from a traditional approach to IT operations to a cloud-native approach.

<--->

## what does a site reliability engineer do?

A site reliability engineer is a unique role that requires either a background as a software developer with additional operations experience, or as a sysadmin or in an IT operations role that also has software development skills. 

SRE teams are responsible for how code is deployed, configured, and monitored, as well as the availability, latency, change management, emergency response, and capacity management of services in production.

Site reliability engineering helps teams to determine what new features can be launched and when by using service-level agreements (SLAs) to define the required reliability of the system through service-level indicators (SLI) and service-level objectives (SLO). 

An SLI is a defined measure of specific aspects of provided service levels. Key SLIs include request latency, availability, error rate, and system throughput. An SLO is based on the target value or range for a specified service level based on the SLI.

An SLO for the required system reliability is then determined based on the downtime agreed upon as acceptable. This downtime level is referred to as an error budget, the maximum allowable threshold for errors and outages. 

{{< /columns >}}

With SRE, 100% reliability is not expected; failure is planned for and accepted. 

The development team is able to "spend" the error budget when releasing a new feature. Using the SLO and error budget, the development team can determine whether or not a product or service is able to launch based on the available error budget.
If a service is running within the error budget, then the development team can launch whenever they want, but if the system currently has too many errors or goes down for longer than the error budget allows then no new launches can take place until the errors are within budget.   

The development team conducts automated operations tests to demonstrate reliability. 

Site reliability engineers split their time between operations tasks and project work. According to SRE best practices from Google, a site reliability engineer can only spend a maximum of 50% of their time on operations, which should be monitored to ensure they don’t go over. 
The rest of the time should be spent on development tasks like creating new features, scaling the system, and implementing automation.
Excess operational work and poorly performing services can be redirected back to the dev team to run instead of the site reliability engineer spending too much time on the operations of an application or service. 
Automation is an important part of the site reliability engineer’s role. If they are dealing with a problem repeatedly then they will automate a solution. This also helps ensure that operations work remains at half of their workload. 

Maintaining the balance between operations and development work is a key component of SRE. 


Clud security refers to the tools, technologies, processes and controls you put in place to secure your cloud environment.

We cannot copy and paste the existing on prem security controls into the cloud (the previous environment), because it is a mistake and never works.

## The cloud security models
To properly implemented a proper strategy and roadmap need to be in place with the security architecture or framework that oversees the entire process.

A cloud security model contains all the processes, tools and skills that you need to maintain your cloud at a secure posture.

At an high level a Cloud Security Model comprises:
1. Cloud security principles 
2. Cloud security governance framework


# Cloud security principles
The key principles which define the cloud security are:
1. Shared responsibility
2. teh identity perimeter
3. zero trust architecture
4. security as code
5. automation in incident response

## Shared Responsibility
The shared responsibility model is a principle that is shared between all cloud providers and states that security is an obligation that is shared between the cloud provider and the customer.

The cloud provider will follow the best practices to secure your data.
You need to configure everything with the best practises.


For example in AWS contract:
- "The cloud provider will handle security of the cloud while the customer handles security in the cloud"

Shared Responsibility is one of the most fundamental principles of the cloud.

## Identity Perimeter
It is very important to allow the access to the cloud only to the authorized people.

The provider alllows the access to the cloud, but you need to ensure an access policy.

The common controls are:
- Multi-factor Authentication
- Context-based controls
- Risk scoring
- Single sign-on

## Zero trust architecture
Zero trust is a concept and not a product.

It means that you don't trust any service or user either within or outside the network and verify everything.
- Every request is inspected before authorize it.

In this context, also if the identity is into the private network it is not allowed to access the resources and services without requests inspection and validation.

An identity can be a user, application, a cloud service, etc.

In a zero trust world, the identity becomes the firewall that allows and disallows access.
- Who you are becomes your firewall. You're validated and authorized -> you pass. Otherwise -> you don't pass.

## Security as code
A lot of security best practices in the cloud revolve around coding.

### Infrastructure as code (IaC)
The infrastructure is defined into a code template.
- It is then processed by the provided and converted into the actual infrastructure in the cloud!

A few lines of code will let you spin up a complete server in the cloud.

We can use commercial or open source tools to scan these templates for weaknesses.

Knowing how to spin un a basic network or server in the cloud using Terraform is an essential skill!

## API calls
The cloud can be looked at as a series of API calls that are amnually or autmatically called in response to events.

Cloud security professionals must learn how to make cloud services integrate and talk to each other in an automated manner.

In a cloud environment, public APIs can be left insecure and open channels for attackers.


## Serverless
Servless is an execution model where there is a full abstraction of the environment and only code exists to run and secure.

The servers are present!

The idea is:
- **==you don't need to manage servers, you just need  to write the code.==**
You write the code of your function and the provider manages the server, OS, scalability, security and updates.

You pay only for the time your code is executed.

### AWS Serverless function (AWS lambda)
You write code in python, javascript (Node.js), Java etc and it is executed only when it's needed (event, click, an API call and so on).


## Automation in Incident Response
<mark style="background: #BBFABBA6;">Incidence response means to reply rapdly and in an organized way to a sexcurity incident.</mark>

We can use tools and software to manage these incidents, with alerts and so on.

## Threat Intelligence
Nowdays we can have acces to Threat Intelligence Data that provides us insights into potential attacks, indicators of compromise, alerts on possible attacks and so on.

threat intelligence enables faster decision making.
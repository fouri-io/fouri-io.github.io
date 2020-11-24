---
title: "Reducing Time to Market with AWS CDK"
date: 2020-11-21T09:22:30-06:00
comments: true
categories:
  - scale
tags:
  - aws
  - cdk
---
![Photo by Kevin Ku on Unsplash](/assets/images/kevin-ku-aiyBwbrWWlo-unsplash-fouri.jpg)

Times have certainly changed in software development over the past fifteen years. The days of filing IT tickets to provision a server or create a database table, followed by waiting, waiting some more, and (you guessed it) waiting again are largely gone. Cloud computing and [containerization](https://www.docker.com/resources/what-container) have resulted in a pardigmn shift, moving Operational IT chips to the development side of the table --aka DevOps. Developers really are the new [Kingmakers](https://www.activestate.com/blog/developers-new-kingmakers/) by shaping reality to fit their narrative.

With this great power does come great responsiblity and thankfully AWS is there to help. The AWS console will allow users to rapidly provision servers, databases, apis, and much more. But when building beyond a proof-of-concept the challenges with console administration quickly become apparent. Consider the following:
1. What happens if I need to replicate this configuration?
2. How do I build UAT or Dev environments?
3. How do I track infrastructure changes --i.e. version my infrastructure?
4. ... and the list goes on

### Along comes the Blue Pill - CloudFormation

Thankfully, AWS came to the rescue again with [CloudFormation](https://aws.amazon.com/cloudformation/). This Infrastructure-As-Code allows developers to declare infrastructure in JSON or YAML files and use your favorite version control (e.g. git, subversion) to manage changes. It essentially solves all of the problems listed above. "Great Colby, what about this CDK thing? You should rename the title." Ok, Ok, I'm getting there, show some patience. Although CloudFormation works, it is incredibly dense/verbose/structured. Every piece of infrastructure, accounts, security groups need to be hand-bombed in. When the architecture starts to complicate: private VPCs, Subnets, Internet Gateways, and NATS, this can become a daunting task quickly. A lot of hours/days/weeks have been spent tuning CloudFormation scripts. 

Just when you have tweaked your 8000 line CloudFormation masterpiece, AWS Cloud Development Kit (CDK) comes strolling along with its big, bad, self giving you a rakish wink as it passes on its way to brevity and the future. You may be asking yourself "I am good with CloudFormation, why should I switch, besides it works." My answer is simple: assembly language works, some people are good at it, but there are faster/easier ways to build software, and unlike you oh-contrarion-complainy-one, the world values time-to-market and maintainability. In fact, Cloudformation is an output of CDK, just like assemby is to Java or C. Now that I have convinced you, lets talk CDK.

### Take the Red Pill - CDK
A good overview can be taken from the [AWS docs](https://aws.amazon.com/cdk/): 
> The AWS Cloud Development Kit (AWS CDK) is an open source software development framework to define your cloud application resources using familiar programming languages.

> Provisioning cloud applications can be a challenging process that requires you to perform manual actions, write custom scripts, maintain templates, or learn domain-specific languages. AWS CDK uses the familiarity and expressive power of programming languages for modeling your applications. It provides you with high-level components called constructs that preconfigure cloud resources with proven defaults, so you can build cloud applications without needing to be an expert. AWS CDK provisions your resources in a safe, repeatable manner through AWS CloudFormation. It also enables you to compose and share your own custom constructs that incorporate your organization's requirements, helping you start new projects faster.

"Great, great, but what does all that really mean to me?" It is easiest to explain by outlining a real-world problem with CDK flexing its power to solve all your problems (well maybe not all  --you are pretty screwed up). Let me present the following common architecture pattern solving the following use-cases
1. Launch a micro-service hosted in a container
2. I would like it to be serverless -- i.e. I don't want to manage servers
3. I want to control traffic by isolating my service from the public internet -- i.e. Private VPN
4. TODO - Availability, Load Balancing, Security

TODO - Architecture Diagram here

TODO - Show the code that can accomplish

TODO - Explain constructs and patterns

"But I want to have something I can launch REALLY quickly, can't you just give it to me? Gimme, Gimme, Gimme"  Okay, okay, you paid your dues even though you act like a child, follow below to get this infrastructure launched with about fifteen minutes of work.


### Other Resources
- [AWS docs - Creating Fargate Service using CDK](https://docs.aws.amazon.com/cdk/latest/guide/ecs_example.html)
- [AWS docs - Getting started with CDK](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html)



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

//**************TODO**********//
- Fix diagram cropping
- Add explanation of contstructs
- Add instructions on how to do on own and links to github
- Add the next Blog post on ECS Service Extensions
- Cleanup Readme
- Fix Profile Pic
- Add LinkedIn to Profile

![Photo by Kevin Ku on Unsplash](/assets/images/kevin-ku-aiyBwbrWWlo-unsplash-fouri.jpg)

Times have certainly changed in software development over the past fifteen years. The days of filing IT tickets to provision a server, followed by waiting, waiting some more, and (you guessed it) waiting again are largely gone. Cloud computing and [containerization](https://www.docker.com/resources/what-container) have resulted in a pardigmn shift, sliding the Operational IT chips to the development side of the table --aka DevOps. Developers really are the new [Kingmakers](https://www.activestate.com/blog/developers-new-kingmakers/) by shaping reality to fit their narrative.

With this great power comes great responsiblity and thankfully AWS is there to help. The AWS console will allow users to rapidly provision servers, databases, apis, and much more. When moving beyond the proof-of-concept phase, challenges with console administration quickly emerge. Consider the following:
1. What happens if I need to replicate this configuration?
2. How do I build UAT (gamma) or Dev (beta) environments?
3. How do I track infrastructure changes --i.e. version my infrastructure?
4. How do I know if something changed, and how do I recover?
4. ... and the list goes on

### CloudFormation to the Rescue...kind of

Thankfully, AWS solved for this with [CloudFormation](https://aws.amazon.com/cloudformation/). This templating framework creates Infrastructure-As-Code, allowing developers to define resources in JSON or YAML files. Since they are just files, they can be versioned with any version control system. It essentially solves all of the problems listed above. 

> Sidebar: Are we at the stage where we can stop saying version control system? Git clearly won the war. Can we just say Git at this point or are there still enough Subversion folks out there that we have to keep saying version control? And if you are using Clearcase, then please remain quiet and let the grown-ups talk.

*"Colby get back to the point. what about this CDK thing? You should rename the title to CloudFormation or Colby's rambling"* 

Ok, Ok, I'm getting there, show some patience. Although CloudFormation works, it is incredibly dense/verbose/structured. Every piece of infrastructure: EC2 Instances, Containers, S3 buckets need to be coded along with all associated IAM roles and linkage. When the architecture starts to complicate: private VPCs, Subnets, Internet Gateways, and NATS, this can become a daunting task quickly. A lot of hours/days/weeks have been spent tuning CloudFormation scripts. 

Just when you have tweaked your 5000 line CloudFormation masterpiece, AWS Cloud Development Kit (CDK) comes strolling along with its Big-Bad-Self, giving you a rakish wink as it passes on its way to brevity and the future. You may be asking yourself 

*"I am good with CloudFormation, why should I switch? Besides it works."* 

My answer is simple: assembly language works, some people are good at it, but there are faster/easier ways to build software, and unlike you Oh-Contrarion-Complainy-One, the world values time-to-market and maintainability. In fact, Cloudformation is an output of CDK, just like assemby is to Java or C. Now that I have convinced you...Let's talk CDK.

### Take the Red Pill - CDK
A good overview can be taken from the [AWS docs](https://aws.amazon.com/cdk/): 
> The AWS Cloud Development Kit (AWS CDK) is an open source software development framework to define your cloud application resources using familiar programming languages.

> Provisioning cloud applications can be a challenging process that requires you to perform manual actions, write custom scripts, maintain templates, or learn domain-specific languages. AWS CDK uses the familiarity and expressive power of programming languages for modeling your applications. It provides you with high-level components called constructs that preconfigure cloud resources with proven defaults, so you can build cloud applications without needing to be an expert. AWS CDK provisions your resources in a safe, repeatable manner through AWS CloudFormation. It also enables you to compose and share your own custom constructs that incorporate your organization's requirements, helping you start new projects faster.

*"Great, great, but what does all that really mean to me?"* 

It is easiest to explain by outlining a real-world problem with CDK, flexing its power to solve all your problems (well maybe not all  --you are pretty screwed up). Let me present this common architecture pattern solving the following use-cases
1. Launch a micro-service hosted in a container
2. I would like it to be serverless -- i.e. I don't want to manage servers
3. I want to control traffic by isolating my service from the public internet -- i.e. Private VPN
4. I need my service to be available even if an outage happens in a Data Center -- i.e. Separate Availability Zones
5. I need all the goodness. What is this project management triangle you speak of? I want it perfect, I want it now, and I want it cheap --aka Manager Requirements -- *Disclosure: I am a manager so I can say that :-)*

Cool! We can do that, let me introduce you to CDK, the first dose is free, after that it will cost you -- tell your friends.

Curtain call *Skynet* a fully available, replicated, isolated, container service that has a DynamoDB serving content. It happens to be a killing machine intent on destroying the human race, but let's look beyond that minor pitfall and appreciate all the other cool stuff!

Humblebrag - I can provision this in X lines of code versus the Y lines it would take in CloudFormation. I can actually do it in even less with the very recent introduction of ECS Service Extenstions, but I will save that for another day/blog.

![Application Load Balanced Fargate Architecture](/assets/images/applicationloadbalancedfargate.png)

Now let's take a look at the code to accomplish this.
```java
// The code that defines your stack goes here
const vpc = new ec2.Vpc(this, "FouriVPC", {
  maxAzs: 3, // Default is all AZs in region
  cidr: "10.0.0.0/24"
});

const cluster = new ecs.Cluster(this, "SkynetCluster", {
  vpc: vpc,
  clusterName: "skynet-demo"
});

  // Create a role where permissions can be granted
const taskRole = new iam.Role(this, "SkynetTaskWorkerRole", {
  assumedBy: new iam.ServicePrincipal("ecs-tasks.amazonaws.com"),
});

// Create a load-balanced Fargate service and make it public
new ecs_patterns.ApplicationLoadBalancedFargateService(this, "SkynetFargateService", {
  cluster: cluster, // Required
  cpu: 512, // Default is 256
  desiredCount: 6, // Default is 1
  taskImageOptions: { 
    image: ecs.ContainerImage.fromRegistry("fouri/skynet"),
    containerName: "skynet-container",
    enableLogging: true,
    taskRole: taskRole,
    containerPort: 4000
  },
  memoryLimitMiB: 2048, // Default is 512
  publicLoadBalancer: true, // Default is false
  listenerPort: 4000
});

// Build DynamoDB Infrastructure
const tableName = 'skynet';

const table =  new dynamodb.Table(this, "SkynetTable", {
    tableName,
    partitionKey: { name: 'pk', type: dynamodb.AttributeType.STRING },
    sortKey: { name: 'sk', type: dynamodb.AttributeType.STRING },
    removalPolicy: cdk.RemovalPolicy.DESTROY
});

table.grantReadWriteData(taskRole);
```


TODO - Explain constructs and patterns

"But I want to have something I can launch REALLY quickly, can't you just give it to me? Gimme, Gimme, Gimme"  Okay, okay, you paid your dues even though you act like a child, follow below to get this infrastructure launched with about fifteen minutes of work.


### Other Resources
- [AWS docs - Creating Fargate Service using CDK](https://docs.aws.amazon.com/cdk/latest/guide/ecs_example.html)
- [AWS docs - Getting started with CDK](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html)



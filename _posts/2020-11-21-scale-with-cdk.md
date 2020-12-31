---
title: "Reducing Time to Market with AWS CDK"
date: 2020-12-30T09:22:30-06:00
comments: true
categories:
  - development
tags:
  - aws
  - cdk
---

**Audience**: This article is intended for intermediate/advanced Cloud/DevOps practitioners. If you are beginner, you could still get some good info, but may be a tough read. I suggest checking out 'Other Resources' at the bottom.

![Photo by Kevin Ku on Unsplash](/assets/images/kevin-ku-aiyBwbrWWlo-unsplash-fouri.jpg)


Times have certainly changed in software development over the past fifteen years. The days of filing IT tickets to provision a server, followed by waiting, waiting some more, and (you guessed it) waiting again are largely gone. Cloud computing and [containerization](https://www.docker.com/resources/what-container) have resulted in a paradigm shift, sliding the Operational IT chips to the development side of the table --aka DevOps. Developers really are the new [Kingmakers](https://www.activestate.com/blog/developers-new-kingmakers/) by shaping reality to fit their narrative.

With this great power comes great responsibility and thankfully AWS is there to help. The AWS console will allow users to rapidly provision servers, databases, apis, and much more. When moving beyond the proof-of-concept phase, challenges with console administration quickly emerge. Consider the following:
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

My answer is simple: assembly language works, some people are good at it, but there are faster/easier ways to build software, and unlike you Oh-Contrarion-Complainy-One, the world values time-to-market and maintainability. In fact, CloudFormation is an output of CDK, just like assembly is to Java or C. Now that I have convinced you...Let's talk CDK.

### Take the Red Pill - CDK
A good overview can be taken from the [AWS docs](https://aws.amazon.com/cdk/): 
> The AWS Cloud Development Kit (AWS CDK) is an open source software development framework to define your cloud application resources using familiar programming languages. Provisioning cloud applications can be a challenging process that requires you to perform manual actions, write custom scripts, maintain templates, or learn domain-specific languages. AWS CDK uses the familiarity and expressive power of programming languages for modeling your applications. It provides you with high-level components called constructs that pre-configure cloud resources with proven defaults, so you can build cloud applications without needing to be an expert. AWS CDK provisions your resources in a safe, repeatable manner through AWS CloudFormation. It also enables you to compose and share your own custom constructs that incorporate your organization's requirements, helping you start new projects faster.

*"Great, great, but what does all that really mean to me?"* 
Think of it this way, it will do all the CloudFormation work using an opinionated best practices DevOps approach with the ability to override.

It is easiest to explain by outlining a real-world problem with CDK, flexing its power to solve all your problems (well maybe not all  --you are pretty screwed up). Let me present this common architecture pattern solving the following use-cases
1. Launch a micro-service hosted in a container
2. I would like it to be serverless -- i.e. I don't want to manage servers
3. I want to control traffic by isolating my service from the public internet -- i.e. Private VPN
4. I need my service to be available even if an outage happens in a Data Center -- i.e. Separate Availability Zones
5. I need all the goodness. What is this project management triangle you speak of? I want it perfect, I want it now, and I want it cheap --aka Manager Requirements -- *Disclosure: I am a manager so I can say that :-)*

Cool! We can do that, let me introduce you to CDK, the first dose is free, after that it will cost you -- tell your friends.

Curtain call *Skynet* a fully available, replicated, isolated, container service that has a DynamoDB serving content. It happens to be a killing machine intent on destroying the human race, but let's look beyond that minor pitfall and appreciate all the other cool stuff!

Humblebrag - I can provision this in ~60 lines of code versus ~1100 lines it would take in CloudFormation. I can actually do it in even less with the very recent introduction of ECS Service Extensions, but I will save that for another day/blog.

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

"But I want to have something I can launch REALLY quickly, can't you just give it to me? Gimme, Gimme, Gimme"  Okay, okay, you paid your dues, follow below to get this infrastructure launched with about five minutes of work.

### Try it on your own
You need to have some prerequisites ready : AWS Account with infrastructure building permissions, AWS CLI, CDK, git (Check 'Other Resources' below for help getting these working)

We are going to clone the demo project, do a build with npm, and a cdk deploy
```
git clone https://github.com/fouri-io/skynet-cdk-demo.git
cd skynet-cdk-demo
npm install     // installs the needed node packages
npm run build   // builds the project and makes corresponding js files from javascript
cdk synth       // Synthesizes CDK artifacts -- Cloudformation
cdk deploy      // Will run for a few mins and deploy all of your infrastructure
```
After all the installing is complete, you should see something like the following:
SkynetCdkDemoStack: deploying...
SkynetCdkDemoStack: creating CloudFormation changeset...
[██████████████████████████████████████████████████████████] (41/41)

You will also have some outputs giving you an http endpoint to hit:
Go ahead and try out your new service:
```
http://<your output address here>:4000/ping         // You should receive 'pong' back
http://<your output address here>:4000/initialize
http://<your output address here>:4000/missions
http://<your output address here>:4000/seedTargets  // Load targets into Dynamo
http://<your output address here>:4000/targets      // Retrieve targets from Dynamo
```

WOW! Let's reflect on what is in place: VPC, subnets, Internet Gateways, NATS, security groups, a Fargate hosted Docker Container Node API with a Dynamo Table backing it, and all of it is plumbed together using best practices.

### Further Down the Rabbit Hole
"That is a pretty cool demo, but would I want to host my own stuff."
Since this is simply hosting a docker container, you can change a single line in CDK to change out the Skynet docker container for one of your own.  Simply change the line below pointing to your DockerHub hosted container and re-launch. You keep all the benefits listed above hosting your own containers. That is powerful!

`image: ecs.ContainerImage.fromRegistry("fouri/skynet")`

### Remember to Cleanup your Room
Remember, AWS will charge you for resources, so when you are done playing with this, make sure to destroy Skynet before it becomes self-aware
`cdk destroy`

I hope you have gained some knowledge after all this. If not, well you get what you pay for :-) Drop a comment below if you have any suggestions for additional context or insight or if you just want to say Hi.

>"That’s how it is with people. Nobody cares how it works as long as it works."
--The Matrix



### Other Resources
- [AWS docs - Creating Fargate Service using CDK](https://docs.aws.amazon.com/cdk/latest/guide/ecs_example.html)
- [AWS docs - Getting started with CDK](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html)
- [Installing AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-mac.html#cliv2-mac-install-cmd)
- [Installing AWS CDK](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html#getting_started_install)
- [Skynet API Code](https://github.com/fouri-io/skynet)




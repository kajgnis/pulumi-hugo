---
title_tag: "More AWS Examples"
title: More...
h1: More AWS examples
meta_desc: Pulumi Crosswalk for AWS provides simple, out of the box VPC functionality that follows widely accepted best practices.
meta_image: /images/docs/meta-images/docs-clouds-aws-meta-image.png
menu:
  clouds:
    parent: aws-guides
    identifier: aws-guides-more
    weight: 11
aliases:
  - /docs/aws/athena/
  - /docs/aws/cloudwatch/
  - /docs/aws/dynamodb/
  - /docs/aws/ec2/
  - /docs/aws/ec2/
  - /docs/aws/ecr/
  - /docs/aws/iam/
  - /docs/aws/kinesis/
  - /docs/aws/s3/
  - /docs/aws/sns/
  - /docs/aws/sqs/
  - /docs/clouds/aws/aws-guides/athena/
  - /docs/clouds/aws/aws-guides/cloudwatch/
  - /docs/clouds/aws/aws-guides/dynamodb/
  - /docs/clouds/aws/aws-guides/ec2/
  - /docs/clouds/aws/aws-guides/ec2/
  - /docs/clouds/aws/aws-guides/ecr/
  - /docs/clouds/aws/aws-guides/iam/
  - /docs/clouds/aws/aws-guides/kinesis/
  - /docs/clouds/aws/aws-guides/s3/
  - /docs/clouds/aws/aws-guides/sns/
  - /docs/clouds/aws/aws-guides/sqs/
---

The `@pulumi/aws` library enables fine-grained control over the AWS SQS resource meaning it can be coded, deployed, and managed entirely in code. Below are a bunch more awesome things you can do with it. (TODO)

## Create an Athena Database and NamedQuery

The `@pulumi/aws` library enables fine-grained control over the AWS Athena resource meaning it can be coded, deployed, and managed entirely in code.

```javascript
const aws = require("@pulumi/aws");

const databaseBucket = new aws.s3.Bucket("mydatabasebucket");

const database = new aws.athena.Database("mydatabase", {
    name: "mydatabase",
    bucket: databaseBucket.bucket
});

const namedQuery = new aws.athena.NamedQuery("mynamedquery", {
    database: database.id,
    query: database.id.apply(n => `SELECT * FROM ${n} limit 10;`)
});
```

## Create a CloudWatch dashboard

The `@pulumi/aws` library enables fine-grained control over the AWS CloudWatch resource meaning it can be coded, deployed, and managed entirely in code.

```javascript
const aws = require("@pulumi/aws");

const dashboard = new aws.cloudwatch.Dashboard("mydashboard", {
    dashboardName: "my-dashboard",
    dashboardBody: JSON.stringify({
        widgets: [
            {
                type: "metric",
                x: 0,
                y: 0,
                width: 12,
                height: 6,
                properties: {
                    metrics: [
                        [
                            "AWS/EC2",
                            "CPUUtilization",
                            "InstanceId",
                            "i-012345"
                        ]
                    ],
                    period: 300,
                    stat: "Average",
                    region: "us-east-1",
                    title: "EC2 Instance CPU"
                }
            },
            {
                type: "text",
                x: 0,
                y: 7,
                width: 3,
                height: 3,
                properties: {
                    markdown: "Hello world"
                }
            }
        ]
    })
});

const loginsTopic = new aws.sns.Topic("myloginstopic");

const eventRule = new aws.cloudwatch.EventRule("myeventrule", {
    eventPattern: JSON.stringify({
        "detail-type": [
            "AWS Console Sign In via CloudTrail"
        ]
    })
});

const eventTarget = new aws.cloudwatch.EventTarget("myeventtarget", {
    rule: eventRule.name,
    targetId: "SendToSNS",
    arn: loginsTopic.arn
})

const logGroup = new aws.cloudwatch.LogGroup("myloggroup");

const logMetricFilter = new aws.cloudwatch.LogMetricFilter("mylogmetricfilter", {
    pattern: "",
    logGroupName: logGroup.name,
    metricTransformation: {
        name: "EventCount",
        namespace: "YourNamespace",
        value: "1"
    }
});

const logStream = new aws.cloudwatch.LogStream("mylogstream", {
    logGroupName: logGroup.name
});

const metricAlarm = new aws.cloudwatch.MetricAlarm("mymetricalarm", {
    comparisonOperator: "GreaterThanOrEqualToThreshold",
    evaluationPeriods: 2,
    metricName: "CPUUtilization",
    namespace: "AWS/EC2",
    period: 120,
    statistic: "Average",
    threshold: 80,
    alarmDescription: "This metric monitors ec2 cpu utilization"
});
```

## Create an DynamoDB table

The `@pulumi/aws` library enables fine-grained control over the AWS DynamoDB resource meaning it can be coded, deployed, and managed entirely in code.

```javascript
const aws = require("@pulumi/aws");

const db = new aws.dynamodb.Table("mytable", {
    attributes: [
        { name: "Id", type: "S" },
    ],
    hashKey: "Id",
    readCapacity: 1,
    writeCapacity: 1,
});
```

## Create an EC2 instance

The `@pulumi/aws` library enables fine-grained control over the AWS EC2 resource meaning it can be coded, deployed, and managed entirely in code.

```javascript
const pulumi = require("@pulumi/pulumi");
const aws = require("@pulumi/aws");

// Create a VPC.
const vpc = new aws.ec2.Vpc("vpc", {
    cidrBlock: "10.0.0.0/16",
});

// Create an an internet gateway.
const gateway = new aws.ec2.InternetGateway("gateway", {
    vpcId: vpc.id,
});

// Create a subnet that automatically assigns new instances a public IP address.
const subnet = new aws.ec2.Subnet("subnet", {
    vpcId: vpc.id,
    cidrBlock: "10.0.1.0/24",
    mapPublicIpOnLaunch: true,
});

// Create a route table.
const routes = new aws.ec2.RouteTable("routes", {
    vpcId: vpc.id,
    routes: [
        {
            cidrBlock: "0.0.0.0/0",
            gatewayId: gateway.id,
        },
    ],
});

// Associate the route table with the public subnet.
const routeTableAssociation = new aws.ec2.RouteTableAssociation("route-table-association", {
    subnetId: subnet.id,
    routeTableId: routes.id,
});

// Create a security group allowing inbound access over port 80 and outbound
// access to anywhere.
const securityGroup = new aws.ec2.SecurityGroup("security-group", {
    vpcId: vpc.id,
    ingress: [
        {
            cidrBlocks: [ "0.0.0.0/0" ],
            protocol: "tcp",
            fromPort: 80,
            toPort: 80,
        },
    ],
    egress: [
        {
            cidrBlocks: [ "0.0.0.0/0" ],
            fromPort: 0,
            toPort: 0,
            protocol: "-1",
        },
    ],
});

// Find the latest Amazon Linux 2 AMI.
const ami = pulumi.output(aws.ec2.getAmi({
    owners: [ "amazon" ],
    mostRecent: true,
    filters: [
        { name: "description", values: [ "Amazon Linux 2 *" ] },
    ],
}));

// Create and launch an Amazon Linux EC2 instance into the public subnet.
const instance = new aws.ec2.Instance("instance", {
    ami: ami.id,
    instanceType: "t3.nano",
    subnetId: subnet.id,
    vpcSecurityGroupIds: [
        securityGroup.id,
    ],
    userData: `
        #!/bin/bash
        amazon-linux-extras install nginx1
        amazon-linux-extras enable nginx
        systemctl enable nginx
        systemctl start nginx
    `,
});

// Export the instance's publicly accessible URL.
module.exports = {
    instanceURL: pulumi.interpolate `http://${instance.publicIp}`,
};
```

## Create an ECR repository

The `@pulumi/aws` library enables fine-grained control over the AWS ECR resource meaning it can be coded, deployed, and managed entirely in code.

```javascript
const aws = require("@pulumi/aws");

const repository = new aws.ecr.Repository("myrepository");

const repositoryPolicy = new aws.ecr.RepositoryPolicy("myrepositorypolicy", {
    repository: repository.id,
    policy: JSON.stringify({
        Version: "2012-10-17",
        Statement: [{
            Sid: "new policy",
            Effect: "Allow",
            Principal: "*",
            Action: [
                "ecr:GetDownloadUrlForLayer",
                "ecr:BatchGetImage",
                "ecr:BatchCheckLayerAvailability",
                "ecr:PutImage",
                "ecr:InitiateLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:CompleteLayerUpload",
                "ecr:DescribeRepositories",
                "ecr:GetRepositoryPolicy",
                "ecr:ListImages",
                "ecr:DeleteRepository",
                "ecr:BatchDeleteImage",
                "ecr:SetRepositoryPolicy",
                "ecr:DeleteRepositoryPolicy"
            ]
        }]
    })
});

const lifecyclePolicy = new aws.ecr.LifecyclePolicy("mylifecyclepolicy", {
    repository: repository.id,
    policy: JSON.stringify({
        rules: [{
            rulePriority: 1,
            description: "Expire images older than 14 days",
            selection: {
                tagStatus: "untagged",
                countType: "sinceImagePushed",
                countUnit: "days",
                countNumber: 14
            },
            action: {
                type: "expire"
            }
        }]
    })
});
```

## Create an IAM role, policy, and attachment

The `@pulumi/aws` library enables fine-grained control over the AWS IAM resource meaning it can be coded, deployed, and managed entirely in code.

```javascript
const aws = require("@pulumi/aws");

const role = new aws.iam.Role("myrole", {
    assumeRolePolicy: JSON.stringify({
        Version: "2012-10-17",
        Statement: [{
            Action: "sts:AssumeRole",
            Principal: {
                Service: "ec2.amazonaws.com"
            },
            Effect: "Allow",
            Sid: ""
        }]
    })
});

const rolePolicy = new aws.iam.RolePolicy("myrolepolicy", {
    role: role,
    policy: JSON.stringify({
        Version: "2012-10-17",
        Statement: [{
            Action: [ "ec2:Describe*" ],
            Effect: "Allow",
            Resource: "*"
        }]
    })
});

const policy = new aws.iam.Policy("mypolicy", {
    policy: JSON.stringify({
        Version: "2012-10-17",
        Statement: [{
            Action: [
              "ec2:Describe*"
            ],
            Effect: "Allow",
            Resource: "*"
        }]
    })
});

const rolePolicyAttachment = new aws.iam.RolePolicyAttachment("myrolepolicyattachment", {
    role: role,
    policyArn: policy.arn
});

const user = new aws.iam.User("myuser");

const group = new aws.iam.Group("mygroup");

const policyAttachment = new aws.iam.PolicyAttachment("mypolicyattachment", {
    users: [user],
    groups: [group],
    roles: [role],
    policyArn: policy.arn
});
```

## Create a Kinesis stream

The `@pulumi/aws` library enables fine-grained control over the AWS Kinesis resource meaning it can be coded, deployed, and managed entirely in code.

```javascript
const aws = require("@pulumi/aws");

const stream = new aws.kinesis.Stream("mystream", {
    shardCount: 1
});
```

## Create S3 bucket resources

The `@pulumi/aws` library enables fine-grained control over the AWS S3 resource meaning it can be coded, deployed, and managed entirely in code.

```javascript
const aws = require("@pulumi/aws");

const bucket = new aws.s3.Bucket("my-bucket");

const bucketMetric = new aws.s3.BucketMetric("my-bucket-metric", {
    bucket: bucket.bucket
});

const bucketNotification = new aws.s3.BucketNotification("my-bucket-notification", {
    bucket: bucket.bucket
});

const bucketObject = new aws.s3.BucketObject("my-bucket-object", {
    bucket: bucket.bucket,
    content: "hello world"
});

const bucketPolicy = new aws.s3.BucketPolicy("my-bucket-policy", {
    bucket: bucket.bucket,
    policy: bucket.bucket.apply(publicReadPolicyForBucket)
})

function publicReadPolicyForBucket(bucketName: string) {
    return JSON.stringify({
        Version: "2012-10-17",
        Statement: [{
            Effect: "Allow",
            Principal: "*",
            Action: [
                "s3:GetObject"
            ],
            Resource: [
                `arn:aws:s3:::${bucketName}/*` // policy refers to bucket name explicitly
            ]
        }]
    });
}
```

## Create an SNS topic and subscription

The `@pulumi/aws` library enables fine-grained control over the AWS SNS resource meaning it can be coded, deployed, and managed entirely in code.

```javascript
const aws = require("@pulumi/aws");

const topic = new aws.sns.Topic("mytopic");

const topicSubscription = new aws.sns.TopicSubscription("mytopicsubscription", {
    topic: topic,
    protocol: "sqs",
    endpoint: queue.arn
});
```

## Create an SQS queue

```javascript
const aws = require("@pulumi/aws");

const queue = new aws.sqs.Queue("myqueue");
```

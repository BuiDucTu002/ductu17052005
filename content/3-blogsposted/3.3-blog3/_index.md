---
title: "3.3. Blog 3 - Infrastructure as Code with AWS CDK"
date: "`r Sys.Date()`"
weight: 3
chapter: false
---

# INFRASTRUCTURE AS CODE WITH AWS CDK: AUTOMATING CLOUD DEPLOYMENTS

This blog explores how to use AWS CDK (Cloud Development Kit) to automate AWS infrastructure deployment, minimizing manual effort, increasing code reuse, and improving development velocity.

---

## 1. Introduction to AWS CDK

AWS CDK is a framework that allows you to define AWS cloud infrastructure using programming languages (TypeScript, Python, Java, C#) instead of writing CloudFormation templates directly.

### AWS CDK Advantages

- **Type-safe**: Errors detected at compile-time
- **Reusable constructs**: Components can be reused
- **Familiar programming**: Use familiar languages
- **Easier maintenance**: Code is more readable and maintainable than YAML
- **Powerful abstractions**: Higher-level constructs

---

## 2. AWS CDK vs Terraform vs CloudFormation

| Criteria | CloudFormation | Terraform | AWS CDK |
|----------|--------|-----------|---------|
| Format | YAML/JSON | HCL | TypeScript/Python/Java |
| Learning Curve | High | Medium | Low (for developers) |
| AWS Services Support | 100% | ~70% | 100% (AWS services) |
| State Management | Native | External | CloudFormation-based |
| Multi-cloud | No | Yes | No (AWS only) |
| Community | Large | Very Large | Growing |
| Debugging | Difficult | Easy | Easy |

**Recommendation**: Use CDK if:
- ✅ Only using AWS
- ✅ Team familiar with JavaScript/Python
- ✅ Need code reuse
- ✅ Want type safety

---

## 3. AWS CDK Installation

### Requirements
```bash
# Node.js (v14.0.0+)
node --version

# NPM
npm --version
```

### Install CDK CLI
```bash
npm install -g aws-cdk

# Verify
cdk --version
```

### Create CDK Project
```bash
# Create directory
mkdir enggo-infrastructure
cd enggo-infrastructure

# Initialize CDK project (TypeScript)
cdk init app --language typescript

# Install dependencies
npm install
```

---

## 4. Example: Deploy Enggo-Backend Infrastructure

### 4.1 VPC Stack

```typescript
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';

export class VpcStack extends cdk.Stack {
  public readonly vpc: ec2.Vpc;

  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create VPC
    this.vpc = new ec2.Vpc(this, 'EnggoVpc', {
      cidr: '10.0.0.0/16',
      maxAzs: 2,
      natGateways: 1,
      subnetConfiguration: [
        {
          cidrMask: 24,
          name: 'Public',
          subnetType: ec2.SubnetType.PUBLIC,
        },
        {
          cidrMask: 24,
          name: 'Private',
          subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS,
        },
      ],
    });

    // Output VPC ID
    new cdk.CfnOutput(this, 'VpcId', {
      value: this.vpc.vpcId,
    });
  }
}
```

### 4.2 RDS Stack

```typescript
import * as cdk from 'aws-cdk-lib';
import * as rds from 'aws-cdk-lib/aws-rds';
import * as ec2 from 'aws-cdk-lib/aws-ec2';

export class DatabaseStack extends cdk.Stack {
  public readonly database: rds.DatabaseInstance;

  constructor(
    scope: cdk.App,
    id: string,
    vpc: ec2.Vpc,
    props?: cdk.StackProps
  ) {
    super(scope, id, props);

    // Security Group for RDS
    const dbSecurityGroup = new ec2.SecurityGroup(this, 'DbSecurityGroup', {
      vpc: vpc,
      description: 'Security group for Enggo RDS',
      allowAllOutbound: true,
    });

    // RDS Instance
    this.database = new rds.DatabaseInstance(this, 'EnggoDb', {
      engine: rds.DatabaseEngine.mysql({
        version: rds.MysqlEngineVersion.VER_8_0_35,
      }),
      instanceType: ec2.InstanceType.of(
        ec2.InstanceClass.T3,
        ec2.InstanceSize.SMALL
      ),
      vpc: vpc,
      vpcSubnets: {
        subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS,
      },
      securityGroups: [dbSecurityGroup],
      multiAz: true,
      storageEncrypted: true,
      deletionProtection: false,
      removalPolicy: cdk.RemovalPolicy.SNAPSHOT,
    });

    // Output Database Endpoint
    new cdk.CfnOutput(this, 'DbEndpoint', {
      value: this.database.dbInstanceEndpointAddress,
    });
  }
}
```

### 4.3 ECS Fargate Stack

```typescript
import * as cdk from 'aws-cdk-lib';
import * as ecs from 'aws-cdk-lib/aws-ecs';
import * as ecs_patterns from 'aws-cdk-lib/aws-ecs-patterns';
import * as ec2 from 'aws-cdk-lib/aws-ec2';

export class EcsStack extends cdk.Stack {
  constructor(
    scope: cdk.App,
    id: string,
    vpc: ec2.Vpc,
    dbEndpoint: string,
    props?: cdk.StackProps
  ) {
    super(scope, id, props);

    // ECS Cluster
    const cluster = new ecs.Cluster(this, 'EnggoCluster', {
      vpc: vpc,
      clusterName: 'enggo-cluster',
    });

    // Fargate Service with Load Balancer
    const fargateService =
      new ecs_patterns.ApplicationLoadBalancedFargateService(
        this,
        'EnggoService',
        {
          cluster: cluster,
          cpu: 256,
          desiredCount: 2,
          taskImageOptions: {
            image: ecs.ContainerImage.fromRegistry(
              'your-account-id.dkr.ecr.region.amazonaws.com/enggo-backend:latest'
            ),
            containerPort: 8080,
            environment: {
              DB_HOST: dbEndpoint,
              DB_NAME: 'enggo_db',
            },
          },
          publicLoadBalancer: true,
          protocol: ecs_patterns.ApplicationProtocol.HTTP,
        }
      );

    // Auto Scaling
    fargateService.service.autoScaleTaskCount({
      minCapacity: 2,
      maxCapacity: 4,
    });

    fargateService.service
      .autoScaleTaskCount()
      .scaleOnCpuUtilization('CpuScaling', {
        targetUtilizationPercent: 70,
      });

    fargateService.service
      .autoScaleTaskCount()
      .scaleOnMemoryUtilization('MemoryScaling', {
        targetUtilizationPercent: 80,
      });

    // Output Load Balancer URL
    new cdk.CfnOutput(this, 'LoadBalancerDNS', {
      value: fargateService.loadBalancer.loadBalancerDnsName,
    });
  }
}
```

### 4.4 Main App

```typescript
import * as cdk from 'aws-cdk-lib';
import { VpcStack } from './vpc-stack';
import { DatabaseStack } from './database-stack';
import { EcsStack } from './ecs-stack';

const app = new cdk.App();

// VPC Stack
const vpcStack = new VpcStack(app, 'EnggoVpcStack', {
  env: {
    account: process.env.CDK_DEFAULT_ACCOUNT,
    region: process.env.CDK_DEFAULT_REGION,
  },
});

// Database Stack
const dbStack = new DatabaseStack(
  app,
  'EnggoDbStack',
  vpcStack.vpc,
  {
    env: {
      account: process.env.CDK_DEFAULT_ACCOUNT,
      region: process.env.CDK_DEFAULT_REGION,
    },
  }
);

// ECS Stack
const ecsStack = new EcsStack(
  app,
  'EnggoEcsStack',
  vpcStack.vpc,
  dbStack.database.dbInstanceEndpointAddress,
  {
    env: {
      account: process.env.CDK_DEFAULT_ACCOUNT,
      region: process.env.CDK_DEFAULT_REGION,
    },
  }
);
```

---

## 5. CDK CLI Commands

```bash
# View CloudFormation template before deployment
cdk synth

# Show changes
cdk diff

# Deploy stack
cdk deploy

# Deploy multiple stacks
cdk deploy EnggoVpcStack EnggoDbStack EnggoEcsStack

# Delete resources
cdk destroy
```

---

## 6. Best Practices

### 6.1 Code Organization

```
lib/
├── stacks/
│   ├── vpc-stack.ts
│   ├── database-stack.ts
│   ├── ecs-stack.ts
│   └── index.ts
├── constructs/
│   ├── ecs-service-construct.ts
│   └── database-construct.ts
└── main.ts
```

### 6.2 Reuse Constructs

```typescript
// Create custom construct
export interface EnggoBackendProps extends cdk.StackProps {
  vpc: ec2.Vpc;
  instanceType: ec2.InstanceType;
}

export class EnggoBackendStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props: EnggoBackendProps) {
    super(scope, id, props);
    // Implementation
  }
}
```

### 6.3 Use Context Values

```typescript
// cdk.json
{
  "context": {
    "environment": "production",
    "instanceCount": 3
  }
}

// TypeScript code
const environment = this.node.tryGetContext('environment') || 'development';
const instanceCount = this.node.tryGetContext('instanceCount') || 1;
```

---

## 7. CI/CD Integration

### GitHub Actions

```yaml
name: CDK Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - run: npm install
      - run: npm run build
      - run: npx cdk deploy --all --require-approval never
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## 8. Results

### Technical Benefits
- ✅ Complete infrastructure as code
- ✅ Type-safe configuration
- ✅ Reusable components
- ✅ Version control for infrastructure
- ✅ Automated deployments

### Business Benefits
- ✅ Reduced deployment time
- ✅ Fewer configuration errors
- ✅ Increased team productivity
- ✅ Easier disaster recovery
- ✅ Consistent infrastructure

---

## 9. Additional References

- [AWS CDK Documentation](https://docs.aws.amazon.com/cdk/v2/guide/)
- [CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/)
- [CDK Patterns](https://cdkpatterns.com/)
- [AWS Best Practices for CDK](https://aws.amazon.com/blogs/developer/)

---

**Published:** April 2026  
**Last Updated:** May 2026

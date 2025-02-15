+++
title = "Create Internal Construct Hub"
weight = 200
+++

## Create the Internal Construct Hub Infrastructure

{{% notice warning %}} Please note that the Construct Hub web interface is delivered through a publicly accessible CloudFront distribution. Restricting access to specific users and groups is beyond the scope of this workshop. However, you may consider the following best practices to implement it:

- Use signed URLs or signed cookies to grant temporary access to the content in your CloudFront distribution.
- Use AWS Web Application Firewall (WAF) to protect your CloudFront distribution from common web attacks and unwanted traffic. You can create custom rules to block or allow access based on specific criteria, such as IP addresses, user agents, or geographic locations.
- Use Lambda@Edge to add custom logic to your CloudFront distribution. You can use Lambda@Edge to perform authorization checks and allow or deny access to your content based on specific criteria.
- For access inside of an Intranet or private networks, disable your CloudFront distribution and provide access to the origin S3 bucket through an internal Application Load Balancer using interface endpoints on AWS PrivateLink for Amazon S3.
  {{% /notice %}}

### Create the Internal Construct Hub Stack

As an Internal Construct Hub Administrator, the first step is to create an instance of Construct Hub in an AWS Account. Before we can use the Construct Hub library in our stack, we need to install the npm module in our project:

{{<highlight bash>}}
npm install construct-hub
{{</highlight>}}

By default, Construct Hub has a single package source configured, which is the public npmjs.com registry. However, it also supports CodeArtifact repositories and custom package source implementations. For our purposes, we will create a CodeArtifact domain and repository to add as a package source for our Internal Construct Hub.

Edit the file under `lib/internal-construct-hub-stack.ts` and use the following code:

{{<highlight typescript>}}
import * as cdk from 'aws-cdk-lib';
import * as codeartifact from 'aws-cdk-lib/aws-codeartifact';
import { ConstructHub } from 'construct-hub';
import * as sources from 'construct-hub/lib/package-sources';
import { Construct } from 'constructs';

export class InternalConstructHubStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create a CodeArtifact domain and repo for user construct packages
    const domain = new codeartifact.CfnDomain(this, 'CodeArtifactDomain', {
      domainName: 'cdkworkshop-domain',
    });

    const repo = new codeartifact.CfnRepository(this, 'CodeArtifactRepository', {
      domainName: domain.domainName,
      repositoryName: 'cdkworkshop-repository',
    });

    repo.addDependency(domain);

    // Create internal instance of ConstructHub, register the new CodeArtifact repo
    new ConstructHub(this, 'ConstructHub', {
      packageSources: [
        new sources.CodeArtifact({ repository: repo })
      ],
    });
  }
}
{{</highlight>}}

## Bootstrapping an Environment
The first time you deploy an AWS CDK app into an environment (account/region),
you can install a "bootstrap stack". This stack includes resources that
are used in the toolkit's operation. For example, the stack includes an S3
bucket that is used to store templates and assets during the deployment process.

You can use the `cdk bootstrap` command to create the bootstrap stack in your AWS Account:

{{<highlight bash>}}
cdk bootstrap
{{</highlight>}}

You should see output like this:
{{<highlight bash>}}
Environment aws://[account-id]/[region] bootstrapped
{{</highlight>}}

## Deploy
Use `cdk deploy` to deploy the CDK app:

{{<highlight bash>}}
cdk deploy
{{</highlight>}}

{{% notice info %}} Deploying the Internal Construct Hub Stack for the first time may take ~10-12 minutes. You can take a break or continue through the Construct Library sections of the workshop {{% /notice %}}

<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/demo-frontend/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# Demo Frontend App

[![Watch the video](https://img.boltops.com/boltopspro/video-preview/multiple/demo-backend.png)](https://youtu.be/p1nX1WppoNo)

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

The demo frontend app is useful for demonstration purposes.  The frontend app calls the [backend demo app](https://github.com/boltopspro-docs/demo-backend) and echos back the response.  This is a Sinatra Ruby app.

## Install

To install the dependencies.

    git clone git@github.com:boltopspro/demo-frontend.git
    cd demo-frontend
    bundle

## Start Server Locally

Start app with `bin/web`. Example:

    $ bin/web
    == Sinatra (v2.0.5) has taken the stage on 8081 for development with backup from Puma
    Puma starting in single mode...
    * Version 3.12.0 (ruby 2.5.3-p105), codename: Llamas in Pajamas
    * Min threads: 0, max threads: 16
    * Environment: development
    * Listening on tcp://0.0.0.0:8081
    Use Ctrl-C to stop

The frontend app runs on port 8081. The frontend app simply makes a network call to the backend app and echos back the response.

Make sure you have the backend app running. Here's the [backend-app README](https://github.com/boltopspro-docs/demo-backend)

Test with curl:

    $ curl localhost:8081
    frontend calling backend. response from backend: 42

## Create ECR Repos

Let's get ready to deploy to ECS.  We'll use AWS_PROFILE to switch between accounts. More info: [Settings Setup](https://github.com/boltopspro-docs/reference-architecture/blob/master/docs/settings-setup.md).

First, create the ECR repos on both AWS acccounts.

    AWS_PROFILE=dev aws ecr create-repository --repository-name demo/frontend
    AWS_PROFILE=prd aws ecr create-repository --repository-name demo/frontend

If you need to list the repo again:

    AWS_PROFILE=dev aws ecr describe-repositories
    AWS_PROFILE=prd aws ecr describe-repositories

If you are using a Single AWS Account, use the commands: [One Account: Create ECR Repos](docs/one-account.md#create-ecr-repos).

## ECS Deployment

Let's deploy the demo app to an ECS cluster. We'll use the [ufo](http://ufoships.com/) tool for deployment.  The project has already been set up with starter `.ufo` files for you. We'll just need to adjust some of the `.ufo` settings.

### Adjust ECR Repos

Adjust the [.ufo/settings.yml](.ufo/settings.yml) file, which looks like this:

```yaml
development:
  image: 112233445566.dkr.ecr.us-west-2.amazonaws.com/demo/frontend
  aws_profile: dev

production:
  image: 778899001122.dkr.ecr.us-west-2.amazonaws.com/demo/frontend
  aws_profile: prd
```

Adjust the [.ufo/settings.yml](.ufo/settings.yml) file to use the ECR repos that were created in the previous step.

If you are using a Single AWS Account, follow the instructions in: [One Account: Adjust ECR Repos](docs/one-account.md#adjust-ecr-repos).

## Adjust Network Settings

Next, adjust the network settings that the ECS service will use.

Since the frontend app is meant to be an external application, the ecs tasks will run on the `PrivateAppSubnets` and the ELB will run on the `PublicSubnets`. Adjust the `.ufo` network related files:

* [.ufo/settings/network/development.yml](.ufo/settings/network/development.yml)
* [.ufo/settings/network/production.yml](.ufo/settings/network/production.yml)

Here's what the [network/development.yml](.ufo/settings/network/development.yml) settings file looks like:

```yaml
vpc: vpc-111
ecs_subnets: # required: at least 2 subnets
  - subnet-111,subnet-222,subnet-333 # use PrivateApp Subnets
elb_subnets: # defaults to same subnets as ecs_subnets when not set
  - subnet-111,subnet-222,subnet-333 # use Public Subnets
```

We can grab the values from the CloudFormation Console Outputs tab of the VPC. Here's an example of the development VPC.

![](https://img.boltops.com/boltopspro/demo-apps/backend/dev-vpc-outputs.png)

Repeat the process for the [network/production.yml](.ufo/settings/network/production.yml) file.

## Adjust ECS Environment Variables

The frontend knows how to connect to the backend endpoint via an environment variable. The environment variable can be configured in the `.ufo/variables` files.

* [.ufo/variables/development.rb](.ufo/variables/development.rb)
* [.ufo/variables/production.rb](.ufo/variables/production.rb)

Here's the [variables/development.rb](.ufo/variables/development.rb) example.

```ruby
@environment = helper.env_vars(%Q[
  BACKEND_ENDPOINT=http://demo-backend-web.dev.private
])
```

It is already preconfigured with the example endpoint we used in the [demo-backend](https://github.com/boltopspro-docs/demo-backend). The setting works if you have configured the route53 DNS.  If you have not configured an pretty DNS name, you can also use the ELB DNS name. The DNS name is available on the ufo managed CloudFormation stack Outputs.

![](https://img.boltops.com/boltopspro/demo-apps/backend/dev-ufo-outputs.png)

## Deploy with ufo ship

Then you're ready to deploy with the `ufo ship` command. For convenience, we'll provide example commands for development and production environments. Use the commands for the environment you're working on.

    UFO_ENV=development ufo ship demo-frontend-web --no-wait
    UFO_ENV=production  ufo ship demo-frontend-web --no-wait

Check the CloudFormation and ECS console for the deployment to complete. It usually take about 5 to 6 minutes. After deployment completes you can grab the DNS endpoint from the ufo managed CloudFormation stack Outputs.  Here's an screenshot example of development:

![](https://img.boltops.com/boltopspro/demo-apps/frontend/dev-ufo-outputs.png)

You can also use the `ufo ps` to get the external ELB DNS endpoint.  Examples:

    $ UFO_ENV=development ufo ps demo-frontend-web
    Elb: demo-fr-Elb-1LI7LP2RLHM3H-508625321.us-west-2.elb.amazonaws.com
    $ UFO_ENV=production  ufo ps demo-frontend-web
    Elb: demo-fr-Elb-DME9QUQKDOOL-1381496041.us-west-2.elb.amazonaws.com

## Test Deployed Frontend

Since we created an external public-facing ELB, we can test the application from anywhere. Here's an example checking the both environments.

    $ curl demo-fr-Elb-1LI7LP2RLHM3H-508625321.us-west-2.elb.amazonaws.com
    frontend calling backend. response from backend: 42
    $ curl demo-fr-Elb-DME9QUQKDOOL-1381496041.us-west-2.elb.amazonaws.com
    frontend calling backend. response from backend: 42

Repeat the process for the production account.

## Go Back to Reference Architecture

That's it. Go back to the main [boltopspro/reference-architecture](https://github.com/boltopspro-docs/reference-architecture/blob/master/README.md)

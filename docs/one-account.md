<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/demo-frontend/blob/master/docs/one-account.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

## One AWS Account Commands

[![Watch the video](https://img.boltops.com/boltopspro/video-preview/single/demo-frontend.png)](https://www.youtube.com/watch?v=t_14fzAtJ9c)

Here are the summarized commands if you're using the One AWS account strategy.

## Create ECR Repos

With a One Account Strategy, we'll create one ECR repo for both development and production.

    AWS_PROFILE=one aws ecr create-repository --repository-name demo/frontend

If you need to list the repo again:

    AWS_PROFILE=one aws ecr describe-repositories

### Adjust ECR Repos

1. Adjust `settings.yml` to use the same `image` value for `development` and `production`.
2. Adjust the `stack_naming` to `append_env`. With `stack_naming: append_env`, ufo will append the environment to the CloudFormation stack names.  Example `settings.yml`:
3. Use `aws_profile: one`

```yaml
base:
  stack_naming: append_env #  on single AWS account setups, append the environment for different CloudFormation stack names

development:
  image: 112233445566.dkr.ecr.us-west-2.amazonaws.com/demo/frontend # same as production
  aws_profile: one
  cfn_profile: development
  network_profile: development

production:
  image: 112233445566.dkr.ecr.us-west-2.amazonaws.com/demo/frontend # same as development
  aws_profile: one
  cfn_profile: production
  network_profile: production
```

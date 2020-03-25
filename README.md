# terraform-aws-elastic-beanstalk-environment

Terraform module to provision AWS Elastic Beanstalk environment




## Usage

**IMPORTANT:** The `master` branch is used in `source` just as an example. In your code, do not pin to `master` because there may be breaking changes between releases.
Instead pin to the release tag (e.g. `?ref=tags/x.y.z`) of one of our [latest releases](https://github.com/ceibo-it/terraform-aws-elastic-beanstalk-environment/releases).



For a complete example, see [examples/complete](examples/complete)

```hcl
  provider "aws" {
    region = var.region
  }

  module "vpc" {
    source     = "git::https://github.com/ceibo-it/terraform-aws-vpc.git?ref=tags/0.0.1"
    namespace  = var.namespace
    stage      = var.stage
    name       = var.name
    cidr_block = "172.16.0.0/16"
  }

  module "subnets" {
    source               = "git::https://github.com/ceibo-it/terraform-aws-dynamic-subnets.git?ref=tags/0.0.1"
    availability_zones   = var.availability_zones
    namespace            = var.namespace
    stage                = var.stage
    name                 = var.name
    vpc_id               = module.vpc.vpc_id
    igw_id               = module.vpc.igw_id
    cidr_block           = module.vpc.vpc_cidr_block
    nat_gateway_enabled  = true
    nat_instance_enabled = false
  }

  module "elastic_beanstalk_application" {
    source      = "git::https://github.com/ceibo-it/terraform-aws-elastic-beanstalk-application.git?ref=tags/0.0.1"
    namespace   = var.namespace
    stage       = var.stage
    name        = var.name
    description = "Test elastic_beanstalk_application"
  }

  module "elastic_beanstalk_environment" {
    source                             = "git::https://github.com/ceibo-it/terraform-aws-elastic-beanstalk-environment.git?ref=master"
    namespace                          = var.namespace
    stage                              = var.stage
    name                               = var.name
    description                        = "Test elastic_beanstalk_environment"
    region                             = var.region
    availability_zone_selector         = "Any 2"
    dns_zone_id                        = var.dns_zone_id
    elastic_beanstalk_application_name = module.elastic_beanstalk_application.elastic_beanstalk_application_name

    instance_type           = "t3.small"
    autoscale_min           = 1
    autoscale_max           = 2
    updating_min_in_service = 0
    updating_max_batch      = 1

    loadbalancer_type       = "application"
    vpc_id                  = module.vpc.vpc_id
    loadbalancer_subnets    = module.subnets.public_subnet_ids
    application_subnets     = module.subnets.private_subnet_ids
    allowed_security_groups = [module.vpc.vpc_default_security_group_id]

    // https://docs.aws.amazon.com/elasticbeanstalk/latest/platforms/platforms-supported.html
    // https://docs.aws.amazon.com/elasticbeanstalk/latest/platforms/platforms-supported.html#platforms-supported.docker
    solution_stack_name = "64bit Amazon Linux 2018.03 v2.12.17 running Docker 18.06.1-ce"

    additional_settings = [
      {
        namespace = "aws:elasticbeanstalk:application:environment"
        name      = "DB_HOST"
        value     = "xxxxxxxxxxxxxx"
      },
      {
        namespace = "aws:elasticbeanstalk:application:environment"
        name      = "DB_USERNAME"
        value     = "yyyyyyyyyyyyy"
      },
      {
        namespace = "aws:elasticbeanstalk:application:environment"
        name      = "DB_PASSWORD"
        value     = "zzzzzzzzzzzzzzzzzzz"
      },
      {
        namespace = "aws:elasticbeanstalk:application:environment"
        name      = "ANOTHER_ENV_VAR"
        value     = "123456789"
      }
    ]
  }
}
```




## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| additional_security_groups | List of security groups to be allowed to connect to the EC2 instances | list(string) | `<list>` | no |
| additional_settings | Additional Elastic Beanstalk setttings. For full list of options, see https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options-general.html | object | `<list>` | no |
| alb_zone_id | ALB zone id | map(string) | `<map>` | no |
| allowed_security_groups | List of security groups to add to the EC2 instances | list(string) | `<list>` | no |
| application_port | Port application is listening on | number | `80` | no |
| application_subnets | List of subnets to place EC2 instances | list(string) | - | yes |
| associate_public_ip_address | Whether to associate public IP addresses to the instances | bool | `false` | no |
| attributes | Additional attributes (e.g. `1`) | list(string) | `<list>` | no |
| autoscale_lower_bound | Minimum level of autoscale metric to remove an instance | number | `20` | no |
| autoscale_lower_increment | How many Amazon EC2 instances to remove when performing a scaling activity. | number | `-1` | no |
| autoscale_max | Maximum instances to launch | number | `3` | no |
| autoscale_measure_name | Metric used for your Auto Scaling trigger | string | `CPUUtilization` | no |
| autoscale_min | Minumum instances to launch | number | `2` | no |
| autoscale_statistic | Statistic the trigger should use, such as Average | string | `Average` | no |
| autoscale_unit | Unit for the trigger measurement, such as Bytes | string | `Percent` | no |
| autoscale_upper_bound | Maximum level of autoscale metric to add an instance | number | `80` | no |
| autoscale_upper_increment | How many Amazon EC2 instances to add when performing a scaling activity | number | `1` | no |
| availability_zone_selector | Availability Zone selector | string | `Any 2` | no |
| default_security_group_enabled | Enable default security group with allowed_security_groups in inbound rules, and allowing all for outbound | bool | `true` | no |
| delimiter | Delimiter to be used between `name`, `namespace`, `stage`, etc. | string | `-` | no |
| description | Short description of the Environment | string | `` | no |
| dns_subdomain | The subdomain to create on Route53 for the EB environment. For the subdomain to be created, the `dns_zone_id` variable must be set as well | string | `` | no |
| dns_zone_id | Route53 parent zone ID. The module will create sub-domain DNS record in the parent zone for the EB environment | string | `` | no |
| elastic_beanstalk_application_name | Elastic Beanstalk application name | string | - | yes |
| elb_scheme | Specify `internal` if you want to create an internal load balancer in your Amazon VPC so that your Elastic Beanstalk application cannot be accessed from outside your Amazon VPC | string | `public` | no |
| enable_log_publication_control | Copy the log files for your application's Amazon EC2 instances to the Amazon S3 bucket associated with your application | bool | `false` | no |
| enable_stream_logs | Whether to create groups in CloudWatch Logs for proxy and deployment logs, and stream logs from each instance in your environment | bool | `false` | no |
| enhanced_reporting_enabled | Whether to enable "enhanced" health reporting for this environment.  If false, "basic" reporting is used.  When you set this to false, you must also set `enable_managed_actions` to false | bool | `true` | no |
| env_vars | Map of custom ENV variables to be provided to the application running on Elastic Beanstalk, e.g. env_vars = { DB_USER = 'admin' DB_PASS = 'xxxxxx' } | map(string) | `<map>` | no |
| environment_type | Environment type, e.g. 'LoadBalanced' or 'SingleInstance'.  If setting to 'SingleInstance', `rolling_update_type` must be set to 'Time', `updating_min_in_service` must be set to 0, and `loadbalancer_subnets` will be unused (it applies to the ELB, which does not exist in SingleInstance environments) | string | `LoadBalanced` | no |
| force_destroy | Force destroy the S3 bucket for load balancer logs | bool | `false` | no |
| health_streaming_delete_on_terminate | Whether to delete the log group when the environment is terminated. If false, the health data is kept RetentionInDays days. | bool | `false` | no |
| health_streaming_enabled | For environments with enhanced health reporting enabled, whether to create a group in CloudWatch Logs for environment health and archive Elastic Beanstalk environment health data. For information about enabling enhanced health, see aws:elasticbeanstalk:healthreporting:system. | bool | `false` | no |
| health_streaming_retention_in_days | The number of days to keep the archived health data before it expires. | number | `7` | no |
| healthcheck_url | Application Health Check URL. Elastic Beanstalk will call this URL to check the health of the application running on EC2 instances | string | `/healthcheck` | no |
| http_listener_enabled | Enable port 80 (http) | bool | `true` | no |
| instance_refresh_enabled | Enable weekly instance replacement. | bool | `true` | no |
| instance_type | Instances type | string | `t2.micro` | no |
| keypair | Name of SSH key that will be deployed on Elastic Beanstalk and DataPipeline instance. The key should be present in AWS | string | `` | no |
| loadbalancer_certificate_arn | Load Balancer SSL certificate ARN. The certificate must be present in AWS Certificate Manager | string | `` | no |
| loadbalancer_managed_security_group | Load balancer managed security group | string | `` | no |
| loadbalancer_security_groups | Load balancer security groups | list(string) | `<list>` | no |
| loadbalancer_ssl_policy | Specify a security policy to apply to the listener. This option is only applicable to environments with an application load balancer | string | `` | no |
| loadbalancer_subnets | List of subnets to place Elastic Load Balancer | list(string) | `<list>` | no |
| loadbalancer_type | Load Balancer type, e.g. 'application' or 'classic' | string | `classic` | no |
| logs_delete_on_terminate | Whether to delete the log groups when the environment is terminated. If false, the logs are kept RetentionInDays days | bool | `false` | no |
| logs_retention_in_days | The number of days to keep log events before they expire. | number | `7` | no |
| managed_actions_enabled | Enable managed platform updates. When you set this to true, you must also specify a `PreferredStartTime` and `UpdateLevel` | bool | `true` | no |
| name | Solution name, e.g. 'app' or 'cluster' | string | - | yes |
| namespace | Namespace, which could be your organization name, e.g. 'eg' or 'cp' | string | `` | no |
| preferred_start_time | Configure a maintenance window for managed actions in UTC | string | `Sun:10:00` | no |
| region | AWS region | string | - | yes |
| rolling_update_enabled | Whether to enable rolling update | bool | `true` | no |
| rolling_update_type | `Health` or `Immutable`. Set it to `Immutable` to apply the configuration change to a fresh group of instances | string | `Health` | no |
| root_volume_size | The size of the EBS root volume | number | `8` | no |
| root_volume_type | The type of the EBS root volume | string | `gp2` | no |
| solution_stack_name | Elastic Beanstalk stack, e.g. Docker, Go, Node, Java, IIS. For more info, see https://docs.aws.amazon.com/elasticbeanstalk/latest/platforms/platforms-supported.html | string | - | yes |
| ssh_listener_enabled | Enable SSH port | bool | `false` | no |
| ssh_listener_port | SSH port | number | `22` | no |
| stage | Stage, e.g. 'prod', 'staging', 'dev', or 'test' | string | `` | no |
| tags | Additional tags (e.g. `map('BusinessUnit`,`XYZ`) | map(string) | `<map>` | no |
| tier | Elastic Beanstalk Environment tier, 'WebServer' or 'Worker' | string | `WebServer` | no |
| update_level | The highest level of update to apply with managed platform updates | string | `minor` | no |
| updating_max_batch | Maximum number of instances to update at once | number | `1` | no |
| updating_min_in_service | Minimum number of instances in service during update | number | `1` | no |
| version_label | Elastic Beanstalk Application version to deploy | string | `` | no |
| vpc_id | ID of the VPC in which to provision the AWS resources | string | - | yes |
| wait_for_ready_timeout | The maximum duration to wait for the Elastic Beanstalk Environment to be in a ready state before timing out | string | `20m` | no |




## Outputs

| Name | Description |
|------|-------------|
| all_settings | List of all option settings configured in the environment. These are a combination of default settings and their overrides from setting in the configuration |
| application | The Elastic Beanstalk Application specified for this environment |
| autoscaling_groups | The autoscaling groups used by this environment |
| ec2_instance_profile_role_name | Instance IAM role name |
| elb_zone_id | ELB zone id |
| endpoint | Fully qualified DNS name for the environment |
| hostname | DNS hostname |
| id | ID of the Elastic Beanstalk environment |
| instances | Instances used by this environment |
| launch_configurations | Launch configurations in use by this environment |
| load_balancers | Elastic Load Balancers in use by this environment |
| name | Name |
| queues | SQS queues in use by this environment |
| security_group_id | Security group id |
| setting | Settings specifically set for this environment |
| tier | The environment tier |
| triggers | Autoscaling triggers in use by this environment |




## Copyright

Copyright © 2020 [Ceibo IT](https://ceibo.it/copyright)




## License 

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) 

See [LICENSE](LICENSE) for full details.

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.




## Trademarks

All other trademarks referenced herein are the property of their respective owners.



## NOTICE

terraform-aws-elastic-beanstalk-environment
Copyright 2020 Ceibo


This product includes software developed by
Cloud Posse, LLC (c) (https://cloudposse.com/)
Licensed under Apache License, Version 2.0
Copyright © 2017-2019 Cloud Posse, LLC

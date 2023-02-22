# test# Descripion
This repository is used for deploying all business central related resources

# Terraform Version
"> 0.14.x"

# Dependencies
1. This repository will create resources in 2 different AWS accounts, in order to do that you will need to export 3 different variables from `FI.SPAN AWS Account - infra-nonprod-app` AWS environment variables:
```shell
export TF_VAR_aws_fispan_infra_nonprod_app_account_access_key=""
export TF_VAR_aws_fispan_infra_nonprod_app_account_secret_key=""
export TF_VAR_aws_fispan_infra_nonprod_app_account_token=""
```
2.   secrets.tfvar file for database, encryption, user and pass
```shell
rds_password = "business-central onelogin secure notes"
secrets = {
  bc1 = { ## this key must be the same name we have in bc_instances.container_name in bi.tfvars file
    database_creds = {
      database   = "password used in Environment Provisioning Step 9"
      encryption = "strong random password"
    },
    user_creds = {
      user = "name of the admin user for the environment"
      pass = "strong random password"
    }
  },
  .
  .
  .
}
```
3. ECS Cluster

# Resources
* Subnets
* Security groups
* S3 bucket
* RDS database
* IAM roles
* ECS Tasks/Services/Loadbalanacers

# Parameters
parameter | type | example | description
----------|------|---------|------------
 aws_region | string | us-west-2 | the primary region
 sgs_description | string |  BC RDS database | description of the security group for RDS
 rds_password | string | "" | password for RDS database loaded from external secrets file
 peering_connections | list(map) | [{ "name" = "vpn-bc-non-prod-us-west-2a" "route_table" = "vpn-bc-non-prod-us-west-2a" "cidr" = "172.20.0.0/20" "peering_connection" = "pcx-06092f7ad62aaec69" }] | a list of maps containing peering connections to add
 ecs_iam_role_actions | list(map) | { actions = ["s3:GetObject"] resource = ["arn::s3..."] effect = "Allow"} | a list of maps where each one represents the necessary rules for the ecs task role
 kms_key_arn | string | arn:aws:kms:us-west-2... | the arn of the KMS key to use to encrypt all bc resources
 bi_cluster | string | bi-non-prod | name of the ecs cluster to deploy to
 bc_name_prefix | string | bc-non-prod | the name to attach to all resources
 bi_vpc_id | string | vpc-12345678990 | the id of the VPC that resources are being deployed into
 rds_sgs_rules | list(map()) | { name= "web-ingress" type= "ingress" from_port = 443 to_port= 443 protocol= "TCP" cidr_blocks = ["0.0.0.0/0"] } | a map of secrutity groups to add for the RDS instance
 rds_iam_role_actions | list(map) | { actions = ["s3:GetObject"] resource = ["arn::s3..."] effect = "Allow"} | a list of maps where each one represents the necessary rules for the rds role
 public_subnets | list(string) | ["subnet-1234566", "subnet-7894568"] | list of subnets to use for load balancers
 subnet_cidrs | list(map()) | [ {"name" = "bc-non-prod-rds-us-west-2a" "az" = "us-west-2a" "cidr" = "172.20.1.0/27"}] | a list of maps that contain the values necessary to create a subnet
 nat_gateways | map() | {us-west-2a: nat123456789} | a map of AWS regions to nat gateway ids
 bc_instances | list(map) | various | a list of maps where each map contains all necessary parameters to configure a ecs task/service/lb module
 tags | map(string) | {"creator" = "soso@fispan.com "} | collection of tags to apply to resources

# Running

  ```shell
  rm -Rf .terraform/ #critical to remove the terraform state inbetween different environment runs
  
  # INITIALIZE
  terraform init --backend-config ./backends/bi.tfvars 

  # First PLAN
  terraform plan -target=module.bc-instances --var-file envs/bi.tfvars --var-file secrets.tfvars

  # First APPLY
  terraform apply -target=module.bc-instances --var-file envs/bi.tfvars --var-file secrets.tfvars

  # Second PLAN
  terraform plan --var-file envs/bi.tfvars --var-file secrets.tfvars

  # Second APPLY
  terraform apply --var-file = ./envs/bi.tfvars --var-file ./secrets.tfvars
  ```

# Comments
1. You must run terraform for creating new environments according to the above section `Running` 
2. MS RDS can take up to 20 minutes to deploy
3. The password should be put into a seperate secrets.tfvars file that is not commited to the repo
4. When performing updates, it is common to see module.bc-instances["business-central-1"].aws_load_balancer_policy.policy being replaced for every environment


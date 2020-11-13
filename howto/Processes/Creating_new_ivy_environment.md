# Setup ivy environment (works on commercial and GovCloud AWS)

## Step 0 - AMIs, S3 Buckets, ACM certificate and Route53 Hosted Zone

**NOTE:** It is required the account you are going to setup ivy in has
already joined the master/parent accounts, do not continue until you
have accepted the invitation.

### AMIs

Note: Using [ami-bakery](https://github.com/nxtlytics/ivy-ami-bakery)

AMIs are created by AMI Builder in a tools account then:

1.  EC2 console
2.  AMIs section
3.  Update the latest set of AMIs to share the images with the newly
    created Accounts

### S3 Buckets

Ivy requires at least 2 buckets:

-   ivy-\<sysenv shortname\>-infra
-   ivy-\<sysenv shortname\>-logs

These if you'd like you can create these using terraform/terragrunt.

**Note:** terraform/terragrunt plays well with a s3 bucket:
`ivy-<sysenv shortname>-ivy-terragrunt-core` 

You can use the [terraform s3 bucket](https://github.com/nxtlytics/ivy-terraform-modules/tree/master/aws/s3-terraform-state-bucket), take a look at the [default policy](./AWS_policies.md#simple-block-everything-not-coming-from-within-the-org):

### Route53 Hosted Zone

Create a public hosted zone within the account following nomenclature:
\<sysenv shortname\>.example.com and then add the NS records to the main
example.com hosted zone in the master account. This is done using
terraform/terragrunt

**Note: **GovCloud does not let you create public hosted zones just
create the hosted zone in the main account and then setup the
delegation here also, youn may use our [terraform route53 module](https://github.com/nxtlytics/ivy-terraform-modules/tree/master/aws/route53)

### Wildcard certificate

Create the wildcard certificate within the account so resources can use
it, this can be done using our terraform [ACM certificates module](https://github.com/nxtlytics/ivy-terraform-modules/tree/master/aws/acm_certificates):

-   Example certificate creation:

    **terragrunt output**  Expand source

    ```bash
    $ cd path/to/acm_certificates/star-example.com/
    $ terragrunt output
    ...
    aws_acm_certificate-this-domain_validation_options = [
      {
        "domain_name" = "example.com"
        "resource_record_name" = "_7ce69558b28ae0d84dfdbd7b997b5c4b.example.com."
        "resource_record_type" = "CNAME"
        "resource_record_value" = "_8f18fc3b475133a299ab1a2923f98f30.vhzmpjdqfx.acm-validations.aws."
      },
      {
        "domain_name" = "*.example.com"
        "resource_record_name" = "_7ce69558b28ae0d84dfdbd7b997b5c4b.example.com."
        "resource_record_type" = "CNAME"
        "resource_record_value" = "_8f18fc3b475133a299ab1a2923f98f30.vhzmpjdqfx.acm-validations.aws."
      },
    ]
    ```

-   You can then add these records using our [terraform route53 module](https://github.com/nxtlytics/ivy-terraform-modules/tree/master/aws/route53)

**Note:** Wait until AWS has issued the certificate

### Default Public SSH Key

**Note:** Remember to add your default public ssh key to EC2 in the new account

## *NOTE:* when the document below mentions rain it refer to this [repo](https://github.com/nxtlytics/ivy-rain)

## Step 1 - Create VPC and VPN

### Add sysenv shortname block to config/constants.py

-   Example:
    [dev](https://github.com/nxtlytics/ivy-rain/blob/master/config/constants.py#L14-L153)
    and [SSL certificate](https://github.com/nxtlytics/ivy-rain/blob/master/config/constants.py#L224-L232)
-   Make sure to update your documentation with the new names and subnets

### Create VPC

**Create VPC Cloudformation Template**

```bash
AWS_PROFILE=<newaccount> ./rain.py <sysenv shortname> apply --template VPC
AWS_PROFILE=<newaccount> ./rain.py <sysenv shortname> apply --template SecurityGroups
```

### Add VPC to transit gateway

Login to the new account's AWS console:

1.  Go to VPC Dashboard
2.  Transit gateway attachments
3.  Create Transit Gateway Attachments → select private subnets on each
    availability zone (If you do not see a transit gateway you have not
    join a master account)
4.  Login to transit account → VPC → Transit gateway attachment → accept
    the new attachment
5.  Still the transit account, Go to Transit gateway route tables and
    add the new subnet to it
6.  Now you can add the new account's subnet to route tables on all
    accounts you want to have access to (Remember you also need to do
    the same on the new account's subnets)

### Create Pritunl Box

**Note: ** This box requires access to the MongoDB cluster

**Create pritunl cloudformation template**

```bash
AWS_PROFILE=<newaccount> ./rain.py <sysenv shortname> apply --template Pritunl
```

### Configure pritunl host in vpn.example.com

Login as the pritunl administrator user on vpn.example.com and confirm the
new host is present

1.  Create New server 
2.  Attach new host to new server
3.  Add New Route (new subnet) to new server
4.  Attach organization to new server (Follow examples from step 1)
5.  Remove routes, hosts and orgs that the new server does not need
6.  start the new server
7.  Test it works and update

## Step 2 - Create Mesos Masters

### Create the cloudformation template

**Create Mesos Master Cloudformation Templates**

```bash
AWS_PROFILE=<newaccount> ./rain.py <sysenv shortname> apply --template MesosMasters
```

### Configure consul on the pritunl box

**Configure consul for VPN users to be able to resolve hostnames in this
sysenv**

```bash
While on the new VPN
$ ssh 10.255.<new account octet>.1
$ bash -x /opt/ivy/configure_consul.sh
```

### Confirm Mesos, Marathon, Chronos and Consul are working

- http://master.mesos.service.\<sysenv shortname\>.ivy:8500/ui/\<sysenv shortname\>/services
- The URL above should show all health checks passed

## Step 3 - Import base consul KV config

Follow Instructions [here](https://github.com/nxtlytics/ivy-rain/blob/master/scripts/consul-kv-tool/README.md)

## Step 4 - Create the rest of the infrastructure

**Create the rest of the infrastructure**

```bash
for i in 'Cassandra' 'Kafka' 'MesosAgents'; do AWS_PROFILE=<newaccount> ./rain.py <sysenv shortname> apply --template "${i}"; done
DB_app_PASS='<REDACTED>' AWS_PROFILE=<newaccount> ./rain.py <sysenv shortname> apply --template RDS
```

### Monitor RDS using DataDog

See [Metrics in Ivy](../Metrics_in_Datadog.md) and [datadog-agent container](https://github.com/nxtlytics/ivy-base-containers/tree/master/datadog-agent)

## Step 5 - (GovCloud Only) Create Route53 records

As mentioned above using our [terraform route53 module](https://github.com/nxtlytics/ivy-terraform-modules/tree/master/aws/route53)

## Step 6 - Start Adding applications on Marathon

http://master.mesos.service.\<sysenv shortname\>.example.com:8080/ui/\#/apps

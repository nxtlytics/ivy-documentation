# Creating new AWS / GovCloud accounts


# Creating a new commercial AWS account

1.  Retrieve IAM credentials for commercial organization payer
    commercial account and set them up in your `~/.aws/credentials`
    and/or `~/.aws/config`

2.  Retrieve the SAML Document from your Identity Provider (IDP)
    **References:**
    - [G Suite - How to setup custom SAML application](https://support.google.com/a/answer/6087519?hl=en)

3.  Clone https://github.com/nxtlytics/ivy-accounts-tools

``` bash
git clone git@github.com:nxtlytics/ivy-accounts-tools.git
```

or

```bash
git clone https://github.com/nxtlytics/ivy-accounts-tools.git
```

4.  Create a new sub account

``` bash
AWS_PROFILE=regular-aws pipenv run python setup_account.py -a <accountname> -e infeng+<accountname>@example.com \
    -f </path/to/saml/document.xml> -s <SAML_PROVIDER, defaults to gsuite> -t <IVY_TAG, defaults to ivy> \
    [-l {CRITICAL,ERROR,WARNING,INFO,DEBUG}]
```

**Example Output:**

``` bash
$ aws --profile regular-aws organizations list-accounts --query 'Accounts[?Name == `ivy-sandbox-dev`].Id | [0]' --output text
260432279923
```


5.  Add the commercial SSO role to users in G Suite

    1.  Open G Suite Admin

    2.  Users -> select user to edit -> User Information -> **AWS SSO**

    3.  Add `arn:aws:iam::<new account id>:role/SSOAdministratorAccess,arn:aws:iam::<new account id>:saml-provider/<IVY_TAG>-<SAML_PROVIDER>` or `arn:aws:iam::<new account id>:role/SSOViewOnlyAccess,arn:aws:iam::<new account id>:saml-provider/<IVY_TAG>-<SAML_PROVIDER>` (accordingly)

    4.  Ensure duration is set to **28800**

    5.  Rinse and repeat for all users that require access to new account

6.  Configure local SAML credentials for new commercial account. Add the following to your **`~/.saml2aws`** configuration file:

``` text
[<new account name>]
app_id               =
url                  = <URL from G Suite's SAML>
username             = <you>@example.com
provider             = GoogleApps
mfa                  = Auto
skip_verify          = false
timeout              = 0
aws_urn              = urn:amazon:webservices
aws_session_duration = 28800
aws_profile          = <new account name>
resource_id          =
subdomain            =
role_arn             = arn:aws:iam::<new account id>:role/SSOAdministratorAccess
region               = us-west-2
```

    The SPid above is specific to the commercial/gov providers in the G Suite console.

    You can now use **`saml2aws login -a <new account name>`** to log into the new account with your personal user credentials.  
    From now on, you can use **`aws --profile <new account name> ...`**
    instead of the assumed role that was created earlier

7. Recover the root password for the new account, and add it to 1Password

    1.  Go to <https://console.aws.amazon.com/console/home>

    2.  Enter the email address you used to create the account above

    3.  Click "Forgot password"

    4.  An email will be sent to the email address to allow password reset

8. Store new root credentials on your password manager

# Creating GovCloud accounts

1.  Create GovCloud account pairing using the root credentials in the Web Console  
    \*NOTE:\* This step cannot be done via the CLI unless you have IAM credentials for the root user (in this case, you will not have these)  
    \*NOTE2:\* You MUST be a US citizen to complete this step.
    Completing this step as a non-US citizen probably results in paperwork and general sadness.
    1.  Sign into the commercial account created earlier

    2.  Go to [https://console.aws.amazon.com/billing/home?\#/account](https://console.aws.amazon.com/billing/home#/account)

    3.  Click "Sign up for AWS GovCloud (US)"  
        ![GovCloud-button.png](../images/Creating_new_AWS_GovCloud_accounts/GovCloud-button.png)

    4.  Select the free support plan.  
        - If nothing happens, click it again. You might get redirected
        to the home page. Start over.

    5.  Fill out the scary form

        This may take a while to complete. AWS will send you an email
        once access is approved

2.  Wait until you get an email from AWS, then head to
    <https://portal.aws.amazon.com/billing/signup/web/v1.0/govcloud/signup>
    and complete the signup process

3.  Save the GovCloud Administrator account credentials in 1Password

4.  Create IAM credentials for the GovCloud Administrator account, and
    add them to your **`~/.aws/credentials`** file

5.  Add the account alias

    ``` bash
    aws --profile <newaccount> iam create-account-alias --account-alias <accountname>
    ```

6.  Set up the SSO provider

    ``` bash
    aws --profile <newaccount> iam create-saml-provider --saml-metadata-document file://path/to/commercial/IDPMetadata.xml --name ivy-gsuite
    ```

    You can download the SAML provider from an existing account if necessary or download it from 1password

7.  Add the SSO AdministratorAccess and viewer roles and trust the SSO
    provider

    ``` bash
    # SSOAdministratorAccess
    aws --profile <newaccount> iam create-role --role-name SSOAdministratorAccess --max-session-duration 28800 --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Action":"sts:AssumeRoleWithSAML","Effect":"Allow","Condition":{"StringEquals":{"SAML:aud":"https://signin.amazonaws-us-gov.com/saml"}},"Principal":{"Federated":"arn:aws-us-gov:iam::<new account id>:saml-provider/ivy-gsuite"}}]}'
    aws --profile <newaccount> iam attach-role-policy --role-name SSOAdministratorAccess --policy-arn arn:aws-us-gov:iam::aws:policy/AdministratorAccess
    # SSOViewOnlyAccess
    aws --profile <newaccount> iam create-role --role-name SSOViewOnlyAccess --max-session-duration 28800 --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Action":"sts:AssumeRoleWithSAML","Effect":"Allow","Condition":{"StringEquals":{"SAML:aud":"https://signin.amazonaws-us-gov.com/saml"}},"Principal":{"Federated":"arn:aws-us-gov:iam::<new account id>:saml-provider/ivy-gsuite"}}]}'
    aws --profile <newaccount> iam attach-role-policy --role-name SSOViewOnlyAccess --policy-arn arn:aws-us-gov:iam::aws:policy/job-function/ViewOnlyAccess
    ```

8.  Add the SSO role to users in G Suite

    1.  Open G Suite Admin

    2.  Users -\> select user to edit -\> User Information -\> **AWS
        SSO**

    3.  Add
        **arn:aws-us-gov:iam:::role/SSOAdministratorAccess,arn:aws-us-gov:iam:::saml-provider/ivy-gsuite**
        or **arn:aws-us-gov:iam:::role/SSOViewOnlyAccess,arn:aws-us-gov:iam:::saml-provider/ivy-gsuite**
        (accordingly)

    4.  Ensure duration is set to **28800**

    5.  Rinse and repeat for all users that require access to new
        account

9.  Configure local SAML credentials for the GovCloud account. Add the following to your **`~/.saml2aws`** configuration file

``` text
[<new account name>]
app_id               = <URL from G Suite's SAML>
url                  = 
username             = <you>@example.com
provider             = GoogleApps
mfa                  = Auto
skip_verify          = false
timeout              = 0
aws_urn              = urn:amazon:webservices
aws_session_duration = 28800
aws_profile          = <new account name>
resource_id          =
subdomain            =
role_arn             = arn:aws-us-gov:iam::<new account id>:role/SSOAdministratorAccess
region               = us-gov-west-1
```

    See, I told you the SPid is different! Learn from my mistakes.

10. Remove IAM credentials for the GovCloud Administrator account

11. Invite the new account to the AWS GovCloud organization (you need to
    login to `ivy-aws-us-gov-west-1-master-prod`  and send the invite to
    \<new account id\>)

12. Remove default VPCs from accounts (from all regions)

    you may want to use
    [vpc_cleaner.py](https://github.com/nxtlytics/ivy-accounts-tools/blob/master/vpc_cleaner/vpc_cleaner.py),
    make sure you understand how to use it and execute a dry run before
    actually making the changes.

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

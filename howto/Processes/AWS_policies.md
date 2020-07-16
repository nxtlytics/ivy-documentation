# AWS policies

# Overview

This is the place where you'll find best practices for creating AWS
policies

# Simple block everything not coming from within the org

The following example is a simple way to make sure the call came from
inside our house, the policy looks like below:

**Block everything not coming from within the org template**

``` java
{
    "Version": "2012-10-17",
    "Id": "Policy1580848985515",
    "Statement": [
        {
            "Sid": "DenyEverythingExceptOrg",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "*:*",
            "Resource": [
                "arn:<aws or aws-us-gov or aws-cn>:<Rest of ARN>",
                "arn:<aws or aws-us-gov or aws-cn>:<Rest of another ARN>"
            ],
            "Condition": {
                "StringNotEquals": {
                    "aws:PrincipalOrgID": "<o-xxxxxxxxx>"
                }
            }
        }
    ]
}
```

Pitfalls

1.  Not least privilege.
2.  It will not allow the contents of the bucket to become public.

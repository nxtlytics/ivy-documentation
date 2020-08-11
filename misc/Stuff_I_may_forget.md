# 01 - Stuff I may forget

# AWS

## AWS services in Scope by Compliance Program

[AWS Services in Scope](https://aws.amazon.com/compliance/services-in-scope/)

## About Global services (e. g. IAM, Route53)

### Cloudtrail

Remember that changes to global services are only available on CloudTrail under the `us-east-1`  region.

## Cloudfront

### Origin Access Identity (OAI) to use with private s3 buckets

- [Cloudfront + Private S3](https://aws.amazon.com/premiumsupport/knowledge-center/cloudfront-access-to-amazon-s3/)
    -   [Youtube Video Explanation](https://youtu.be/x95vzaeEsRk)

## GovCloud Endpoints

[AWS GovCloud Endpoints](https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/using-govcloud-endpoints.html)

## Route53

For record types that include a domain name, enter a fully qualified domain name, for example, www.example.com. The trailing dot is optional.
[Source](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/ResourceRecordTypes.html)

Subdomain Takeover

-   [ubiquiti - longer explanation](https://hackerone.com/reports/145224)
-   [data.gov](https://hackerone.com/reports/317005)

# Azure

## Azure services by FedRAMP and DoD CC SRG audit scope

[Azure Services in Scope](https://docs.microsoft.com/en-us/azure/azure-government/compliance/azure-services-in-fedramp-auditscope)

## Organize Azure Resources

Azure provides four levels of management scope: management groups, subscriptions, resource groups, and resources. The following image shows
the relationship of these levels.

![azure-resource-organization.png](./images/azure-resource-organization.png)

Source: [Azure docs](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-setup-guide/organize-resources?tabs=AzureManagmentGroupsAndHierarchy) and [version date](https://github.com/MicrosoftDocs/cloud-adoption-framework/blob/6ce592263ee78c731760b0e9603b6ec5173911ee/docs/ready/azure-setup-guide/organize-resources.md) 09 Apr 2019 

## Upgrade to a general-purpose v2 storage account

[Upgrade storage account](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-upgrade?tabs=azure-portal)

## What does "External Azure Active Directory" mean?

Anyone who shows up as "External Azure Active Directory" is a user who is already present in another Azure AD and have used Office365 email to redeem the invite.

[Source](https://github.com/MicrosoftDocs/azure-docs/issues/29140)

# Certbot

## How does a certbot route53 dns challenge looks like?

**Event name: ChangeResourceRecordSets**

```bash
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "AssumedRole",
        "principalId": "<Redacted>:<User>",
        "arn": "arn:aws:sts::<Redacted>:assumed-role/<Role Name>/<Redacted>",
        "accountId": "<Redacted>",
        "accessKeyId": "<Redacted>",
        "sessionContext": {
            "sessionIssuer": {
                "type": "Role",
                "principalId": "<Redacted>",
                "arn": "arn:aws:iam::<Redacted>:role/<Role Name>",
                "accountId": "<Redacted>",
                "userName": "<Role Name>"
            },
            "webIdFederationData": {},
            "attributes": {
                "mfaAuthenticated": "false",
                "creationDate": "2020-01-06T18:49:29Z"
            }
        }
    },
    "eventTime": "2020-01-06T18:50:43Z",
    "eventSource": "route53.amazonaws.com",
    "eventName": "ChangeResourceRecordSets",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "<Redacted>",
    "userAgent": "Boto3/1.9.241 Python/2.7.16 Darwin/18.6.0 Botocore/1.12.241",
    "requestParameters": {
        "hostedZoneId": "<Redacted>",
        "changeBatch": {
            "changes": [
                {
                    "action": "DELETE",
                    "resourceRecordSet": {
                        "type": "TXT",
                        "tTL": 10,
                        "resourceRecords": [
                            {
                                "value": "\"<Redacted>\""
                            }
                        ],
                        "name": "_acme-challenge.<record you want a certificate for>"
                    }
                }
            ],
            "comment": "certbot-dns-route53 certificate validation DELETE"
        }
    },
    "responseElements": {
        "changeInfo": {
            "status": "PENDING",
            "id": "/change/C1LIPXHPO7OHSA",
            "comment": "certbot-dns-route53 certificate validation DELETE",
            "submittedAt": "Jan 6, 2020 6:50:43 PM"
        }
    },
    "additionalEventData": {
        "Note": "Do not use to reconstruct hosted zone"
    },
    "requestID": "<Redacted>",
    "eventID": "<Redacted>",
    "eventType": "AwsApiCall",
    "apiVersion": "2013-04-01",
    "recipientAccountId": "<Redacted>"
}
```

# DNS

## How DNS works

[How DNS works](https://howdns.works/)

# Linux

## STIG-Partitioned Enterprise Linux (spel)

[plus3it/spel](https://github.com/plus3it/spel)

# Python

## "is" vs "=="

```python
>>> a = [1, 2, 3]
>>> b = a
>>> a is b
True
>>> a == b
True
>>> c = list(a)
>>> a == c
True
>>> a is c
False
# • "is" expressions evaluate to True if two
#   variables point to the same object
# • "==" evaluates to True if the objects
#   referred to by the variables are equal
```

# Windows

## How to retrieve hidden (dotted/starred) passwords from GUIs

1.  Download 32-bit or 64-bit version from [here](https://www.nirsoft.net/utils/bullets_password_view.html)
    (depending on the arch of the application you want to extract from)
2.  Disable anti-virus real time protection (even Windows Defender)
3.  Extract files from the file downloaded on step 1
4.  Run the executable you extracted on previous step
5.  Open application that has hidden password and you should see the password on `bullets password view` 
6.  Done!

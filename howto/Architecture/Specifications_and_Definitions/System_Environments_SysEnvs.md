# System Environments (SysEnvs)

Scope

Define what a system environment (“sysenv”) is, what it contains, how to
address it, and it’s architecture.

# What is a SysEnv?

A SysEnv is a deployment of a collection of components in a cloud or
on-premise environment. This term was chosen to represent the collection
of Ivy deployed software without confusion between any of the
following terms:

- Geographic environment
- Deployment separation/classification environments
- Customer or product specific terminology

A SysEnv's name should be something that does not pose a serious risk if
exposed outside of ivy but should be internally consistent when
communicating about systems.

SysEnv names are immutable, and will not be changed once created.

## Components

The fundamental components that go in to deriving a SysEnv.

### Usage Definitions

- Key Name - the "key" that should be used to define a key-value object/space for the values.
  - tag keys
  - object keys when serializing/deserializing (subject to language/format formatting guidelines)
  - should be single word values when adding additional components
  - always lowercase when written and/or styled to indicate it is a technical term with a definition (formatting example: "This project does not yet have a target `provider` or `site`." OR (when code/pre-formatted text is unavailable) "This project does not yet have a target *provider* or *site*.")
  - can be namespaced like `sysenv-<key-name>` (e.g., `sysenv-provider` ) when context is unclear.

### Component Definitions

In the Ivy parlance, a SysEnv maps to a hyphenated combination of:

|     | Concept                                 |  Key Name | Example(s)                                                                                                                                                                                                                                                                                      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         Notes and Description |
| --- | ---                                     |       --- | ---                                                                                                                                                                                                                                                                                             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           --- |
|   1 | Namespace (Fixed String)                | namespace | `ivy` - ivy managed (e.g., corporate infrastructure, ivy managed cloud accounts/organizations, etc.)                                                                                                                                                                                            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        This allows us to scope our SysEnv names in providers where names must be globally unique across the provider's entire customer base.<br>**Example:** GCP Projects and AWS S3 buckets must have globally unique names and identifiers. |
|   2 | Infrastructure Provider and/or Owner    |  provider | `ivy` - ivy owned (e.g., corporate infrastructure)<br><br>`aws/aws-gov/aws-cn` - AWS cloud environments<br><br>`gcp` - Google cloud platform<br><br>`<xyz>` - short code for entity (enclaves, on-premise deployments outside of our control)                                                                  | This should be a short code or code word that helps us understand the scope of managing entity of the underlying infrastructure.<br><br>For Ivy managed providers, this should be the cloud provider, data center provider, or ivy owned building/space.<br><br>For enclaves or non-Ivy managed providers, this should be a short code or code word that helps us understand the scope of managing entity of the underlying infrastructure.<br><br>**NEEDS SIGN OFF/SUGGESTION:** If we are using code words (to keep the SysEnv from being sensitive externally) choose from succulent plant taxonomy with a preference towards sharing meaning or alliterative commonality (see and dig into specific examples from wikipedia). Collapse multiword names into a single word without spaces or segmentation.<br><br>E.g.,<br><br>- blueagave<br>- saguaro |
|   3 | Geographic Location or Site Delineation (if there is one) |      site | AWS/GCP/Azure/Most Cloud Providers:<br><br>- Should not need one <br>- You may do use to indicate an account should only use a single region <br><br>Data centers/Colocation:<br><br>- `us` - single location (nationwide, etc)<br>- `us-1` - a location within USA                                                                                                              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               Use provider naming if possible, otherwise simply use us or us-<number> for any opaque deployments.<br>Do not create additional codenames in the absence of a provided name. Simply fall back to the opaque naming scheme, as the names do not matter in 3rd party deployments. |
|   4 | Purpose                                 |   purpose | `master` - billing / security account<br><br>`app` - main product account<br><br>`tools` - general build tooling / repositories<br><br>`corp` - corporate utilities<br><br>`compliance` - compliance enforcement lambdas<br><br>`logarchive` - cloudtrail logging, app log retention<br><br>... |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      Consider utilizing pre-existing app SysEnvs for new product deployments unless the product at hand requires isolation at an security/logical/compliance/reporting level. |
|   5 | Phase                                   |     phase | `local` - Local development<br><br>`dev` - Development<br><br>`prod` - Production<br><br>`stage` - Staging<br><br>`qa` - QA                                                                                                                                                                     |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

Other names

SysEnv may also be written as sysenv or sysEnv in code, however it is
preferred to use the SysEnv when writing documentation.

## SysEnv naming

### Full name

SysEnvs are named by dropping any unnecessary characters, lower casing
all characters, replacing spaces with dashes, and hyphenating all
remaining components together into one string.

Care should be taken to ensure that the full name of the SysEnv is not
unnecessarily long or complex.

#### Examples

For example, a production AWS deployment of fictional product
“Mr. Potato Head” would be named `ivy-aws-mr-potato-head-prod`.

A development on-premise deployment in the Austin-1 office of fictional
product “Yodelling Coach” would be named
`ivy-aus-1-yodelling-coach-dev`.

A QA deployment in GCP us-central1 of fictional product “Visionary
Squirrel” would be named `ivy-gcp-us-central1-vsqrrl-qa`.

A production customer deployment of fictional product “Scooby Snacks” in
an unknown (codenamed) location would be named
`ivy-wanda-us-1-scooby-snacks-prod`.

Deployment providers like AWS will use the full name of the SysEnv as
the account name for login:

``` text
https://ivy-aws-mr-potato-head-prod.signin.aws.amazon.com/console/
```

#### Limitations

Full names must be chosen wisely depending on the deployment provider.
Length limitations are listed below:

|                         |                      |
|-------------------------|----------------------|
| **Provider**            | **Max. Name Length** |
| AWS Account ID Alias    | 63                   |
| Google Cloud Project ID | 30                   |
| Azure Subscription Name | 64                   |
| On-Premise              | 64                   |

### Short name (aka DC/VPC name)

SysEnvs require shorter names than their full version for the sake of
brevity if mentioned in a conversation or written on paper.

A short name should be quickly pronounceable and easy to remember.

Short names are used as the VPC names for AWS/GCP deployment providers.
This is to limit the length/repetition of resource names.

Short names are also used as a component in the DNS identifier for
services and hosts in a SysEnv such as:

``` text
task-runner-i-42bcdef6.node.helix.ivy
                            ^^^^^
```

Choose names wisely...

Care should be taken to ensure that any names used are non-trademarked
and non-overlapping.

Document it!

All short names are to be cataloged in your documentation.

#### Examples

|Full Name|Short Name|
|--- |--- |
|ivy-aws-mr-potato-head-prod|whiskey|
|ivy-wanda-us-1-scooby-snacks-prod|helix|

## Name is not machine-parseable

** SysEnv names are not designed to be parsable by string splitting. **

A SysEnv should be referred to in code by its entire name.

SysEnv names may be machine-generated from their components if each
component name is known.

# What does a SysEnv contain?

A SysEnv contains the following:

Master container

- AWS: Account
- GCP: Project
- On-Prem: No such isolation level, used for naming only

Infrastructure

All:

- Separate public and private IP space
- Separate management VPN profile
- Peering/VPN access to Tools SysEnv (artifact access)

AWS: Dedicated VPC

GCP: Dedicated VPC

On-Prem: Customer-provided IP network with dedicated IP space

- AWS: EC2
- GCP: GCE
- On-Prem: Hypervisor or Docker container capability

<!-- -->

- AWS: EBS / S3
- GCP: Block Storage / GCS
- On-Prem: Customer-provided iSCSI or NFS block storage, or object storage.

<!-- -->

- Network
- Compute
- Storage

Security

- AWS: IAM assumable roles
- GCP: Cloud IAM
- On-Prem: Customer-provided security mechanism

Deployed services

- Orchestration masters
- Worker nodes

<!-- -->

- Service orchestration layer
- Application containers
- Database nodes
- Queue nodes
- User interfaces
- Monitoring



Deployments in cloud providers will follow best practices for the above
components, however we will need to ensure that we make strategic use of
the cloud-provided components to ensure when we are faced with a
customer-provided infrastructure that we have suitable alternatives
available.

# AWS IVY SysEnv Hierarchy

ivy-controlled AWS SysEnvs will be deployed in a federated manner,
allowing for centralized IAM management via SSO.

We will utilize AWS Organizations to ensure billing and tagging
enforcement across all sub accounts.

## Master SysEnv Accounts

Master accounts are the root node on the AWS Organization tree. All
billing, IAM, and compliance rules will live here.

These accounts will be MFA enabled with a hardware token, and will
delegate access via Single Sign-On to any account in the organization
that the individual has access to.

- **`ivy-aws-master-<phase>`**Master account for commercial SysEnvs in AWS
- **`ivy-aws-gov-master-<phase>`**Master account for GovCloud SysEnvs in AWS

Master accounts will be phased (dev, prod, qa), and will provide
separate IAM credentials for all users via STS. IAM access keys will be
scoped via STS to only the SysEnv requested.

AWS credential scripts can be used to make the process seamless for
developers and administrators to acquire credentials for CLI
applications.

## Application SysEnvs

Application SysEnvs will be created any time absolute separation between
applications must be maintained or when an existing SysEnv does not
accurately describe a new application deployment.

For instance, when working on ITAR restricted projects, the application
and all data must live in a separate SysEnv in GovCloud. This ensures
that there is no leak of information via the AWS console (S3 bucket
names, IAM role names, instance IDs).

Application SysEnvs will also be created per region and peered to the
Tools SysEnv each.

## Other SysEnvs by Purpose

Each of the following SysEnvs will initially be deployed as **prod**
phases to limit the complexity of the network, and we will add
development, QA, and staging phases later once necessary.

Compliance

- Compliance enforcement lambdas (CloudCustodian/etc) \[Optional\]

Tools

- CI/CD artifact servicing (npm/maven/docker registry, OS package repo)
- Confluence / Jira

LogArchive

- CloudTrail logging
- ELB logs
- Application log backups
- Compliance logs

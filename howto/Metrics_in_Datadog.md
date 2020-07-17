# Metrics with Ivy

Datadog is not certified for handling FOUO data or metric names, if you
require statistics for FOUO-specific data you would have to find a different solution.

*Note:* Checkout our [Basic Dashboards](https://github.com/nxtlytics/ivy-terraform-modules/tree/master/datadog/basic-dashboards) terraform module

# Overview

DataDog is currently the metrics provider of choice for Ivy for a
few reasons:

- Supports many different host-level metric types (cassandra, kafka, mesos, k8s, docker...)
- Collectors use really simple and vendor agnostic protocol called StatsD

We are **not using the following features** of Datadog for cost and
information security reasons:

- Tracing
- Log management
- Container statistics

If you need Tracing or Logging, these will be handled via in-house
services like Zipkin or Elasticsearch/Logstash/Kibana

## Examples

Go to <https://app.datadoghq.com/dashboard/lists/> for a selection of Infrastructure-managed dashboards

`Insert screenshots here`

# Implementing DataDog

DataDog delivers an Agent binary that is running on all the Mesos agents in the Ivy SysEnvs. This Agent has a collector interface listening on `UDP:0.0.0.0:8125`.

The collector implements the StatsD protocol and provides a few additional features not present in the standard StatsD implementation - namely tag support.

Applications should simply emit UDP StatsD messages to the host defined as in the environment variable `STATSD_HOST` (in the form of `host:port`).

## Pitfalls

Before getting into the meat of implementing Datadog, make sure you
understand these pitfalls that we've seen cause issues with DataDog and
many StatsD based products in general:

-   Avoid sending too many tags per metric (high cardinality tags), DataDog charges per metric+tag permutation
-   Avoid creating many metrics - Datadog charges per 100 custom metrics
-   Avoid sending too many metrics too frequently. Use histograms where possible. Datadog will aggregate your histogram into count, min, max, average, and p99, p95, p90 statistics.
-   ...add any additional pitfalls here...

## Prefixes

All Ivy metrics should start with the prefix `ivy.app_name`.

This is to prevent metrics conflicts with 3rd party applications and make our lives easier when/if we move to a 3rd party hosted environment.

## Tags

DataDog will add a set of common tags to your metrics automatically by default:

|                       Tag | Value                                                                                |
| ------------------------- | ------------------------------------------------------------------------------------ |
|            **`hostname`** | `mesos-agent-i-{random}.node.sysenv.ivy`                                             |
|         **`ivy_service`** | (your app should override this, if you do not it will default to `mesos_agent`)      |
|          **`ivy_sysenv`** | `ivy-aws-dev`                                                                        |
|          **`ivy_region`** | `us-west-2`                                                                          |
|     **`ivy_environment`** | `dev`                                                                                |
|             **`aws_...`** | Common AWS tags such as the Cloudformation stack, billing account name, etc.         |
|    **`docker_image:...`** | `nxtlytics/mr-potato-head:ci.12.abcdef`                                              |
|   **`marathon_task:...`** | `/backend/mr-potato-head`                                                            |

However, your application should emit these **application-specific** common tags:

|           Tag | Purpose                                                                                                                                                                                            |
|           --- | ---                                                                                                                                                                                                |
| `ivy_service` | *Name of your application*</br>This allows you to ensure that in the case a marathon task name does not match the service that you can untangle the mismatched stats.                              |
|   `ivy_phase` | *Phase of your application*</br>This allows untangling of stats emitted from a `staging` version of an application that exists in a `dev` sysenv.</br>Example: production, staging, or development |

## Emitting Metrics

### Dockerized Applications

It is the application's responsibility to emit metrics to the DataDog
collector agent running on the Docker host.

In most cases, the common settings are as follows:

-   **`STATSD_HOST`** → `172.17.0.1:8125` (universal docker0 bridge IP address the container has access to in bridged or host networking mode), AMI Bakery configures this [here](https://github.com/nxtlytics/ivy-ami-bakery/blob/master/roles/docker/tasks/Amazon.yml#L15).
-   `PHASE` → production, staging, or development

Your application should emit metrics over UDP to this IP address and port combination.  
Your application should NOT rely on a collector or sidecar interrogating your application to have metrics populate.  
Your application should NOT include a DataDog collector instance inside your container.

### Host-based Applications

In the rare case that your application has a dedicated host assigned to
it (cassandra, kafka, mesos masters, postgres instances), you will need
to either emit metrics via StatsD, or you can use the direct DataDog
integrations to collect metrics from your application.

In most cases the common settings are as follows:

-   **`STATSD_HOST`** → `127.0.0.1:8125` (localhost for the machine the application is on)

### Managed services

For situations when we are not able to run a datadog-agent in a host,
like with a managed service, and datadog supports pulling metrics from
this service we created a datadog-agent container that we can run in a
container scheduler. 

Example: A PostgreSQL host in RDS, we create a `datadog`  role and give
it access to `pg_stats`  in the `postgres`  DB, passing this
configuration via environment variables to a datadog-agent container
with network access to said managed service.

## Environment variable patterns

| Environment Variable Type | Description                                                                                               |
|---------------------------|-----------------------------------------------------------------------------------------------------------|
| toggle metrics            | Should accept strings `all` , `none`  and array of metrics Example: `['ivy.<service>.<metrics>(.type)']`  |

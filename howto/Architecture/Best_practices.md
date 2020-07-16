# App Best Practices

This is a living document, everything stated here can change, check
history

# Audience

-   Developers (and teams of developers) of codebases that will rely
    on continuous integration and continuous deployments.

# The meat of it

Your codebase will only be deployed as a container into any environment,
therefore you should work towards your codebase resulting into a
container image keeping in mind the following:

-   Separate your build and run stages.
    -   The build stage is a transform which converts a codebase into an
        executable bundle known as an artifact (container image in this
        case). Using a version of the code at a commit specified by the
        deployment process, the build stage fetches vendors dependencies
        and compiles binaries and assets.
    -   The release stage takes the artifact produced by the build stage
        and combines it with the deploy’s current config. The resulting
        release contains both the artifact and the config and is ready
        for immediate execution in the execution environment.
    -   The run stage (also known as “runtime”) runs the container in
        the execution environment, by launching some set of the app’s
        processes against a selected release.
-   If possible, build and release stages should be executed in a
    container. Example:
    [ivy-circleci-orb](https://github.com/nxtlytics/ivy-circleci-orb).
-   Containers only have 1 IP address, having more than 1 IP address is
    a special case under most container schedulers
-   Configuration should be obtained via environment variables
    -   If an API for secrets is available use it.
-   Containers should only expose 1 port for inter-service communication
-   Logs should be written to standard output and standard error
    -   If an API for logs is available use it.
-   Backing services and runtime dependencies (example: database) should
    be declared and tracked, preferably within your codebase 
    -   Keep in mind: "a chain is only as strong as the weakest of its
        links"
-   During runtime, state should persist in a backing service (example:
    database). The memory space or filesystem of the process can be used
    as a brief, single-transaction cache.
-   Scaling out should be preferred over scaling up.
-   Processes should shut down gracefully when they receive a
    `SIGTERM`  signal.
-   fully qualified domain name (FQDN) should generally be used, but in
    certain circumstances an IP might be given as a fqdn and the
    codebase should support this.
-   Use HTTP 1.1
    -   Keep in mind [HTTP 1.1 can upgrade to websockets](https://developer.mozilla.org/en-US/docs/Web/HTTP/Protocol_upgrade_mechanism)
-   Use
    [saplings](https://github.com/nxtlytics?q=sapling&type=&language=)
    -   If a sapling is not available for your programming language
        start one.
-   Moving away from any of these patterns should be documented.
-   Patterns to avoid:
    -   [Host-networking](https://docs.docker.com/network/host/)

# Currently supported backing services

-   RDS PostgreSQL
-   S3
-   Chronos
-   Cassandra
-   Kafka
-   IAM Roles

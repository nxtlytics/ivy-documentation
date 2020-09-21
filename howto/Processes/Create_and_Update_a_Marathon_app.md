# Create (and Update) a Marathon App

## Software version

- Marathon: 1.9.109
- Mesos: 1.9.0

## Requirements

- Connect to the environment's VPN
- A version of the app container has been created
- A version of the app container can be pulled by any of the mesos agents

## Create a new Marathon App

1. On your browser fo to http://master.mesos.service.<environment>.<namespace>:8080
2. Click `Create Application`
3. Click `JSON Mode`
4. Delete the content and replace it with the one below (**DO NOT** click `Create Application` yet):

```json
{
  "id": "/<namespace>/<app name>",
  "role": "*",
  "cmd": null,
  "cpus": 0.1,
  "mem": 128,
  "disk": 0,
  "gpus": 0,
  "instances": 1,
  "acceptedResourceRoles": [
    "*"
  ],
  "container": {
    "type": "DOCKER",
    "docker": {
      "forcePullImage": false,
      "image": "<Image to use>",
      "parameters": [],
      "privileged": false
    },
    "volumes": [],
    "portMappings": [
      {
        "containerPort": 8080,
        "hostPort": 0,
        "labels": {},
        "name": "webserver",
        "protocol": "tcp",
        "servicePort": 10010
      }
    ]
  },
  "env": {
    "ENVIRONMENT_VARIABLE_01": "VALUE01",
    "ENVIRONMENT_VARIABLE_02": "VALUE02"
  },
  "networks": [
    {
      "mode": "container/bridge"
    }
  ],
  "portDefinitions": [],
  "maxLaunchDelaySeconds": 300,
  "upgradeStrategy": {
    "maximumOverCapacity": 1,
    "minimumHealthCapacity": 0
  }
}
```
5. Important Environment variables
- `SERVICE_NAME`
  - This variable tells registrator what name to give consul for these containers
  - This also makes `<Service Name>.service.<sysenv shortname>.<ivy tag>:8080` possible
- For consul health checks see [here](https://github.com/gliderlabs/registrator/blob/v6/docs/user/backends.md#consul-http-check).
  - Examples:

```shell
SERVICE_80_CHECK_HTTP=/health/endpoint/path
SERVICE_80_CHECK_INTERVAL=15s
SERVICE_80_CHECK_TIMEOUT=1s        # optional, Consul default used otherwise
SERVICE_CHECK_SCRIPT=nc $SERVICE_IP $SERVICE_PORT | grep OK
SERVICE_CHECK_TTL=30s
```

6. Replace the values as necessary for your application, if you want more details fo [here](https://mesosphere.github.io/marathon/1.9/docs/application-basics.html)

7. Click `Create Application`

8. If you want other URLs to find its way to this application you need to do the following:
9. Go to `http://master.mesos.service.<sysenv shortname>.<ivy tag>:8500/ui/<sysenv shortname>/kv/service/`
10. Click `Create`
  - `Key or Folder`
    - `<Service Name>/public` for `external.example.com`
    - and/or
    - `<Service Name>/private` for `internal.example.com`
    - For multiple records you can use
      - `public.0` and/or `private.0`
      - `public.1` and/or `private.1`
  - `Value`
    - `something.example.com`

## Related Resources

- [Infrastructure Troubleshooting](./Infrastructure_Troubleshooting.md)

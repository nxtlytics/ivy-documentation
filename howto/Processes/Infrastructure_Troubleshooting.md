# Infrastructure Troubleshooting

## Consul services and Marathon replicas do not match

Note: Normally this happens when the consul template is out of sync with the marathon services.
      Sometimes only a couple of nodes are out of sync too

### To check all the template is in sync run the command below from one of the mesos agents

```shell
$ consul exec -service=mesos -tag=agent md5sum /etc/haproxy/haproxy.cfg | grep '/etc/haproxy/haproxy.cfg' | awk '{ print $2 }' | sort -u
```

The output should be only 1 line, if multiple lines you have some of the nodes are out of sync

### How to force sync?

```shell
$ rm -rf /opt/consul/data/services/* && consul reload && docker restart registrator
Configuration reload triggered
registrator
```

## Repro for DNS peaks filebeat issue

You need 2 machines swarm-dev-1 (manager) and swarm-dev-2 on the same network, running docker in swarm mode

Make sure the host has this little script, used for waiting services:
```sh
echo '#!/bin/sh
until curl -sf "$1" 1>/dev/null; do sleep 1; done' | sudo tee /usr/local/bin/wait-for
sudo chmod +x /usr/local/bin/wait-for
```

Create filebeat-bug.yml config, it has to be available on each host

```sh
docker config rm filebeat-bug.yml
docker config create filebeat-bug.yml filebeat.yml
```

Start the services
```sh
docker stack up x -c infra.yml
```

Monitor DNS on host swarm-dev-1
```sh
sudo tcpdump -i eth0 udp port 53
```

Wait a few minutes ..

Remove elasticsearch service when the services are ready
```sh
docker service rm x_elasticsearch
```

Then DNS reqs start peaking at [0:20][1], one DNS req every millisecond or more

Stop filebeat and see the traffic calm down, from [0:40][1]

```sh
docker service rm x_filebeat
```

Stopping other services completely eliminate DNS reqs traffic

[1]: https://streamable.com/xexfh

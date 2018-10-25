## Repro for DNS peaks filebeat issue

You just need 1 machine running docker in swarm mode (`docker swarm init`)

Make sure it has this little script, used for waiting services:
```sh
echo '#!/bin/sh
until curl -sf "$1" 1>/dev/null; do sleep 1; done' | sudo tee /usr/local/bin/wait-for
sudo chmod +x /usr/local/bin/wait-for
```

Start the services
```sh
docker stack up x -c infra.yml
```

Monitor DNS on host swarm-dev-1
```sh
sudo tcpdump -i eth0 udp port 53
```

Wait a few minutes .. and remove elasticsearch service
```sh
docker service rm x_elasticsearch
```

Then DNS reqs start peaking at [0:28][1], one DNS req every millisecond or more

Stop filebeat and see the traffic calm down, from [0:58][1]

```sh
docker service rm x_filebeat
```

Stopping other services completely eliminate DNS reqs traffic

[1]: https://streamable.com/xu5xy

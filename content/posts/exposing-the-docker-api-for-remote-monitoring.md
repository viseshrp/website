---
title: "Exposing the Docker API for Remote Monitoring"
date: 2023-09-24T11:37:28-04:00
draft: false
---

If you want to monitor your docker containers, there are excellent tools like [Dozzle](https://dozzle.dev/)
and [Uptime Kuma](https://uptime.kuma.pet/). Focusing on the former for now, this requires explicit configuration of
the Docker daemon on the target host.

For monitoring containers on a local host (the same host where Dozzle/Kuma are already running), one would
just bind the `docker.sock` file to the monitoring container.

Using Docker CLI:
```shell
-v /var/run/docker.sock:/var/run/docker.sock
```

Using Docker Compose:
```yaml
volumes:
   - /var/run/docker.sock:/var/run/docker.sock
```

Monitoring containers running on remote hosts is different and requires enabling and exposing the TCP port
`2375` on the target host. Following are the steps I followed to expose the Docker API on a Raspberry Pi 4B
running the official Debian-based OS.

Edit the file located at `/etc/docker/daemon.json`. If not present, create one:
```json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://192.168.68.78:2375"]
}
```
This exposes the Docker API `/var/run/docker.sock` of the host at port 2375.
The IP must exactly match what the monitoring tool will use to connect with. Also, note that this method is
insecure and gives 'root' access to your containers, so only use within a closed network.

Now:
```shell
$ systemctl edit docker.service
```
You will see an override file for `docker.service`. Add the below in the uncommented space:
```shell
### Editing /etc/systemd/system/docker.service.d/override.conf
### Anything between here and the comment below will become the new contents of the file

[Service]
ExecStart=
ExecStart=/usr/bin/dockerd

...
```
This removes additional duplicate options that were added in the previous config file.

Restart the service.
```shell
# reload systemctl configs
$ sudo systemctl daemon-reload
# restart docker
$ sudo systemctl restart docker.service
```

Verify now:
```shell
# check using netstat
$ sudo netstat -lntp | grep dockerd
tcp        0      0 192.168.68.78:2375      0.0.0.0:*               LISTEN      1955/dockerd
```

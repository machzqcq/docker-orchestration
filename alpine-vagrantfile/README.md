# Vagrantfile using Alpine Linux
First install vagrant plugin to support Alpine linux `vagrant plugin install vagrant-alpine`. It is a ruby gem FYI

DOCKER 1.12 ON ALPINE IS NOT FULLY FUNCTIONAL! WILL UPDATE WHEN SUCCEEDED!


# Start environment
`vagrant up` This should create 3 alpine linux vm's

# Common issues

If you see the below error when doing vagrant up, then `rm ~/.vagrant.d/tmp/*`

```
==> smgr: Box download is resuming from prior download progress
An error occurred while downloading the remote file. The error
message, if any, is reproduced below. Please fix this error and try
again.

HTTP server doesn't seem to support byte ranges. Cannot resume.

```   

### Disable shared folders

`config.vm.synced_folder '.', '/vagrant', disabled: true` See Vagrantfile content

# Install docker on alpine

Docker installation on alpine is expert mode as described [here](https://docs.docker.com/engine/installation/binaries/)

- `vagrant ssh smgr`
- `sudo apk upgrade`
- `sudo apk add wget curl git iptables xz procps`  

```
snode1-alpine:~$ sudo apk add wget curl git iptables xz procps
(1/9) Installing wget (1.16.1-r0)
(2/9) Installing expat (2.1.0-r1)
(3/9) Installing pcre (8.36-r2)
ERROR: pcre-8.36-r2: No error information
(4/9) Installing git (2.2.1-r0)
(5/9) Installing iptables (1.4.21-r1)
(6/9) Installing xz-libs (5.0.7-r0)
(7/9) Installing xz (5.0.7-r0)
(8/9) Installing libproc (3.3.9-r0)
(9/9) Installing procps (3.3.9-r0)
Executing busybox-1.22.1-r15.trigger
OK: 239 MiB in 55 packages
```   

- `snode1-alpine:~$ sudo /bin/bash cgroupfs-mount.sh` See the script in the git repo
- `snode1-alpine:~$ wget https://experimental.docker.com/builds/Linux/x86_64/docker-1.12.0-rc4.tgz`
- `sudo addgroup docker`
- `sudo addgroup $(whoami) docker`
- `sudo killall dockerd`
- `tar -xvzf docker-1.12.0-rc4.tgz`
- `sudo mv docker/* /usr/bin/`
- `sudo dockerd &`
- Logout and login
- `docker info`

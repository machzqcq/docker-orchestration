# Vagrantfile using Alpine Linux
First install vagrant plugin to support Alpine linux `vagrant plugin install vagrant-alpine`. It is a ruby gem FYI

DOCKER 1.12 ON ALPINE IS NOT FULLY FUNCTIONAL! WILL UPDATE WHEN SUCCEEDED!

```
snode1-alpine:~$ docker run -it hello-world
ERRO[0098] Handler for POST /v1.24/containers/create returned error: No such image: hello-world:latest
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c04b14da8d14: Pull complete
Digest: sha256:0256e8a36e2070f7bf2d0b0763dbabdd67798512411de4cdcf9431a1feb60fd9
Status: Downloaded newer image for hello-world:latest
ERRO[0100] containerd: start container                   error=oci runtime error: rootfs_linux.go:53: mounting "/dev/mqueue" to rootfs "/var/lib/docker/vfs/dir/a56bc92ecfca901761aefd2b8e850eac0522a746c2fa0b48f71aa98b95cb07f0" caused "no such device" id=dcfe82a11387df432cb0b3904ab85f2e28ae26525c8eb17f3e146d279bc2ff59
                                                                             ERRO[0100] Create container failed with error: oci runtime error: rootfs_linux.go:53: mounting "/dev/mqueue" to rootfs "/var/lib/docker/vfs/dir/a56bc92ecfca901761aefd2b8e850eac0522a746c2fa0b48f71aa98b95cb07f0" caused "no such device"
                                                                       ERRO[0101] Handler for POST /v1.24/containers/dcfe82a11387df432cb0b3904ab85f2e28ae26525c8eb17f3e146d279bc2ff59/start returned error: oci runtime error: rootfs_linux.go:53: mounting "/dev/mqueue" to rootfs "/var/lib/docker/vfs/dir/a56bc92ecfca901761aefd2b8e850eac0522a746c2fa0b48f71aa98b95cb07f0" caused "no such device"

```


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

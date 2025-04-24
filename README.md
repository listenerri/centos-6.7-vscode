# centos-6.7-vscode

Build a centos 6.7 docker images with upgraded toolchain in order to support vscode remote development.

Advice from Microsoft: `https://code.visualstudio.com/docs/remote/faq#_can-i-run-vs-code-server-on-older-linux-distributions`

This method should not only work with centos 6.7, but also work with ubuntu 14.04, 18.04, etc., although I didn't test it.

## Build toolchain

NOTE: There is an prebuilt toolchain pkg be placed at `./files/toolchain-x86_64-gcc-8.5.0-glibc-2.28.tar.xz`, just start to build centos 6.7 docker images if you do not want build toolchains again, or follow this section to generate yourself toolchain pkg.

1. Build toolchain using crosstool-ng

```
docker build -t toolchain-builder -f Dockerfile.build-toolchain .
```

2. Copy built toolchain files from container to local host

```
docker run --rm -it -d --name mytoolchain toolchain-builder bash
docker exec -it mytoolchain bash -c 'cd /home/ubuntu && tar Jcf toolchain-x86_64-gcc-8.5.0-glibc-2.28.tar.xz toolchain-x86_64-gcc-8.5.0-glibc-2.28/x86_64-linux-gnu'
docker cp mytoolchain:/home/ubuntu/toolchain-x86_64-gcc-8.5.0-glibc-2.28.tar.xz ./files/
docker stop mytoolchain
```

The toolchain file has been saved at ./files/toolchain-x86_64-gcc-8.5.0-glibc-2.28.tar.xz on host.

3. Delete toolchain-builder image (optional)

```
docker rmi toolchain-builder
```

## Build centos 6.7 using new toolchain

```
docker build -t centos-6.7-vscode -f Dockerfile.centos-6.7-vscode .
```

## Remote development test

1. Run a container from centos-6.7-vscode image

```
docker run --name centos-6.7-vscode-test -it -d centos-6.7-vscode bash
```

2. Ensure container is running

```
docker ps -a
```

3. Test vscode dev container

Start vscode and open `Remote Explore` from left side bar, switch drop down menu to `Dev Containers`, right click on `centos-6.7-vscode-test` container and click `Attach in Current Window`


## Known issues

### fetch Error: getaddrinfo ENOTFOUND

patchelf caused an exception in the node program of vscode server and failed to execute dns

First, create the ignition config file from the butane file:

```
butane coreos.bu -d . -o /tmp/coreos.ign
```

Now deploy a coreos virtual machine using that ignition file for configuration:

```
virt-install -n coreos.virt -r 4096 -w network=default \
  --disk pool=default,size=10,backing_store=fedora-coreos-41.20241109.1.0-qemu.x86_64.qcow2,backing_format=qcow2 \
  --graphics none \
  --import \
  --os-variant fedora-coreos-next \
  --qemu-commandline="-fw_cfg name=opt/com.coreos/config,file=/tmp/coreos.ign"
```

Once the machine is up there will be a web service running on port `2040`:

```
core@localhost:~$ curl localhost:2040
Hostname: ee54368a09ee
IP: 127.0.0.1
IP: ::1
IP: 10.88.0.2
IP: fe80::246d:a9ff:fef9:27cf
RemoteAddr: 10.88.0.1:52146
GET / HTTP/1.1
Host: localhost:2040
User-Agent: curl/8.9.1
Accept: */*
```

We can access the same service from a container:

```
core@localhost:~$ podman run --rm docker.io/curlimages/curl -sSfv host.containers.internal:2040
Trying to pull docker.io/curlimages/curl:latest...
Getting image source signatures
Copying blob 4ca545ee6d5d done   |
Copying blob 43c4264eed91 done   |
Copying blob 90d3ba855880 done   |
Copying config 53bc06bb9a done   |
Writing manifest to image destination
* Host host.containers.internal:2040 was resolved.
* IPv6: (none)
* IPv4: 10.88.0.1
*   Trying 10.88.0.1:2040...
* Connected to host.containers.internal (10.88.0.1) port 2040
* using HTTP/1.x
> GET / HTTP/1.1
> Host: host.containers.internal:2040
> User-Agent: curl/8.11.0
> Accept: */*
>
* Request completely sent off
Hostname: ee54368a09ee
IP: 127.0.0.1
IP: ::1
IP: 10.88.0.2
IP: fe80::246d:a9ff:fef9:27cf
RemoteAddr: 10.88.0.1:56132
GET / HTTP/1.1
Host: host.containers.internal:2040
User-Agent: curl/8.11.0
Accept: */*

< HTTP/1.1 200 OK
< Date: Sat, 23 Nov 2024 02:07:33 GMT
< Content-Length: 210
< Content-Type: text/plain; charset=utf-8
<
{ [210 bytes data]
* Connection #0 to host host.containers.internal left intact
```

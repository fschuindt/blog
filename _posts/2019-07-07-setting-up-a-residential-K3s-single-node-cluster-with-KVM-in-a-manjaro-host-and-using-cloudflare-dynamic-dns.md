---
layout: post
title: Setting up a residential K3s single node cluster with KVM in a Manjaro host and using Cloudflare dynamic DNS
categories: IT
image: images/2019-07-07-setting-up-a-residential-K3s-single-node-cluster-with-KVM-in-a-manjaro-host-and-using-cloudflare-dynamic-dns/press.jpg
excerpt: "A guide on how to setup K3s Kubernetes dristro cluster on a local network of virtual computers with KVM."
---

![press]({{ site.baseurl }}/images/2019-07-07-setting-up-a-residential-K3s-single-node-cluster-with-KVM-in-a-manjaro-host-and-using-cloudflare-dynamic-dns/press.jpg)

For long I've been playing with Kubernetes in production environments, at my job and in other projects. But everything was done using managed solutions like the one from Digital Ocean, which is great by the way. But I was willing to put a bit of my hands on it.

I happen to have a desktop computer (i5 8400 16GB RAM running Manjaro) which I'm not using that much, so I planned to setup a VM on it and run a single node Kubernetes cluster myself. But wait a second, for that we had [Minikube](https://github.com/kubernetes/minikube), right? Yes, the problem being it's designed to work inside your own computer and not to have contact with the external world, not even your local network. I was having trouble to make that work when I remembered about [K3s](https://k3s.io/), which is a Kubernetes distribution that's actually simpler than Minikube, it's great and it's also production ready.

One of the key things here is that I wanted to take advantage of Intel's [Vt-d](https://en.wikipedia.org/wiki/X86_virtualization#Intel-VT-x) and [KVM](https://www.linux-kvm.org/page/Main_Page) for virtualization. Also I want this VM to get its own IP address on my local network, so to my router it would appear as another physical machine. For that I'm going to create a bridge network interface between the host and the VM.

Before we start, mind that all the work here will be made in my Manjaro laptop, named `skywitch` a.k.a. "laptop", connecting via SSH to the Manjaro bare metal desktop server, `redwitch` a.k.a "the host" (The SSH setup was already in place). I want to be able to connect to the K3s from `skywitch` and also to access its running services from the outside world. Which will need some port-forwarding at the internet router firmware. (K3s already comes with [Traefik](https://traefik.io/) as an Ingress resource)

For the VM OS I'm going to choose Ubuntu Server 18.04 and its hostname will be named `warlock`.

All the servers, `redwitch` and `warlock` will have static IPv4 address on the local network. All other devices, including my laptop will be using DHCP.

Here's how it's going to look like after finished:  
![the big picture]({{ site.baseurl }}/images/2019-07-07-setting-up-a-residential-K3s-single-node-cluster-with-KVM-in-a-manjaro-host-and-using-cloudflare-dynamic-dns/the_big_picture.png)

*You can ignore the HDMI KVM Switch device for the purpose of this guide.*

And just to make things clear, here's the `/etc/hosts` I've setup to my `skywitch` laptop:
```
127.0.0.1	localhost
127.0.1.1	skywitch
192.168.1.151	redwitch
192.168.1.161	warlock

::1	localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Another important thing is that here in Brazil is common for residential connections to lack a static external IP address, but to have a dynamic one that changes each restart. In my case my address changes depending on the route to the server, so I can be talking to Spotify as one address and to YouTube as other, you never know, but they all point to me. Plus the ports 80 and 443 are blocked to the external world, so people can not serve web pages without having to explicitly use a port like `:8080` in the URL.

And that means two things:

1. I'm going to need some sort of [Dynamic DNS](https://en.wikipedia.org/wiki/Dynamic_DNS) tool (remember [no-ip](https://www.noip.com/)?), in this case [Cloudflare](https://www.cloudflare.com/) will be the choice, as it provides this service for private domain names with no cost.

2. The router port-forwarding step must account for having the ports 80 and 443 blocked.

I'm going to assume you already have a server with a KVM setup and Vt-d enabled. So I'm not going to cover this process here, this can be easily found on Google. But if you're wondering, [here](https://www.fosslinux.com/2484/how-to-install-virtual-machine-manager-kvm-in-manjaro-and-arch-linux.htm) is one of those guides.

So the plan is:
- Setup a bridge network interface on the host.
- Create a Ubuntu 18.04 VM on the host with the bridged network.
- Set up a K3s single node Kubernetes cluster into the VM.
- Describing a Dynamic DNS [`cronJob`](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) into the cluster to update a domain name in Cloudflare with the external IP address.
- Configuring the router to forward certain ports to the VM.
- Deploying a simple application to the cluster and have it exposed to the world.

So let's start.

*Just a quick note: Many of the stuff presented here was taken from other guides on the web, which you can find all listed at the end of this post.*

## Setup a bridge network interface on the host

This will allow us to create a VM that will receive its own IP on our local network, pretty much as every other physical device does.

First install `bridge-utils`, it will come in handy later:
```
$ sudo pacman -S bridge-utils
```

Now you need to find out the name of your main network interface, which can be done with `$ ip link show` or `$ ifconfig`. In my case it was named `enp0s31f6`.

You're also going to need to know the connection gateway and DNS addresses.

To get the gateway address you can just:

```
$ route -n | grep "^0.0.0.0" | tr -s " " | cut -f2 -d" "

192.168.1.1
```

And to know the DNS:
```
$ cat /etc/resolv.conf

# Generated by NetworkManager
search GREATEK
nameserver 192.168.1.1
```

In my case they're the same, but it may differ from vendor to vendor, I'm not sure.

You should also pick up a static IP address for the host machine, I'm going to choose `192.168.1.151`. Plus I'm going to name the bridge interface as `br1`.

Now let's create a `/etc/netctl/kvm-bridge` file:

```
$ sudo vi /etc/netctl/kvm-bridge
```

With the following content:
```
Description="Bridge Interface br10 : enp0s31f6"
Interface=br1
Connection=bridge
BindsToInterfaces=(enp0s31f6)
IP=static
Address='192.168.1.151/24'
Gateway='192.168.1.1'
DNS='192.168.1.1'
MACAddressOf=enp0s31f6
SkipForwardingDelay=yes
set IP=no
```

Replace the values with the ones you want, then:

```
$ sudo systemctl restart NetworkManager.service
```

Then start and enable the `kvm-bridge`:
```
$ sudo netctl start kvm-bridge
```

```
$ sudo netctl enable kvm-bridge
```

And that's it. You can use `$ brctl show` and `$ bridge link` to check bridges and see bridged interfaces respectively. But for now I think we're good. Our host now have a new source for its static IP (`192.168.1.151`) and a bridge network interface on it. Great.

## Create a Ubuntu 18.04 VM on the host with the bridged network

I won't get much into the `libvirt` usage, if you want here's the `virsh` CLI reference page:  
[https://libvirt.org/sources/virshcmdref/html/](https://libvirt.org/sources/virshcmdref/html/)

And the basics:
```
Boot a VM - virsh start <vm>
Stop a VM - virsh shutdown <vm>
Suspend a VM - virsh suspend <vm>
Delete a VM - virsh destroy <vm> and virsh undefine <vm>
```

We're going to use a Shell script in order to create and configure the VM. So create a `create_vm.sh` file:

```
$ vi ~/create_vm.sh
```

With:
```bash
#!/bin/sh

if [ -z "$1" ] ;
then
 echo Specify a virtual-machine name.
 exit 1
fi

sudo virt-install \
     --name $1 \
     --ram 4096 \
     --disk path=/home/fschuindt/hdd_repo/libvirt/images/$1.img,size=30 \
     --vcpus 4 \
     --os-type linux \
     --os-variant ubuntu18.04 \
     --network bridge:br1,model=virtio \
     --graphics none \
     --console pty,target_type=serial \
     --location 'http://archive.ubuntu.com/ubuntu/dists/bionic-updates/main/installer-amd64/' \
     --extra-args 'console=ttyS0,115200n8 serial'
```

Here you shall stop and perform some editing in the file. A few things to look after are the RAM size, the number of CPUs and the `--disk path`, which mine is pointing to `/home/fschuindt/hdd_repo/libvirt/images/` with a 30GB sized disk.

This directory on my host is on a 2TB HDD mount, it's important you have it mapped to a physical device or to a place where you have enough space to install the VM. I just allocated 30GB, but that's arbitrary. Just mind that this space will be actually occupied during the VM creation.

Also the `--network bridge:br1,model=virtio` part shall point to the `br1` bridge interface we've created earlier in the guide.

Save and:
```
$ chmod +x create_vm.sh

$ ./create_vm.sh warlock
```

Where `warlock` is the name of the VM.

Now the S.O. installation will begin, the Ubuntu setup is pretty straight forward so you must complete it with no problem. Just mind tho, at the "**Software selection**" phase to pick up the OpenSSH server.

After finished the installation, it will reboot and end the process.

Let's connect via SSH to the freshly installed VM, for that, find the VM IP using:
```
sudo nmap -sP 192.168.1.0/24
```

It will have the port 22 (OpenSSH) opened. In my case it was `192.168.1.12`.

So, from my laptop:
```
$ ssh fschuindt@192.168.1.12
```

Edit with your username (`fschuindt` for me) and it will ask you for the password you've set during installation. A good thing to do now is to set a static IP to the VM.

Edit the file `/etc/netplan/01-netcfg.yml` with:
```yaml
# warlock VM

# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
     dhcp4: no
     addresses: [192.168.1.161/24]
     gateway4: 192.168.1.1
     nameservers:
       addresses: [1.1.1.1,1.0.0.1]
```

I'm using my local values for gateway, I'm setting Cloudflare DNS as my DNS servers (`1.1.1.1` and `1.0.0.1`) and the VM local static IP to `192.168.1.161`.

To apply just:
```
$ sudo netplan apply
```

You will get disconnected from the SSH session, but that's ok, just connect again with the new IP. Better yet, add it to your `/etc/hosts` file with your VM name, as I showed in the beginning.

Another important thing to do is to upload your SSH public key to the VM and to disable password logins. I won't be covering it here, for that just go to:  
[https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1804](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1804)

Now we have a virtual Ubuntu 18.04 running with a static IP on our local network. Let's do some work in it.

## Set up a K3s single node Kubernetes cluster into the VM

There's nothing much to say in this section really, K3s installation and usage is really simple, I recommend checking [their homepage](https://k3s.io/) and the [docs](https://github.com/rancher/k3s/blob/master/README.md) for a quick start. But here's the basic:

**This is inside the VM.**

To install:
```
$ curl -sfL https://get.k3s.io | sh -
```

This will also configure systemd, so K3s will start after every reboot.

To get the admin `.yaml` file:
```
$ sudo cat /etc/rancher/k3s/k3s.yaml
```

I'm going to save this on my laptop's `~/.kube/`. Saving it named as `config` connects your `kubectl` with the new cluster.

Now our single node K3s cluster is up and running, you can already start playing with `kubectl`. The next steps are really for exposing it to the world, so if you don't want that you can declare work done =]. Otherwise we still have some more stuff to do.

## Describing a Dynamic DNS `cronJob` into the cluster to update a domain name in Cloudflare with the external IP address

So I plan to expose services running on the K3s cluster to the outside world. If I had a static external IP address that would be great, but as you may already know it's not the case. But it's still possible to have a domain name pointed to the server as a Dynamic DNS (DDNS) using one DDNS provider like [no-ip.com](https://www.noip.com/) and [DynDNS](https://dyn.com/). That requires a DDNS client running on my system checking changes on the external IP and updating it against the provider.

The provider will be Cloudflare, as it's not only a DNS provider but also [supports DDNS](https://support.cloudflare.com/hc/en-us/articles/360020524512-Manage-dynamic-IPs-in-Cloudflare-DNS-programmatically). I own the `722.network` domain name on Cloudflare and I'm going to use the subdomain `fschuindt.722.network` to point to the cluster.

The client will be [ddclient](https://github.com/ddclient/ddclient), a well known DDNS client written in Perl. I've setup a [ddclient public Docker image](https://cloud.docker.com/repository/docker/zfschuindt/ddclient) that you can configure and use for that same matter. For using it I'm going to set up a Kubernetes `cronJob` that every 5 minutes will spawn a container using that image and run a DDNS check/update command, then exits and waits to the next execution, and so on.

It's not perfect but it's enough, that shall keep the subdomain name updated.

So if you will, create a `ddclient-job.yml` file:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ddclient-config-map
  labels:
    owner: ddclient
data:
  LOGIN: "your_cloudflare@login.com"
  PASSWORD: "your-cloudflare-global-api-key"
  ZONE_DOMAIN: "your-domain.com"
  ZONE_HOSTNAME_1: "your-host.your-domain.com"
  ZONE_HOSTNAME_2: "other-host.your-domain.com"
  ZONE_HOSTNAME_3: ""
  ZONE_HOSTNAME_4: ""
  ZONE_HOSTNAME_5: ""
  ZONE_HOSTNAME_6: ""
  ZONE_HOSTNAME_7: ""
  ZONE_HOSTNAME_8: ""
  ZONE_HOSTNAME_9: ""
  ZONE_HOSTNAME_10: ""
---
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: ddclient-job
  labels:
    owner: ddclient
spec:
  concurrencyPolicy: Forbid
  failedJobsHistoryLimit: 5
  successfulJobsHistoryLimit: 5
  startingDeadlineSeconds: 60
  # At every 5th minute.
  schedule: "*/5 * * * *"
  jobTemplate:
    metadata:
      name: ddclient-job
      labels:
        owner: ddclient
    spec:
      activeDeadlineSeconds: 240
      backoffLimit: 3
      template:
        metadata:
          name: ddclient-job-pod
          labels:
            owner: ddclient
        spec:
          containers:
          - name: ddclient-job-container
            image: zfschuindt/ddclient:latest
            command: ["bash", "-c", "/ddclient/entrypoint.sh"]
            envFrom:
              - configMapRef:
                  name: ddclient-config-map
          restartPolicy: OnFailure
```

And edit it to fit your needs. Especially the `ConfigMap` section, where you want to provide your Cloudflare credentials with the domain/subdomain names. I've created space for up to 10 subdomains, but if you want more you can easily edit the `entrypoint.sh` file of the image.

Here's the ddclient image GitHub repository if you want:  
[https://github.com/fschuindt/ddclient](https://github.com/fschuindt/ddclient)

Now we can just spawn the job into K3s:
```
$ kubectl apply -f ddclient-job.yml
```

Now someone is working to keep my external IP updated on the `fschuindt.722.network` domain name. Wonderful!

## Configuring the router to forward certain ports to the VM

This part changes for everyone. It's really dependent on which internet router vendor/model you have, but in general the concept is the same: Let the router to know which static IP address on the local network is to forward incoming connections on given ports.

If skipped, the outside world won't be able to connect to the services on the cluster, as the router won't know what to do with those connections, it must deliver it to some device on the network, but without knowing which it drops it.

For configuring this you need to access your router firmware interface, for me it's on http://192.168.1.1/ for any wire-connected device on the network. Then you must provide credentials and look for any "port forward" option.

You can find the default user/password combination for your router as well as instructions for port forwarding (if it supports) on the PDF manual for your router model (every model has one, just check online).

For me I added two entries on the port-forwarding rules list. One forwarding every external connection on the port 7222 to the port 80 on the `192.168.1.161` (the `warlock` VM) and other forwarding every external connection on the port 7223 to the port 443 on the same server, the `192.168.1.161` (`warlock` VM).

It's looking like this:

![port-forwading]({{ site.baseurl }}/images/2019-07-07-setting-up-a-residential-K3s-single-node-cluster-with-KVM-in-a-manjaro-host-and-using-cloudflare-dynamic-dns/port_forwarding.png)

And that will let the router know to which device to send the incoming connections. One more thing to do, let's deploy a service to the K3s and test the whole thing.

## Deploying a simple application to the cluster and have it exposed to the world

**This will be done at the `skywitch` laptop, connected to cluster using `kubectl`.**

Right now we have only one DNS pointing to the cluster, which is `fschuindt.722.network`. I'm going to deploy a service to operate on this address, more precisely a [Ingress resource](https://kubernetes.io/docs/concepts/services-networking/ingress/).

The service we're going to deploy is a simple HTTP "ping/pong" echo. It serves only one route `GET /ping`, which will reply `200 OK, "pong"`. I wrote this service using Elixir and it's on GitHub here:  
[https://github.com/fschuindt/http_echo](https://github.com/fschuindt/http_echo)

It already comes with its own `Dockerfile` and its image is [publicly available at DockerHub](https://cloud.docker.com/u/zfschuindt/repository/docker/zfschuindt/http_echo). Plus if you check the `/k8s` folder on the repository you will find a group of Kubernetes resources for deploying it into Kubernetes. This will make everything easier.

The resources are:

- `config_map.yml`
- `service.yml`
- `ingress.yml`
- `deployment.yml`

**Important: The files in the repository are just examples and may differ a bit from the ones presented here.**

And we're going to create them in this order, so the ConfigMap first:

`config_map.yml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: echo-config-map
  namespace: default
  labels:
    owner: echo
data:
  MIX_ENV: "prod"
```

```
$ kubectl create -f config_map.yml
```

Then the Service:

`service.yml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: echo-service
  labels:
    app: echo
    owner: echo
spec:
  type: NodePort
  selector:
    app: echo
    tier: web
  ports:
  - port: 4080
    targetPort: 4080
```

```
$ kubectl create -f config_map.yml
```

The Ingress:

`ingress.yml`
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: echo-ingress
spec:
  rules:
  - host: echo.warlock.network
    http:
      paths:
      - backend:
          serviceName: echo-service
          servicePort: 4080
  - host: fschuindt.722.network
    http:
      paths:
      - backend:
          serviceName: echo-service
          servicePort: 4080
```

*Notice here I'm also setting the `echo.warlock.network` domain name to be used within the local network, I'm going to add this hostname on my laptop's `/etc/hosts` as well.*

```
$ kubectl apply -f ingress
```

And finally the deployment:

`deployment.yml`
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
 name: echo-deployment
 namespace: default
 labels:
    owner: echo
    app: echo
    tier: web
spec:
  revisionHistoryLimit: 5
  replicas: 1
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        owner: echo
        app: echo
        tier: web
    spec:
      terminationGracePeriodSeconds: 60
      containers:
        - name: echo-container
          image: zfschuindt/http_echo:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 4080
          args: ["bash", "-c", "./app/entrypoint.sh"]
          envFrom:
            - configMapRef:
                name: echo-config-map
          livenessProbe:
            httpGet:
              path: /ping
              port: 4080
```

```
$ kubectl create -f deployment.yml
```

And that shall make the http://fschuindt.722.network:72222/ping available and serving the HTTP echo service to the world. :)

It may be already offline by the time you're reading this, but believe me, it worked.

And that's how my Kubernetes Dashboard looks like:

![kubernetes dashboard]({{ site.baseurl }}/images/2019-07-07-setting-up-a-residential-K3s-single-node-cluster-with-KVM-in-a-manjaro-host-and-using-cloudflare-dynamic-dns/k3s-dashboard.png)

By the way, if you want to install this dashboard on your cluster, check how to do it [here](https://github.com/kubernetes/dashboard).

And with that we shall have our residential K3s cluster running and serving to the outside world on top of a fast and optimized virtualization method. I hope this guide have served you well, setting up this environment brought me new ideas for residential server setups and was a lot of fun!

Thank you for staying with me.

See you soon. :)

## References

- [https://computingforgeeks.com/how-to-create-and-use-network-bridge-on-arch-linux-and-manjaro/](https://computingforgeeks.com/how-to-create-and-use-network-bridge-on-arch-linux-and-manjaro/)
- [https://blog.alexellis.io/kvm-kubernetes-primer/](https://blog.alexellis.io/kvm-kubernetes-primer/)
- [https://linuxconfig.org/how-to-configure-static-ip-address-on-ubuntu-18-04-bionic-beaver-linux](https://linuxconfig.org/how-to-configure-static-ip-address-on-ubuntu-18-04-bionic-beaver-linux)

*Cover picture: "Gutenberg Publishes the World's First Printed Book (Illustration) Civil Rights Medieval Times Famous Historical Events Visual Arts"*

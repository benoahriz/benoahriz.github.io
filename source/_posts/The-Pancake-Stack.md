---
title: The Pancake Stack
date: 2017-08-23 10:19:52
tags:
---

# The Pancake Stack

At [Nitro](https://engineering.gonitro.com/) we've tried to embrace the idea that [simple systems](https://engineering.gonitro.com/can-micro-services-systems-be-simple/) are key to scalable and efficient infrastructure and micro-services and help maximize innovation, velocity and trust.

If you haven't heard this talk by [Rich Hickey](https://en.wikipedia.org/wiki/Clojure) please take the time to watch [this talk](https://www.infoq.com/presentations/Simple-Made-Easy). The points made I feel are invaluable for systems and software development. "Simple" is the opposite of "complex" which means "being intertwined" or "being tied together". Simple is not easy but simple is something to strive for whenever possible. Simple systems are easier to understand and reason about, support and change. If there is one thing we know in this industry it's that there is always something better coming out and it can be valuable to replace older components with newer more simple more efficient ones. Simple systems are good systems. Simple systems are loosely coupled.

All of the current infrastructure and systems stacks/platforms have their benefits and drawbacks but many attempt to solve the same functional goals. Our code has a lifecycle: It's built, deployed, tested and promoted to production, there also needs to exist a mechanism for rolling back versions and killing obsolete or errant tasks. We need a system that can efficiently manage compute resources and get our code in front of our customers. The systems and platforms that can serve this lifecycle are generally composed of several components.

* Compute resources.

* Resource managers.

* Schedulers.

* Service discovery.

* Load Balancers.

There are many stacks out that herd these components such as, Kubernetes, Docker swarm, Mesosphere, Vmware, Rancher, Heroku, ECS.   We decided to take individual application components and create our own stack that serve as our infrastructure platform. Lets walk through each layer.

### **[Cloudflare](https://www.cloudflare.com/)**

Cloudflare is the first layer we use, which handles dns, ssl termination and DOS protection. In a [highly available](https://en.wikipedia.org/wiki/High_availability) system there will be multiple ingress endpoints and cloudflare has a nice api that we use for updating dns records in the event that an AWS zone goes unhealthy. We use a tool [Ben Parli](https://github.com/bparli) developed called [Goavail](https://github.com/bparli/goavail). Goavail is a simple IP monitoring and fast DNS failover agent written in Go. More information on that can be found [here](https://medium.com/@bparli/writing-goavail-a-cloud-monitoring-and-fast-dns-failover-agent-7c59254a5c45).

### **[Traefik](https://github.com/nitro/traefik)**

After ingress traffic makes it through Cloudflare it hits our ingress termination points. We previously had used haproxy for ingress zone termination which was great but moved to a proxy called [Traefik](https://github.com/nitro/traefik).  Since schedulers can place tasks on any number of hosts in any part of the infrastructure there's no guarantees that an app will stay in one place for long so dynamic service mappings are critical for ingress traffic to get to the right place. Traefik made more sense than haproxy as we were able to write a plugin to automatically configure backends using our service discovery tool Sidecar. Ingress traffic comes into traefik and based on the status of the internal apps Sidecar let's Traefik know where to send the traffic.

### **[Sidecar](https://github.com/nitro/sidecar) / [haproxy-api](https://github.com/nitro/haproxy-api)**

Sidecar is an eventually consistent service discovery platform where hosts learn about each other's' state via a [gossip protocol](https://www.serf.io/docs/internals/gossip.html). Hosts exchange messages about which services they are running and which have gone away(tombstoned). Sidecar optionally works alongside the included app called [haproxy-api](https://github.com/nitro/haproxy-api) which takes state information from the Sidecar cluster and updates a locally running copy of haproxy to handle the dynamic port mappings to service ports. In our environment all apps are shipped as Docker containers and Sidecar is designed to work best with docker.  Service names and ports are configured using docker labels which get passed through the system by our deployment tooling Nmesos and singularity/singularity-executor and then to docker.  

### **[Sidecar-executor](https://github.com/Nitro/sidecar-executor) with [Vault](https://github.com/hashicorp/vault), [Docker](https://www.docker.com/) and [Nmesos](https://github.com/Nitro/nmesos)**

With Sidecar and Sidecar Executor you get service health checking unified with service Discovery, regardless of which Mesos scheduler you are running. The system is completely scheduler agnostic to the extent possible.  We happen to use it with the singularity 

Docker is how we ship our applications. Every application gets delivered as a container that is configured by environment variables. We like to adhere to the [12 factor app manifesto](https://12factor.net/) as closely as possible which reduces friction in our infrastructure and keeps things consistent. Every app is delivered as a container and each container is configured via environment variables. Scaling is accomplished by adding more concurrent containers by using our scheduler(in this case its Singularity).

Nmesos is a command line tool that leverages Singularity API to deploy services and schedule jobs in a Apache Mesos cluster.

Sidecar-executor is designed to optionally work with vault so secrets do not have to be exposed to the machines executing deploys or the singularity api.  

### **[Singularity](https://github.com/HubSpot/Singularity)**

[Singularity](http://getsingularity.com/) is a Scheduler (HTTP API and webapp) for running Mesos tasks and keeping track of them.  There’s not much to explain here other than we use Nmesos as mentioned above to send task requests to the mesos cluster and singularity takes care of making sure the right amount of tasks are running at any one time.  

### **[Docker](https://www.docker.com/)**

Docker as mentioned earlier is how we ship our applications.  As well as being the package format of our applications Docker also takes care of assigning dynamic ports and making sure the networking(at the container level) the volumes(again at the container level) and the security of the container is functioning as defined in the job definition.

### **[Mesos](http://mesos.apache.org/) with [Zookeeper](https://zookeeper.apache.org/)**

Mesos might have one of the most important jobs of all.  It keeps track of the resources available and allows tasks to be started in the right places. Apache Mesos abstracts CPU, memory, storage, and other compute resources away from machines (physical or virtual).  This gives us a pool of resources that can be fully utilized.  

Zookeeper allows mesos to keep track of state across the resources.

### **[Ec2](https://aws.amazon.com/ec2/) with [Proviso](https://github.com/nitro/proviso), [Ectd](https://github.com/coreos/etcd) and [Ansible](https://www.ansible.com/)**

We use AWS Ec2 because it has a mature API and allows us to scale infrastructure costs as needed.  We developed an internal tool called Proviso.  We use Proviso to manage our EC2 instances clusters. It stores the cluster state in Etcd, so we can reliably query, provision and decommission instances.  Once a new instance has been provisioned, we use Ansible to run the bootstrap template. This template installs the mandatory packages and sets up the tooling required to monitor the new instance and to register it in the Mesos cluster.  The bootstrap just installs the bare minimum needed tools to get monitoring in place and mesos agent running on the machine. 

Special shout-outs to:

Ben Parli [https://github.com/bparli](https://github.com/bparli)

Mihai Todor [https://github.com/mihaitodor](https://github.com/mihaitodor)

Felix Borrego [https://github.com/felixgborrego](https://github.com/felixgborrego)

Karl Matthias [https://github.com/relistan](https://github.com/relistan)

In following articles I plan on overviewing in more detail the individual software components with examples.
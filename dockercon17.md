# Tues morning

## Intro

Not terribly interesting.

## Solomon Hykes

@solomonstre

"Tools of mass innovation"

This is basically a sales talk.

Brought up "complaint driven development" as the docker team's process

Introduced:
* Multi-stage builds: separate build environment from runtime env.  More than one `FROM` and `COPY` in the file.
* "Desktop to cloud" - whaley docker connects to cloud swarm deployment directly.  Holy crap.
* linuxKit - linux subsystem for... everything?

Demo:
https://github.com/atseadx/atsea
https://github.com/atseadx/atsea/pull/21

This is a really cool demo: pretended to push a PR live.  Did in fact merge live.  Appeared to push to a staging environment: didn't check if that was real.
`docker stack deploy`

These features are in edge, next release

"Going to production securely is hard and getting harder"
swarmKit: secure orchestration.  mutual TLS among all nodes in swarm, even in orch. layer

Diverse infrastructure

Docker secrets, encrypted by default at rest.  Added in swarm.  `docker secret ls`

https://atseashop.com/index.html is the example used several times

Live open sourcing.  Made https://github.com/linuxkit/linuxkit public

Long setup to "shared chassis" - the Moby Project.  Framework for assembling components.  "Reference assemblies" - templates of component integrations.
https://mobyproject.org/ https://github.com/moby/moby/ - this actually seems like a fairly serious tool.

[influxdb](https://www.influxdata.com/) - alternative to prometheus

"RedisOS" containerd, linuxd, hyperkit

## John Gossman
Azure architect from MS

Announced the ability to run linux containers inside windows server under hyperV.  Not released, prototype.

## The images

Laurel, a french person living in SF
https://twitter.com/laurelcomics

## Using docker: dev
John Zaccone IBM @JohnZaccone

Would things have gone differently for Michael Bolton if he had used docker for deployment?

* start as apologia
* docker UCP - highly distributed and secure, using docker's notary to verify fidelity
* https://github.com/jzaccone/office-space-dockercon2017
* New commands `docker container ls` instead of ps; `docker system prune` cleans up unused, untagged
* Demo a `docker-compose.debug.yaml`: `docker-compose -f docker-compose.yaml docker-compose.debug.yaml up` - multiple files overlay!!!
* Multi-stage builds available in v 17.04.0 - currently in edge
* http://training.play-with-docker.com/ - for learning about swarm etc
* can add 'deploy' section to compose file which becomes inputs to swarm mode; use `docker stack` command

Scott Coulton, puppet

## networking w/ docker
Pradeep Padala @ppadala cisco

Contiv - networking solution contiv.io

* open source - no enterprise edition
* Container fabric: for l2, l3, overlay, ACI
* rich policy model - developer driven policy template files for each service

## How to make effective images
Abby Fuller, AWS @abbyfuller

Layers, minimalism, Docker Security Scan

More layers, larger image, longer to build, push, pull

Use the cache by putting unchanging layers first.  First cache invalidation invalidates subsequent layers.  Alternative, `ONBUILD ADD`, `ONBUILD RUN` https://docs.docker.com/engine/reference/builder/#onbuild - instructions only done when the resultant image is used as a base for another

Switching `USER` adds a layer.  Try to use the built-in user, or delay switching.

If you need a big thing, you should make one single step that gets and also removes it, or else that layer ends up in the build.

`FROM scratch` - space saver.  Totally empty image.  Good example for a built artifact (like jar or go), separate from the main container.

node:
`.dockerignore npm-debug.log` - if it doesn't belong, don't put it in the container.

Here is a nice trick to cache node modules:
```
COPY package.json
RUN npm install --production
COPY . .
```
That way, package.json's contents are used as a cache invalidator for the install step

`docker image prune -a` - gets rid of all untagged images not used by a container

`docker system prune -a` - gets rid of networks, volumes, etc


When do I want a full OS:
* compliance maybe
* security maybe
* dev tooling

## Evolution of container orchestration at alibaba

Li Mi, Dir of cloud engineering

Started dockerizing core biz applications last year, over 175k RPS on ecommerce, running 300k containers in production at any moment: largest docker deployment in the world.

40 deployment regions around the world: provide their own services. Launched container service december 2015, both public and private cloud.

Fully compatible to compose v1/2 swarm mode.  Each cluster dedicated to one user.  Assemple in a declarative way.  Use volume plugin (`driver: ossfs`).  Use labels to bind ports to load balancer nginx.  docker engine `placement.preferences.spread`

Deply monitoring agent as peer of docker engine.  Declarative scaling

## Decades of storage innovation ... itty bitty living space
Garrett Mueller, NetApp

What is NetApp?  Network Attached Storage (NAS), snapshots.  Been around since 93.

They have a docker volume plugin.

What can ya do?
* Run stateful services in containers
* instant copies of dev workspaces: cloning, snapshots
* instant copies of prod

Working groups: CSI (container storage interface) and more

His team built snapshot support directly into docker itself, go volume plugin
`docker volume snapshot` and `docker volume clone`

## Local to production at Intuit

Harish Jayakumar - docker solns engineer
JanJaap Lahpor - intuit devops engineer

Product's backend is in docker enterprise

Decided to seriously switch to Docker after swarm got pulled into docker core.

"The more you can do local instead of in hosted environments, the faster you can go"

* Dev productivity
* Hybrid hosting (same artifact on prem and in AWS)
* patching as code

They did the _opposite_ as us: small, on prem
Important to partner with the product owners.

Concept of guard rails:
* isolated hosts,8
* no data storage in container or host (no volumes),
* no container-to-container communication,
* use existing supporting technologies: change nothing but swap vm -> container
* only stateless java 8 (where they had all the knowledge)
* Applied to the _whole_ sdlc

Time to get off the macbook - docker enterprise helped them

What is Docker for Enterprise?
* docker enterprise engine is identical except has LTS
* Docker Trusted Registry:
	* security scanning
	* can be hosted on prem
	* image signing, notary, requirement that images be signed
* universal control plane: swarm++
* Totally pluggable

Did change a single vendor for anything.  Just need orchestrator and service discovery (and you need to solve monitoring and logging)

They ended up building 4 disjoint swarms: 1 for each distinct network zone (public zone, private zone)
_Really_ want a "fleet" manager

Decided bare metal - unorthodox.  Cheaper because able to do fewer hosts.

_It took 1 sprint to get 1 service to production_ - fuck me

Ran into very rough seas - having the vendor relationship was very important.  Quorum loss, crashes, etc.
* Stale ingress LB: dead containers could stay up in the LB.  No easy way to list members.
* In 1.12, they introduced a built-in LB
* Ran into port exhaustion on bare metal (not an issue with VMs because VMs give you disjoint TCP/IP stacks)
* Applications were not coded with sufficient fault tolerance and needed significant changes
* Zombie containers:
	* Caused by automated OS patching on running hosts
	* Right way: drain, patch, reprovision

What is a stack? Logical application is a stack.  The main unit of docker swarm.

## Wednesday

@golubbe

## Head of infrastructure and operations at Visa

@ksr_swamy

Speed and efficiency
Provisioning time: weeks, days
Patching: difficult

Infrastructure growing much faster than headcounts

Machines would decomm in ~3 months

Lessons learned:
* Granularity: big images problematic
* memory footprint
* load balancing - wish there were more features

Visa is about to expand to 5 groups (in other words, they are still in closed beta)

Back to the original dude.  Emphasize polyglot infrastructure.

"start with a secure base and containerize apps"

"build a supply chain" - container as a service

Securing the process from dev to build

Tag containers with things like 'PII' - which would mean "can't deploy to public cloud".  Enable devs to define their security policy needs.

Docker trusted registry has "policies" configurable

Announcements for vendorexpansions: linux for z Systems and power series (something about mainframes)
Docker on GoogleComputePlatform and AlibabaCloud

image2docker converts vmdk to a container

Containerize an oracle DB.  https://store.docker.com/editions/enterprise/docker-ee-server-oraclelinux?tab=description
BOOM

Mark Cavage VP product dev at Oracle

Docker Modernize Traditional Apps program: "turnkey" upgrades to Docker Enterprise without changing source
http://docker.com/MTA

## Aaron Ades AVP solutions engineer MetLife

Talks about th ehistory, 1868 to present (1950s univac, 1982: still have code from 1982 running in production, 2000 goes public)

Talks about the strategy to decompose and retire legacy apps:
* Wrap (wrap the application logic)
* Tap (uhh something about different backends)
* Scrap

Open benefits enrollment: huge traffic surge (25x).  Utilize public cloud (Azure) for surge workloads

"In some extreme cases, almost 70% consolidation" - higher utilization

Story: they formed a team called "MOD squad" in which they made small cross-functional teams.  He's all excited about the chatops angle.  "Most fulfilling project ever worked on".  Did it in 5 months - fuck me

## burnout

Maslach Burnout Inventory
http://www.mindgarden.com/117-maslach-burnout-inventory

* Exhaustion
* Cynicism
* Efficacy

## Tim Tyler Solutions Engineer, MetLife

Customer-facing applications

The idea of "enterprise journey" is a big theme

Started with "let's build a cluster".  Tossed out 'waterfall'.

Built a team:
* cross functional
* very focused
* high diversity
* freedom to explore
* Built a new area for the team
* Solved problems on-demand as opposed to making meetings
* Every member empowered to make decisions within group context

Brought in open source tech, which they basically did not do.
* docker data center 1.12
* redis
* grafana
* prometheus
* rabbit

Still battling culture.
Do they need an Open Source Governance Model?

Process and procedure: still not figured out.  For example: who owns the label dictionary?
Takeaway: "mod squad" team should have done a better job engaging application teams broadly to surface governance questions.

Tips:
* tag and label cluser engines and nodes
* label early and label often
  * geo code
  * test script
  * environment
  * maturity
  * maintainer
  * ops guide location
* follow a label convention `com.company.docker.something.helpful`
* Lint your YAML and compose files
* Chaos monkey: built a doc with failure modes
* Held War Games with the ops team:
  * Done over a week
  * Using prometheus
  * Did not sit with ops while app devs broke the shit
* Going to try https://github.com/sstephenson/bats :
* Have a stable demo environment:
* "break" Demo and let it heal (prometheus and grafana)
* histrix dashboard: https://github.com/Netflix/Hystrix/wiki/Dashboard

Training:
* Expect resistance
* Plan shallow and deep dives
* Plan to do it again and again

Operational handoff
* Don't underestimate it
* "Everything is the same" but it's all brand new

Started in cloud, then in-house.  This is more like athena than other use cases.

Persistent storage _is the elephant in the room_ - is there a right way?  They went super simple.  (elephants and whales are both in order _Artiodactyla_, Undulates)

They have databases in containers even, API gateway in a container

* team of equals that thrived
* seeds of DevOps
* renewed open source interest
* ChatOps

Getting there fast caused a lot of disruption

## Become troubleshooting pro
Jeff Anderson
@programm3rq

### Put data in a volume

`docker diff <image>` - diff filesystem from image
`docker cp` - copy files from/into volume

### Mount local

`sqlite3 -bail test.db` - fails because file permissions

Takeaway: the numeric uid is what matters
Files created by container owned by uid = 0

Prefer `chown $( id -u ) <file>`

Docker for Mac actually does magic to ensure files written by containers match your mac uid

### Networking

Suppose application together with nginx file
Unexpected 502 bad gateway!

The reason: curl from nginx (over localhost) to app fails

localhost, eth0 of docker host.
Each container gets a full copy of the full networking namespace

How to get rid of hardcoded IP and app port publish?

Have to add docker network service discovery:
* Docker runs a resolver at 127.0.0.11
* Resolves container IPs by name or alias
* nginx config: `proxy_pass htttp://web:8000` and `resolver: 127.0.0.11`

### universal control plane
UCP exposes docker daemon API on port 443
The web app provides the docker daemon API through the same interface as the GUI

Client bundle: causes issue with docker-compose but not docker CLI

Useful helpful thing!  A command for examining certs
`openssl x509 -noout -text < example.pem`

Toolbox!
`socat` - bidirectional TCP/DUP/stdio/pipes/sockets
`jq` - CLI for JSON
`nsenter` - enter arbitrary Linux namespace

```
socat -v tcp4-listen:5566,bind=127.0.0.1,reuseaddr,fork unix-connect:/var/run/docker.sock`
docker -H localhost:5566 run 
```

```
curl -s --unix-socket /var/run/docker.sock http://containers/json
```

Bunch of namespaces: PID namespace, ingress host namespace

Suppose host A is working and host B is not doing networking well.

How to Ask a Question

```
<statement of observation>

|---------------------------------------|
| demonstration of relevant observations|
|---------------------------------------|

<question>
```

Forums:
https://forums.docker.com
Slack:
https://dockr.ly/community


## Docker Networking in Production at Visa

@churchofmark Mark Church from Docker
Sasi Kannappan from Visa

Talk about the Visa story and unanticipated problems

Speak in a different language for networking
* 100s or more containers per host
* container lasts minutes or months
* more horizontal distribution of netork traffic (E-W)

Container Network Model is the "contract" or interface between the engine and the networking layer
* Portable policies
* Plugin design (batteries included but removable)

Visa
In production for 6 months
* ~100 containers running atm, can scale to ~800
* 2 prod, 2 sandbox, 2 regions

Core systems in a private cloud, based on virtualized models

Looking at huge transactional growth year-over-year.  Needed to be able to scale continuously instead of periodic major refactors.

Goals:
* microservice architecture
* dynamic scalability
* operational simplicity
* load balancer-less

First pass:
* scheduling - ucp
* service registration - gliderlabs + consul
* service discovery - consul + dns
* load balancing - consul round robin DNS & health check
* connectivity - docker bridge driver (as opposed to overlay)

Each host has consul agent, registrator: extra complexity.  Extra health checks on those additional components

Maybe the takeaway is that rolling your own scheduler is trouble.

Then Docker made Docker Networking Architecture: this colocates a DNS service with each docker container

Every service gets a VIP and a DNS entry

Docker 1.12 introduced health check and built-in load balancing https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/ with Swarm

Second pass:
* scheduling - ucp
* service registration - docker engine
* service discovery - docker engine dns
* load balancing - swarm VIP LB
* connectivity - docker overlay driver

Bridge: choosing ports runs out of nodes
Overlay simplifies much; configure your app to have some internal port mapped to a docker-managed overlay network port

https://success.docker.com

## Closing keynote

Cool Hacks

### play-with-docker
Mantika - http://labs.play-with-docker.com

Can share state by passing around URLs

Can create multi-node swarms

Can use play-with-docker as a docker-machine driver

`PWD_URL=whatever docker-machine create -d pwd`

How does it work?

ELB -> sends to a swarm host.  Each has a PWD daemon.
Created session creates an overlay network
Created instance is an isolated Docker in Docker

3500 containers per host (r3.4xlarge)
Swarm in Swarm

In Docker, they are using internally for training.

### Alex Ellis 
@alexellisuk

app dev ADP

Serverless - Function as a Service

https://github.com/alexellis/faas

Amazon uses for echo

minio-DB


## Ben Golub
@golubbe

# Thursday

## But I'm a SysAdmin

Mike Coleman @mikegcoleman tech evangelist at Docker

IT professional more than developer

VM and containers have fundamentally different benefits.
A VM is a house.  Can only get as small as MBs, generally GBs
A container is an apartment.  Docker engine is the doorman.  Containers _share_ the kernel.

Why docker on bare metal?  Typically, performance (skip the hypervisor)
But problem: give up on the benefits of consolidation.

Most people are doing docker on VMs, though, because high performance less important.

### Container consolidation testing
* docker cs 1.12.3, vmware, commodity hardware
with sysbench to test capacity

8 VMs w/ 1 workload compared to 1 VM w/ 8 containers: increase utilization by 25%, 30% less storage, 7% less RAM

8VMs 1 workload each compared to 8 containers bare metal: increase utilization by 47%

They needed to tune: CPU pinning etc.  Without tuning, performance was _worse_

### Security

Security is not just isolation.
* patches
* image signing/fidelity
* sensitive data/passwords (evidently environment variables considered wildly insecure)

Key features (plug for docker datacenter or CE swarm):
* usable security
* trusted delivery
* infrastructure independent

Trusted Registry allows for multiple signatures on a given image

### Docker for AWS

Docker CE for AWS on docker store starts of with a cloud formation template

"Use cloudwatch for container logging?"

Does a TON of work for you.  Can SSH into managers but workers are all closed.

ssh in, `docker node ls` `docker service ls`

Half of people are moving apps from VMs to containers.

### Big takeaways
* Need to involve everyone together:
  * dev
  * ops
  * security
  * executive
  * performance
* Choose good partners
* Define guardrails: handle one application with defined success criteria
* What's the plan for:
  * port mappings
  * environment configs
  * persistent data

## What have Namespaces done for you lately

Liz Rice, @lizrice from Aqua Security

* namespaces
* cgroups
* fork bomb

### namespaces
Syscall interface
* timeshare system
* proc ids
* mounts (original namespace, so the flag is NS)
* network
* uids
* IPC

### demo
Demo written in golang

```
func run() {
  //cmd := exec.Command(os.Args[2], os.Args[3:]...)
  cmd = exec.Command("/proc/self/exe", "child". args...);
  cmd.Stdin = os.Stdin
  cmd.Stdout = os.Stdout
  cmd.Stderr = os.Stderr
  cmd.SysProcAttr = syscall.SysProcAttr{
    Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS,
  }
  cmd.Run()
}
func child() {
  cmd := exec.Command(os.Args[2], os.Args[3:]...)
  cmd.Stdin = os.Stdin
  cmd.Stdout = os.Stdout
  cmd.Stderr = os.Stderr
  syscall.Sethostname('container')
  syscall.Chroot('/path/to/copy')
  syscall.Mount("proc", "proc", "proc", 0, "")
  syscall.Mount("something", "my_temp", "tmpfs", 0, "")  // written to memory rather than disk
}
```

Steps:
* Need a `run` and a `child`: mods to env in the child
* chroot to get isolated env
* need to mount proc pseudo directory
* need to create a "mount namespace" (NS) so that mounts are isolated from host

Super important fact
* Stuff in container is visible in host `/proc`.  _ENVIRONMENT VARIABLES ARE NOT SECRET IN DOCKER RUNTIME_.  You can also kill proc from host, etc.


### cgroups
/sys/fs/cgroup/memory/docker

Contains files whose values are the cgroup configuration

pids.max => 20
notify_on_release => 1

`:(){ :|: & };:` - create a function named `:`, which calls itself and pipes to itself and forks off

Used as a demo to show that cgroup process limits she implemented worked

## Container performance analysis

Brendan Gregg performance architect Netflix

* identify issues in host v container using system metrics
* app code w/ flame graphs
* kernel tracing tools

Netflix : titus project for cloud runtime platform
* scheduling
* container execution
* http://techblog.netflix.com/2017/04/the-evolution-of-container-usage-at.html
* over 1M containers deployed a week

stream processing: flink
UI: nodejs

cgroups (control groups) restrict usage
cgroup + namespace = container

### CPU shares
CPU limit = 100% * container's shares/total busy shares
Minimum guarantee = 100% * container's shares/total allocated shares

Can make performance anaylsis complicated, since resource allocation is dynamic

### OS config
FS:
https://docs.docker.com/engine/userguide/storagedriver/

Networking

### USE method
Draw a diagram of the system, ID each resource, for each check
* utilization (eg time busy)
* saturation (eg run queue length)
* errors (eg ECC errors)

### Host tools
...assuming you have host access

"most of the time, it's not the container that's to blame" - often just the physical resources

https://brendangregg.com/linuxperf.html
http://techblog.netflix.com/2015/11/linux-performance-analysis-in-60s.html
use method linux

Event tracing: iosnoop (from perf-tools, or BCC project)

systemd-cgtop - a "top" for cgroups

`docker stats` block i/o, PIDS, utilization

`/proc/*/ns/*`

`nsenter -t <PID> -u hostname`
`nsenter -t <PID> -m -p top`

Profiling
naive:
`perf record -F 49 -a -g -- sleep 30; perf script`
Doesn't work: perf not updated

https://github.com/brendangregg/FlameGraph

/sys/fs/cgroup/.../cpu.stat
/proc/PID/status nonvoluntary_ctxt_switches

Hard to diagnose within a container: mixed up host and resident stats

### Tracing

BPF - kernel virtual machine
https://github.com/iovisor/bcc

Slides: https://www.slideshare.net/brendangregg/container-performance-analysis

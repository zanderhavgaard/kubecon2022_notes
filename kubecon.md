# Format

> These are notes/comments

# Blogpost Ideas

- k8s as a generic control plane // k8s as a generic tool for managing the desired state of assets // k8s as a generic executor of desired state (and not just for running services)
  - workloads, applications, deployments, environments, k8s clusters, (cloud)infrastructure, digital twins
  - in the digital twin talk, they talked about how a digital twin is managed from k8s as a resource, and that it could potentially be moved from cluster to cluster
  - keywords:
    - crossplane
    - gitops
    - "control clusters"
    - K8s resource model (KRM)
- CI/CD orchestration vs. choreography
- the rise of high level interfaces (ala PV vs PVC), dev consumes resources defined by DevOps/AppOps/Ops
  - golden paths / paved roads
- debugging k8s applications using ephemeral containers

# gitopscon (Tuesday)

## Opening Keynote - What is GitOps?

"gitops is an implementation of devops"

drift is dangerous and a security issue
drift is when user login to the console and make changes
your entire team should not have access to the console

We declare our state and it is continuously reconciled

closed vs open loop systems

reconcilation can be different actions, like a notification, or making a change

gitops does not neccesarily require k8s, but they tend to go hand-in-hand

"gitops moves at the speed of the k8s audit log"

"it's just a cron job ..."

- teams who use gitops
  - make more deployments

> TODO what were the other ones?

gitops is great - but it is hard, and most teams are still somewhere on the journey towards gitops

### GitOps Principels 1.0

96 interested parties, 60 companies

"what's next for gitops?"

you are doing gitops with k8s, but do you do it with the rest of your infra? what about your s3 bucket?

handling secrets is not a solved problem with gitops

"plain text secrets are technically gitops, but obviously not the right way"

project repo:
https://github.com/open-gitops/project

gitops discussions:
https://github.com/open-gitops/project/discussions

"gitops can allow you to turn off your IT, it can also get you into a lot of trouble if not managed properly"

"all of the discussions around gitops are still open"

## Everything as code: declarative applicatoin delivery with gitops workflows

<computer crashed ...>

## gitops secret management

gitops apply to different contexts:

- infrastructure
- applications
- cloud and non-cloud assest

"not using cloud? gitops still applies"
"you have servers? you can use gitops!"

gitops requires access to some sensitive assets:

- source code repos
- application configurations
- infrastructure management
- access to platform resources (aws, k8s, etc.)

do not store sensitive resources in plain text in git ...

"as soon as commited to git, it is there forever"

understand the capabilities and limitations of your tools

security by obscurity is not security ...

two primary approaces for storing secrets:

- encrypt them in the git repo
- point to them in some safe place (ie. secret store, vault)

"the good news is that there are a lot secret management tools, the bad news is that there are a lot of secret management tools ..."

popular tooling for managing secrets:

- SOPS
- sealed secrets (kubeval)
- ansible vault
- helm vault

kubernetes plugin management tool - krew?
https://krew.sigs.k8s.io/plugins/

> TODO look into krew

k8s solutions for handling secrets:

- controllers
- webhooks
- operators

common access models:

- basic auth
- tokens
- OIDC (is great!)

k8s secrets are great - but not secret! - not great for production

sidecar/init container for managing secrets,
sidecar/init container will fetch secret from remote secret store on startup and make it avaialble to main app

k8s CSI driver, interacts with external secret store and makes them easily available to the k8s env

> TODO look more into k8s CSI driver

### preventative measures

use tools to detect secrets in git:

- detect-secrets
- git-secure
- repo-supervisor
- GitHub secret (automatic) scanning

"Security is continuous"

## gitops based IaC with rancher fleet and crossplane

how to combine gitops and IaC? - introduce two new tools to solve this

Multicloud infrastructure management
is hard ...
end up writing shell scripts that run in terraform

IaC is great, but still immature, no silver bullet to solve all problems, need a lot of different tools for different challengens
No standarized APIs
Drift is still a challenge - can I actually apply my changes?
Platform/SRE/DevOps team don't have time to deal with it ...
Develops typically not interested in managing cloud infra

gitops tools landscape:

- FluxCD
- ArgoCD
- FLEET (suse)

fleet is a tool for managing multiple clusters, in multiple clouds, at scale
Central k8s cluster manages downstream k8s clusters
downstream clusters can be grouped in stages, eg. dev/test/prod
central cluster monitors a git repo and configures downstream clusters based on desired state

We can also create other cloud resources that our applications need, like s3 buckets

"create a self-managed layer"

terrajet - tool to build crossplane providers

crossplane enables dev vs. admin seperation of concerns, ala PV vs PVC

now we combine fleet + crossplane

gitops defines desired state
-> fleet central cluster reads desired state
-> central cluster uses crossplane to provision custom resources on multiple clouds
-> does crossplane or fleet manage clusters?

> kubectl get clusters is pretty cool ...

takeaways (from the presenter):

- common API and workflow
- principle of least privilege
- avoid chaos of drifted configurations
- manage day-2 operations

> TODO find his example repo and figure out how this exactly works ...

# GitOpsCon lightning talks

## intuit

capability team vs. paved road team vs. product dev team
capability team manages tools and capabilities, paved road team makes it easy and accessible for customers to consume capabilities, dev teams make products
Paved road team makes fx ui where devs can request resources

how to differenciate "paved road" from tickets?

added assests have a lifetime, eg. 1 hour to 1 month -> drastically increased experimentation

## cluster api

cluster api seems like a nice k8s native way of managing clusters (in the future?)
bootstrap local cluster with kind and use that to bootstrap other clusters?
Kapp controller?
uses Kapp to manage apps?

# Kubecon day 1

## Opening keynotes

"Focus on the tech, the rest will follow" - this is the case for a tech conference, but not how we want to do things?

Continous Improvement was highlighted by the Mercedes-Benz keynote as an importantt part of their keynote.
Continous \* is still essential for DevOps succes.

## Benout Moussaud - Deployment Ballet talk

"Code not deployed to production is useless"

Dev vs. AppOps (DevOps) vs. SRE vs. Ops

We want to deploy our applications, and thus need to promote it through a number of environments
this implies that we have a versionable artifact

"this is nothing new ..." - formalized in books in the late 2000s

Current challenges in CI/CD:

- Natural design (?)
- impact on modification (?)
- tightly coupled
- Single point of failure
- (pipeline is) Rigid

Deploying to production is not a linear journey, but handled by central orchestration tools
Every time I need a new type of step in my deployment pipeline, I need a new central orchestrator

This gets more complicated when we have scale ...

Challenges at scale:

- Path to production (is different for each team/product)

"The orchestration pattern" vs the Choreography pattern
central control vs distributed coordination

choreographoy pattern:
a resource takes a number of inputs, and produces outputs
other resources watch for the output of other resources, and are thus triggered by an event (output) of another resource

Cartograhper is a tool for this

> TODO check this out on CNCF landscape

each step:

- watch repo
- build image
- tests
- workload config
- deployment

Is encapsulated as a distinct resource, and are collected in a `BluePrint` in `Cartograhper`.

devs specify a workload
ops/sre specify how to enable a workload
cartographer matches workload deployment requests with blueprints (think PV vs PVC)

`cartographer` runs in k8s

workload choreographoy resonates well with gitops

`cartographer` enables propagating values between resources in the build process

`Carvel` one tool, a single responsibility

> TODO look up carvel project

`YTT` for templating yaml
`KAPP` deploy and manage groups if k8s resources as "aplications"
`KAppCtrl` YTT + KAPP

> TODO how is this different from helm?

> NOTE we are still just templating yaml with more abstractions ...

> TODO tryout `$ kubectl tree ...`

Wrap up:

- opintionated way to production
- hide complexity (by using abstractions)
- standard contract (between Dev, ops, sre)
- seperations of concerns (technical)

This seems useful for big organizations, but that it might add a lot of complexity in the admin part of setting the system up.
Though it probably scales better than pipeline based CI/CD?

> TODO we should discuss this

checkout https://cartographer.sh

"how is carvel different from helm?" - apparently carvel can do more things than helm, and even template some things that helm cannot?

NOTE it seems that k8s is also being used not just for running workloads, but also as the interface for controlling other resources, like being the underlying foundation that enable a tool like cartographer to choreograph any number of different resources, that may not be k8s or are downstream clusters. I suppose this means that we end up with these "control clusters" that we use to control/orchestrate everything else we do, in a declarative way, instead of the more push-based pipeline way of doing things.

## Debugging ephemeral containers

this talk is setup as a interactive tutorial, should try it out after

Ephemeral containers is new feature in k8s as of 1.23
They are useful for debugging

debugging without ephemeral containers:

- debugging by attaching using exec and running some commands
  - limited to tools in the image
- debug by copy, by using `$ kubectl debug ...`
  - requires a restart
- will require modifying the number of replicas ...

debugging with ephemeral container:

- example: `kubectl debug -it -n <namespace> <pod> --image busybox -- /bin/sh`
- will create a new container in the same pod

creating an ephemeral container will modify the pod spec, and a `ephemeralContainers` key underneath the `Containers` key

containers are sandboxed, can't see the outside world, or into other containers ...
what enables this is linux namespaces

- which is what enables and controls container processes isolation
- each namespace is identified by an inode
- we can only do things in the namespaces that we are in

we can use `nsenter` to run a command in a namespace
use `nsenter --target <pid>` to start a shell using the same namespaces as that process

TODO look at the slides for how to run commands in the pod with nsenter

when starting an ephemeral container

- the network namespace is shared by default
- we can target the namespaces of a specific container `$ kubectl debug ... --raget <pod> ...`
  - "view of the world" should be the same/very similar to the target container

ephemeral containers are a "sub-resource" in k8s

- must have unique names
- not implemented in kubectl yet, must use api-calls for certain functionality (look in the slides)

`netshoot` container image with tooling for diagnosing system/network problems

distroless containers

- using ephemeral containers should make debugging distroless/scratch containers much easier

when an ephemeral container exits, the container goes away, but an entry will remain in the `ephemeralContainersStatus` that can include sensitive data, recreating the pod will remove this from the pod resource.

## Building digital twins for DFDS with crossplane and k8s

"We need digital twins to act as emissarys on our behalf in the digital world"

digital twin:

- digital representaiton of a physical asset
- mimics the behaviour of physical asset and vice versa
- offers a digital control plane for the physical world
- maybe even tether to the physical counterpart for the assets lifetime

useful for robotics, ships, factories, etc.

"in cyberspace there are no ships, just lots of sensor data ..."

use cases:

- trailers:
  - reduce empty loads
  - ensure integrity of "cold storage" services, ie. a container guarantees a specific temperature that must be maintained
- shipping
  - improve routing to reduce fuel consumption
  - centralize management of IoT sensors
- terminals
  - automate terminal processes (eg. moving containers around)
- Vessels & trucks
  - digital ownership of physical assets
  - enable remote control

The digital twin sits between the real-world (sensors) and control planes and applications

## Tales of burnout in the open source world

Burnout was categorized by the WHO as a condition in 2022.
The definition is chronic stress caused by workplace that is not being successfully managed.

How did we get better?:

- it's not a sprint, but a marathon
  - mental & compassion fatigue
  - inability to drive anything to completion
- put the oxygen mask on yourself first
  - reassess priorities
  - self-care and vacation are important, but not the only solution

taking a step back does not mean you have lost something
learning to be kind to yourself
communication is important
we have look out for each other
all of us help all of us to succeed

psychological safety is key

what happens when a project has no clear goal? -> chaos or stagnation

- stepping down (from leadership) is not a step back
- distribute responsibilities
- provide opportunities to level up

guilt is not anyones friend, nor good for anyone

"move fast, but don't break things!"

- what are we trying to change?
  - and at what cost?
- Rome was not build in a day
- pause and play

it is not worth you own health ...

See something? Speak up! (and do something about it)

> TODO see the slides for resources on burnout

## From k8s to PaaS to ... what's next?

tldr; golden paths / paved roads -> how much should you build yourself? and how to assemble the control plane effectively? -> the keyword is `platform engineering`.

developers are being told to "shift left" all of the things ...
We need abstractions / APIs and tooling to achieve this

"tool sprawl" is a real problem - it's confusing what tools to choose and how to use them for what?

every developer has to think about:

- code
  - developing new version
- ship
  - release new versions
- running
  - monitor running instances

"k8s is a yaml database with an API"

achieve good dev experience:

- treat the platform as a product
- you can't have good DevEX without good UX
- focus on workflows and tooling interoperability

Platform as product: "The golden path"
"make it easy for devs to do the right thing"

dogfood your own platform and products ("drink your own champagne")

you need good ux for good devx:
think personas:

- platform experts
- the "hipster engineer"
- the "99% developer"

User research is key:

- talk with users
- observe users

ArgoCD UI and CLI are good examples of good UX

finding the right balance between standardization and autonomy for platforms

the need for a platform control plane

insights from Ambassador Labs podcast:

- successful orgs are investing in golden paths (and platform teams)
- start small, get big. Should be developer led ("bottom up") - what do you use? what do you need?
- communicat the long-term vision (and biz goals) of your platform
- recognize the sociotechnical factor: adopt a "hospitality" focus for getting people onboard
- A good UI "Paints a thousand CLIs"
- eliminate toil: "automate and create" to remove friction
- Adopt standards: APIs, gitops, KRM (CRDs)

developers are our end users, we create a platform for developers to code, ship and run

developer control plane is what we need to build:

- we have many good tools
- but we need UIs to tie them together for devs

> User centered design -> User centered platform?

"You dont want a 10x developer, you want a developer who can 10 other devs more productive"

> TODO check out platform engineering guide

## Building a nodeless k8s platform

Goals of a fully managed k8s platform

"Traditional" k8s platform design:

- k8s level: KRM, kubectl
- infra/platform level: GKE, nodepools, etc.

"k8s is tool designed for proffessionals" -> we want the power, flexibility and scale that k8s enables

what should `$ kubectl get nodes` return? -> just show the nodes, be transparent
should the node details be available? -> decision was full transparency -> "trust but verify(able)"
should the users be able to interact with the VMs outside of k8s? (can I ssh to a node?) -> no, no need

Implementation:
uses a cluster autoscaler to determine the right number of nodes for the required workloads

# KubeCon Day 2

## Chaos engineering

Service reliability:
Important metrics are MTTR, MTTF (Mean time to failure)

(Chaos engineering is) "the science of breaking things proactively"

Mircoservices presents a reliability challenge, our applications are dependent on other services, that will go down ...

"Chaos engineering is a devops culture"

"Chaos testing" -> do a chaos test before integrating new code

`Litmus` -> end-to-end chaos engineering platform

We want developers to apply chaos to the service before we run it -> "shift left" -> develop **with** chaos?

Developing with chaos makes it much easier to introduce chaos in production

We want to test resilience with chaos both before integrating (CI) and before deploying (CD)

`Okteto` -> tool to deploy dev envs to k8s

We can use `Litmus` to orchestrate "chaos" tests, eg. kill this service at this rate, and apply this amount of load to this serivce, for x amount of time

Feature of `Litmus` is that it has a nice gui, making sharing results of experiments with colleagues easy

`oketeto` can be used to "sync" your local dev env, to a container running in a cluster.
This is cool b/c we can then edit the code, run it in the cluster, and then continousely run the chaos test against the code as we write it.

> TODO look more into oktetu -> looks a bit like bind-mounting your code in a docker container while coding, but for k8s?

Running chaos tests in pipelines when we integrate new work implies that we can actually **prove** that a change made the service more or less resilient

We need to make chaos engineering a commodity, by making it easy and simple to do, the more chaos test we run, the better we get at it

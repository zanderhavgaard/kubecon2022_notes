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
- OpenTelemetry looks like it will be very important in the near future

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

## OpenTelemetry: The vision, the reality, and how to get started

"how many tools do companies use to collect telemetry data" -> research shows that most companies use between 5-10 different tools to collect telemetry -> With OpenTelemetry they want to cut that down to 1

The vision: unified observability
Metrics vs. logs vs. traces

The reality: getting insights is hard, b/c we have too many different tools and too many different ways of aggregating data / instrumentation
This also implies a tight coupling between applications and observability tools

`OpenTelemetry (aka. OTEP/OTLP)` is a observability framework to capture metrics, logs and traces for cloud native software.
OpenTelemetry is the second most active CNCF project, only after k8s!

Applications generate and emit data, OTLP then collects data, processes it and exports results

OpenTelemetry specifies cross-language API spec, SDK spec and Data spec for metrics, logs and traces

Instrumentation of applications is handled with client libraries for languages -> the goal is low-code/no-code application instrumentation, but with flexible and powerful features -> "auto instrumentation" for low barrier-to-entry and black-box observability

`Collection` is handled by a `receiver`, which is then `processed`, and `exported` -> we can then use any combination of receivers, processors and exporters

`OTLP protocol` is a generic protocol to send telemetry, supports gRPC and HTTP 1.1, encoding is protobuf, provides a telemetry data model -> the protocol is what connects the receivers, processors and exporters, and allows for the generic architecture

"Is OTLP production ready?" -> "well... it depends"

State of OTLP:

- Traces
  - API, SDK, Protocol considered stable
  - many client libraries
- Metrics
  - Release candidate expected within a couple of weeks
  - API, SDK, protocol stable
  - Many client libraries, more coming soon
  - Prometheus support
- Logs
  - Hope to be stable during 2022
  - Protocol is stable
  - API, SDK experimental
  - log appenders for many languages are under development
  - focus on integrating with existing logging systems

How to get started with OTLP?

Start with knowing your stack:

- which languages? Multiple?
- which signals? (metrics, logs, traces)
- which protocols? (prometheus, jaeger, etc.)
- which analytics tools? (which exporters to you need?)

When you know these, check the status of components, and figure out what to use

> TODO check out hello-world OTLP guide, https://bit.ly/otel-kubecon

## Recreating production problems in the CI pipeline with eBPF

"I hate writing tests" -> so let's try to test without writing manual tests

We want to capture production data, and then replay that on our new version as part of the CI pipeline

"Treat your tests like cattle, not pets"

ways to replay traffic for testing:

- traffic mirroring (mirror prod traffic)
- traffic replay
  - capture prod traffic, save to persistent storage, then later replay on new version of app in CI pipeline

How to capture traffic? Also service-to-service?

`pixie` CNCF observability platform, automatically trace network in cluster, also CPU profiles, and others.
Philosophy is no code modification and no redeployment

eBPF?
"acts a bit like a breakpoint in a debugger" -> but runs a small program called a `probe`
Define probes, when specific things happen in the code, and then execute eBPF program, when the probe is triggered.
Should have negligible impact on performance, very efficient.

Pixie will trace all network traffic, by "snooping" on the kernel.
This is important, b/c it makes pixie independent of runtime/architecture.
Kernel level tracing -> no blind spots.
Can work with TLS'd traffic, by hooking into encryption libraries.

Pixie is deployed as a daemon set, which runs the pixie PEM (pixie edge module), which collects data.
Data can be queried with a high-level pandas interface.

> Where is data stored?

Pixie comes with a UI that can visualize network traces between services -> which is done automatically.

Integraging Pixie:

- input PxL script to capture data
- output is the data ...

## Choosing cloud native technologies for the journey to multi-cloud

Form3 (the example company) provides services for processing inter-bank transactions.
Challenges: process large volume of payments

- large volume
- must be reliable and durable
- difficult to maintain many connections and services

Backing sector is slow moving and one of the last to move to the cloud ...
... and worried about vendor lock-in.

Cloud agnostic technology is ideal to avoid vendor lock-in.

They deploy the payment service to AWS, Azure and GCP on their managed k8s platforms.
These are linked to on-prem clusters for regulation reasons.
They run a dev/test/prod on each public cloud.

Cilium:

- Uses eBPF to provide network functionality on linux systems.
- network policies
- cloud agnostic
- inbuilt observability

They are not using cluster mesh, but edge routers to connect the on-prem infra to the cloud infra.

> TODO read blog about cilium cluster mesh

NATS JetStream:

- messaging system
- mixed push/pull architecture
- cloud agnostic
- scalable and durable

"rules of thumb" (for choosing tools):

- test each tool end-to-end with different patterns of use/load
- expect errors and retries
- be cloud-agnostic, but also use cloud and push unnecessary complexity, like using managed k8s clusters.

## Service mesh at scale: how xbox secures 22k pods with Linkerd

Choosing which service mesh to use:

- Easily provide mTLS
- implements service mesh interface (SMI)
  - which enables canary deployments
- efficient resource useage
- low latency impact
- observability
- easy setup and maintenance

They looked at:

- linkerd
- consul
- istio
- aspen mesh

Linkerd was the best fit for their requirements / fit the needs

How xbox uses linkerd:

- mTLS with cert rotation
  - with zero config
- high availability mode
  - important for production
- metrics with Prometheus
- SMI extension for canary deployments

"setup seemed too easy"

canary deployments is hard:

- for things that are not HTTP servers is hard, like asynchronous and cronjob type workloads.
- not all services have constant or high traffic volume -> how to evaluate if a deployment is a success?
- traffic to livenes / readiness endpoints skew canary evaluation

engineering and cost savings

- was not a goal, but happened naturally
- engineers freed from managing mTLS
- reduced money spend on other observability solutions

Next up in the xbox linkerd journey:

- service-to-service auth
- multi cluster communication and failover
- proactive fault injection, chaos engineering

# KubeCon Day 3

## From cloud naive to cloud native

"Going to the cloud is easy today ... you just need a credit card"

But it is up to you to actually make going to the cloud a success, and actually add value.

"Using cloud native tools, doesn't just magically make your application cloud native"

The clound native pyramide:

- CN is not a tool, a process or an object - it's a mindset you have to adopt, as well as a kind of unformalized methodology
- Internal development platform -> automated, dev centric operations supporting hyper converged infrastructure. Usually supported by containers and k8s.
- Business Requirements <-> Application development
- Infrastructure -> is seen as the real challenge of adoptin CN

But the most important part is the foundation -> **the mindset** -> you can't train or buy a mindset, it must be cultivated -> culture is what matters.

Cloud based/using vs cloud native -> the imprortant difference is mindset and culture.

Misconceptions:

- "k8s is just a new hypervisor I run stuff on" -> k8s is two things:
  - a platform to build cross-platform platforms
  - a system to manage, enrich and run your applications
- "k8s is treated as a one time implementation; build once, hand over to ops" -> companies need to start thinking in products and not projects
- "we hand out great blueprints and best practices and are enabled to mass migrate to a cloud" -> you need to actually guide and consult your ICT - typically the group of people understanding CN is very small

Most applications are legacy, and are not made for running in a container orchestrated fashion.
By missing experience (on the dev side), new projects are not created with k8s in mind.

"How to stay on the CN side?":

- mindset is the most important
- mass migrating services created it's own set of problems
- identify the group of motivated people and enable them
- You must start from the beginning, there are no shortcuts ...
- Adopting cloud does not make you a CN business, it can only enhance it

"Companies choose cloud provider by cost ...", though the actual cost is not transparent -> cost of migration and knowledge loss can be a lot more costly.

"The platforms you build will be used forever" -> projects are timed and then terminated, leaving whatever is there to run.

Treat your internal development platform as **long lived**.

Development and product management:

- dev tools are sometimes more important than production systems
- don't build for one deployment, build for continous change

k8s:

- implement security, rbac and network policies from day 1 -> b/c it only gets more complicated the longer you wait
- don't skip the basics -> use liveness/readiness probes as well as resource requests/limits -> ensure apps can scale horizontally

multi-cluster vs single cluster: "well it depends"

Managing k8s clusters, anyone should be good with:

- automation of provisioning
- favor declarative vs scripts
- should be easy to create/tear down clusters
- build features you need, not fancy

Do not over-engineer:

- overly complex chained CI/CD processes
- homebrew cloud dev kits

> TOOD look over slides

Use open source, KISS.
Design and build for change.
Think about the end user who is not a k8s expert.

## "It's always DNS, expcept when it isn't"

Good war story.

The takeaway was that they had made a change to the gRPC connection to suit their needs, until they changed the setup, and it ended up causing some very hard to diagnose issues.

## Charting your own course through the Cloud Native landscape

How do you start researching a **big** topic (like k8s)?

It's helpful to have a planned pattern that you can repeat.

Create a feedback loop:
collect -> skim -> refine -> learn -> collect -> ...

At first you dont know what you dont know ... -> what concepts keep coming up? -> should I pay attention to them? -> look for Patterns

What assumed knowledge are they expecting me to have ?

When getting into k8s, they expect you to know linux, containers, network, etc. ... -> figure what assumed knowledge you need.

What resources resonate for you ? -> know what works, and what you get the most out of. Should you start with "k8s the hard way" or something simpler?

Loop of:

- define reasonable goals
- do reasonable goals

Find joy in the progress!

The goal is to keep learning at a consistent pace, and to feel that you keep learning at the right pace -> flow?

"How do I become a better comedian?", "You write better jokes.", "How do I write better jokes?", "you do it a lot, consistently, everyday".

Questions:

- what is your own personal level of tech experience?
- how much time do you have to do learning?
- what is your learning style?

Tips, tricks and takeaways:

- Find the resources that work for you
- Create sustianable habits for studying / learning
- Be good to yourself
- Find a mentor
- Deep learning takes time!

> Check out the slides, https://bit.ly/ChartingYourCourseKubeCon

## Throw away your password and trusting machine identity

> Check out the slides

### Nicolajs summary:

"Secret zero" bootstrapping is solved with OIDC, it's fairly simple to script your way out of if you know what these hardcore principles mean.
OIDC is here to stay.
Trusted Processing Module (TPM) integration looks exciting in the future.
When I say simple, it's like managing a nuclear reactor is simple.
SPIFFE is a standard, and SPIRE can self host an OIDC provider, if you are not using eg. a cloud provider.

## Kubectl said what?

> Slides have a nice overview of different states one might encounter when deploying to k8s, and steps to debug and diagnose these conditions.

## Crossplane intro and deep dive

What is crossplane?:

- framework for building cloud native control planes
  - without writing code
- cloud providers have been building control planes for a long time
  - k8s is a control plane for containers
  - crossplane helps you build your own, with your own opinions
- extensible backend to manage any infrastructure in any environment

"Crossplane is neutral place for vendors and individuals to come together in creating and enabling control planes"

It's a growing project

### The basics: managed resources

> TODO read the slides again

Managed resource -> k8s representation of a resource that exists outside k8s

AWS Examples:

- certs
- SQS
- caches
- k8s clusters
- etc...

Every **kind** represents a type of resource

`spec.forProvider` is where we declare how to configure the resource

managed resource reconciliation:

- Controllers reconcile CRDs with cloud providers and other APIs
- RDS controller runs in the cluster and does the actual reconciliation

Crossplane is asynchronous and runs in a loop, like all other controllers.

### Building your control plane: composition

- Assemble your own resources from multiple vendors/clouds

`Coposite resources` aka `XRD`.
Combines a group of other resources into a single resource.

Values can be propagated from resource to resource in a XRD using `pathces`.

### Extending crossplane

- Crossplane is highly extensible
- backend - `Providers`
  - you can build a provider to manage anything with an API
- Frontend - `Configurations`

devs -> Configuraitons / API -> crossplane (control plane) -> providers -> cloud providers (AWS, DO etc.) / other APIs

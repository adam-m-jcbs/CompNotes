# docker

```
docker ps -a  #ps, but for docker
docker run <image> #workhorse command
docker login #authenticate with hub or similar
docker pull <app> #get images from the registry
docker rm <thing> #remove containers, I think, but maybe more?
docker rmi <image> #remove image, don't let these proliferate!
```


# puppet
puppet is a ruby-based server config management tool.  It is a provisioner and
has resources very similar to the way Terraform does.  The main difference is
terraform is a bit lower level, able to provision whole cloud infra.  Puppet is
more about configuring at the server/node level after the OS and such have been
provisioned by terraform or similar.

puppet is pull-based!  Like ansible and others, it has a boss-worker
architecture (at least, that's how it is almost always deployed... puppet can
run in a more distributed mode, but rarely practiced as I understand).  But, the
workers pull from the boss instead of the boss pushing to the workers (as
ansible does).

This is important to be aware of, and can make puppet better or worse depending
on use-case and priorities.

puppet scripts are "manifests" and look like  `hellopuppet.pp`.  They are run by
the puppet daemon/agent and are written in a DSL (somewhat similar to how
Terraform is written in the DSL of HCL).

manifests rarely are stand-alone.  Rather, modules are common and good practice,
and code is broken up across directories using puppet conventions.  Please see
docs and be aware that while manifests can do things scripts do, they really are
distinct from bash scripts (namely, manifests are stateful and more declarative
in the sense of stating what the state should be, while scripts are more
imperative with a recipe of explicit instructions).

it's important and usually expected to have a `manifests` and `files`
sub-directory pair.  Files are often used to source data, input, etc, into
puppet resources.


# terraform

day 1: go from running nothing to running something
day 2+: evolve it
day n: we done wit it, time to move on: terraform destroy! terraform SMASH

terraform is infrastructure as code by design (mostly written as .tf files)

tf:
    workflow for creating and managing "infra" (eg vpc + sg + lb + VM)

three main commands during workflow:
    + refresh -> terraform compares what tf files say are expected with what it
    sees in the "real world" (e.g. by polling services and shit in the
    background).  "this is what you want dev, lemme go see what's actually
    running"

    + plan    -> figures out what it needs to do to make the real world look like
        your desired config.  It will then tell you what it's going to do to
        make that happen.  Better be OK.

    + apply   ->  kinda obvious I guess 

when done:
    + destroy - think of it as a version of apply, that does opposite

four main commands summary:
    + refresh - TF View <-> Real World
    + plan    - Real World <-> Desired Config
    + apply   - Plan <-> Real World
    + destroy - Inverted Plan <-> Real World
    
all of the above is more about workflow (which is good!).  Now a little on
design/internals/architecture.

took a picture, see it on phone

talked about enterprise, good to know about if I get asked (has git-like
management of configs, offer private repos of terraform modules, etc


#kubernetes

## Ed V Udemy Course

### Info/Reference/Support

+ Q&A Discussion groups
+ GitHub repo (cloned on sunha)

### Sunha k8s notes

### k8s deployment options (potentially conflicting, so one per system)

+ See procedure document in intro lecture for full details

1. Local cluster on local machine, minikube

2. Production cluster via Kops on AWS

3. on-prem / cloud-agnostic via kubeadm (see end of course lecture)

4. Managed production cluster on AWS via EKS (see end of course lecture)

kops allows you to test all features and play with power,
minikube gives local control and testbed

kops is recommended by Ed V.

#### Local cluster via minikube/kubectl

+ You can see minikube config with `cat ~/.kube/config` 
+ 8443 is default k8s api server port

```
minikube start
kubectl create deployment <deployment name> ...
kubectl expose deployment <deployment name> ...
minikube service <deployment name>
minikube stop
minikube delete #only if things really broke
```

#### Production cluster via Kops on AWS
See notes in edv course dir

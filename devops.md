# docker

```
docker ps -a  #ps, but for docker
docker run <image> #workhorse command
docker login #authenticate with hub or similar
docker pull <app> #get images from the registry
docker rm <thing> #remove containers, I think, but maybe more?
docker rmi <image> #remove image, don't let these proliferate!
```

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



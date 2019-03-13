---
title: "Docker as a CI Build System"
date: 2019-03-13
tags: ["devops", "docker", "continuous integration", "ci", "jenkins"]
draft: false
---

In my current position, we define continuous integration builds for all our projects in a `Dockerfile`. For us, Docker provides a set of guarantees that are perfect for CI.

* Docker builds are reproducible.
* Docker is available on every major OS.
* Docker produces universal artifacts.

## Builds are Reproducible Everywhere

A standard Jenkins build runs directly on the system. It's nothing against Jenkins, but if you are just building on a raw node, the underlying instance will have discrepancies. This could be security patch, library versions, or even OS versions. For us, a runner might be Ubuntu or OSX. This means out of date local packages cause build failures. Don't get me wrong, that's a worst case scenario. Worst case always happens in DevOps.

The ideal state of our build is "reproducible state". A successful build that is rerun should succeed with the same output. A build that changes between runs of the same git commit and no upstream changes means someone (or an upstream) messed up. This is the beauty of Docker builds, they are immutable unless you try really hard. The steps in a `Dockerfile` run in the same order every time. Wonderful.

The biggest boon is the system that runs the build doesn't care what language or packages a build needs. The Docker build handles that via `Dockerfile` instead at the encompassing OS. There are no tricks to create virtual environments like in Python. No race conditions on which packages are installed by different builds. Everything is self contained and way faster than spinning up a fresh dedicated VM.

The glaring question one may ask, "Why not use the already built in Docker runner environments that CI solutions like Jenkins support?" Yes, that's a solution, but it doesn't solve another problem.

## Run Builds Anywhere

Running a build anywhere is a dream chased by many a developer. If we guarantee that if the build works on a dev system will work in production, that's gold. Docker is a solution to the problem. A Docker build that works on Windows will work on a Mac. Sure, it's trickery if you look too hard at the man behind the curtain (the VM running Linux). Devs usually don't care about that. When a build and tests succeed on their machine but fails on the build box, they are pissed. So why not let them run the exact same process on their machine that the Jenkins box uses?

This portability provides robust builds and increased developer productivity. Jenkins pipeline files are opaque at best so we don't expect devs to learn the syntax. DevOps doesn't want to write a new Jenkins file for every project either. Rather, we encourage each team write their project's `Dockerfile`. Once the hump of Dockerphobia is surmounted, the teams are more productive since they don't wait on DevOps for their build changes. They don't even have to trigger their Jenkins build to see if it will fail. Less waste overall.

## Docker Images as an Artifact

This is how we used to do Java builds.

1. The `cool-java-lib` repo has new code in master.
2. Jenkins builds the `cool-java-lib` artifact.
3. Jenkins pushes the `cool-java-lib` to our Nexus repository.
4. Trigger downstream builds for `cool-java-lib`.
5. Jenkins starts `alright-java-project` build.
6. Build pulls `cool-java-lib` artifact from Nexus.
7. Build of `alright-java-project` finishes.
8. Build `alright-java-project` Docker.
9. Push `alright-java-project` Docker to repository.

All of that is pretty standard. The steps do have repetition of pulling and pushing to Nexus. If use Docker for the builds, the steps change slightly.

Building the library would look like this:

1. The `cool-java-lib` repo has new code in master.
2. Jenkins builds the `cool-java-lib` Docker.
3. Push `cool-java-lib` to our Docker repository.
4. Jenkins pushes the `cool-java-lib` to our Nexus repository (used for local development).
5. Trigger downstream builds for `cool-java-lib`.

This is similar to what we did before but with a Docker build and push. Why? It's now more complicated, yet this has one major benefit over the previous solution: Docker caching. Docker caching happens when at pulls and from builds. A bunch of downstream builds happening on the same build box can all use that locally cached Docker. This is great for libraries as they don't often change in the scheme of things. On average, our build times dropped by a few minutes due to caching.

The project build:

1. Jenkins starts `alright-java-project` build.
2. Build pulls `cool-java-lib` Docker from repository (or cached version).
3. Build of `alright-java-project` using Docker finishes.
4. Push `alright-java-project` Docker to repository.

Multi-stage Docker builds allow the `alright-java-project` build to utilize the `cool-java-lib` Docker. The `Dockerfile` would probably look like the following:

```Dockerfile
FROM cool-java-lib:master as lib
FROM openjdk as builder

RUN mkdir /app
WORKDIR /app

COPY --from lib mavencache/* mavencache/*
COPY ./src/* /app/src

RUN mvn build
```

In this case, the multi-stage build is basically a glorified Maven cache. The beauty of this, though, is that all language builds support this type of artifact storage. No matter if you're building C++ or Python, this technique works. You get the benefits of tagging, and version control via a Docker repo.

## The Downsides

There are downsides to using Docker builds for CI and not mentioning them is biased. The big one is the use of Docker itself. Containers are great things, but they can be heavier than a cached library. This may not be true in some sense on the repository level since we can reuse layers. You can't do that in Maven repos. This caching requires a good knowledge of how Docker build layers work.

The intimate understanding of how Docker to builds their software is an entry barrier to developers. Dockers often seem like this magical tech that DevOps community peddles. Because of this, there is a lot more upfront work to get devs up to speed on Dockers but the tail is usually a lot better in our experience.

## Reality

We've been using Docker builds like this for a few months and it's made life a lot easier. Everyone has a much better idea of how projects build and exactly what went wrong. Is this for everyone? No, nothing is in DevOps. This works for our use case as we rely heavily on container orchestration systems and have that expertise. In the cases we are not using containers to run a system, we do have to shoehorn this build style in. This is a minority and the benefits are great for now. Like all things in DevOps, we will have a different philosophy in 6 months.

---
title: buildstrategy
authors:
  - "@sbose78"
status: implementable
---

# The BuildStrategy API


## Goals

### User-defined build strategies

Users and enterprises have strong opinions on how to build container images from source code. 
This project aims to enable admins to define build strategies for building container images in a Kubernetes cluster.

### Accomplish more by specifying less!

A slim BuildStrategy is one where the BuildStrategy author gets to accomplish more by specifying less. Without doing so,
we would be making it hard to define BuildStrategies, thereby undermining the very premise of simplicity that this project 
aims to provide with the BuildStrategy CRD. For example, the author of the build strategy should not have to specify how to push images to 
remote registries.

### Simplicity

This project uses the `TaskRun` API under-the-hood to execute an image build without leaking abstraction. The complexity of defining
a BuildStrategy must be less than that of defining a Tekton `Task`.

## Defining a BuildStrategy

A BuildStrategy CR is defined as a list of [corev1.Container](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.11/#container-v1-core)
execution steps.

### What must be specified


The author of the BuildStrategy CR may thereby specify the command(s) that need to be executed inside a container to convert source code
into a container image.

As an example, the build author might wish to express the following commands as container steps to do a [buildpacks-v3](https://github.com/buildpacks/lifecycle) image build:

* detector - chooses buildpacks (via /bin/detect)
* analyzer - restores launch layer metadata from the previous build
* builder - executes buildpacks (via /bin/build)

### What need not be specified

The Build controller should take care of executing build steps common to all strategies without making it mandatory
for users to specify 'well-known' steps. 

The BuildStrategy author need not necessarily specify:
 
* How the image is to be pushed to a registry. 
* Where the root CA certificates are to be picked up for communicating with secure registries.
* How to generate lean runtime images from the built image.
* Or, anything that can be classified as common or popular activity in an image build flow. 
 

From an implementation perspective, well-known `BuildStrategy` steps are `Tekton` `Task` steps that are dynamically generated 
on-the-fly.

### Optional overrides
 
If the BuildStrategy author wishes to be explicit about the "how" of pushing an image to registry, she should be able
express that. 

The BuildStrategy author could convey the same to the controller by annotating the `BuildStrategy`
 with:
 
 ```
 buildrun.build.dev/contains-image-push: true
 buildrun.build.dev/contains-runtime-image: true
 ```


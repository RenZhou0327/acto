# Acto: Push-Button End-to-End Testing of Kubernetes Operators/Controllers
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Regression Test](https://github.com/xlab-uiuc/acto/actions/workflows/unittest.yaml/badge.svg)](https://github.com/xlab-uiuc/acto/actions/workflows/unittest.yaml)


## Overview

Acto is a fully automatic end-to-end testing tool for Kubernetes operators/controllers. 

Acto implements a state-centric approach to test the target operator together with the managed system. 
It continuously instructs the operator to reconcile the system to different states and checks if the system reaches those desired states. 
Acto models operations as state transitions and systematically realizes state-transition sequences to exercise supported operations in different scenarios. 
Acto's automated oracles check if a system’s state is as desired. 

Acto is fully automatic. 
It only needs the operator’s deployment script as the input. 
Testing is done in a local Kubernetes environment supported by different backends: Kind, Minikube, and K3d. 
Details on Acto usage are [here](docs/port.md).

Acto has been applied to 11 popular Kuberentes, where it found 50+ new bugs 
(many are confirmed and 28 are fixed).
See [the lists of bugs](bugs.md) found by Acto.

## Prerequisites
- [Docker](https://docs.docker.com/engine/install/)
- [Golang](https://go.dev/doc/install)
- [k8s Kind cluster](https://kind.sigs.k8s.io/)  
    - `go install sigs.k8s.io/kind@v0.20.0`
- Python dependencies
    - `pip3 install -r requirements.txt`
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

[optional]
- helm
    - [Install Helm](https://helm.sh/docs/intro/install/)

## Getting started

Users need to port the operator before testing it with Acto.
We list the detailed steps of using Acto [here](docs/port.md).
We are actively working on simplifying the process to make it more user-friendly.

## Demo
To show Acto's bug finding capability, we reproduce one of the previous bugs Acto found automatically.

First, run `make` to build Acto's shared objects:
```sh
make
```

To reproduce the bug, run the following command:
```sh
python3 -m acto.reproduce --reproduce-dir test/cassop-330/trial-demo --config data/cass-operator/config.json
```
The files in the `test/cassop-330/trial-demo` directory are the sequence of CRs required to trigger
  this bug.
They are just a small subset of CRs generated by Acto automatically.

Acto first spins up a local Kubernetes cluster using Kind and deploys the cass-operator.
It then deploys CassandraDatacenter using the initial CR and 
  generates a transition to insert a key-value pair to CassandraDatacenter's property
  `spec.additionalServiceConfig.seedService.additionalLabels`.
This transition triggerred the cass-operator to add the key-value pair to `metadata.labels` of 
  the corresponding SeedService resource.
For the next step, Acto deletes the key-value pair. 
Due to a bug in cass-operator, the deleted key-value pair
  is not removed from the SeedService resource in Kubernetes.
Acto automatically detects this bug based on the inconsistency between the CR and the system resources.

## Contributing
Thank you for your interest in Acto!
We welcome all feedback and contributions.
If you wish to file a bug, enhancement proposal or have other questions,
  please use the Github [Issue](https://github.com/xlab-uiuc/acto/issues/new).
If you'd like to contribute code, please open a Pull Request.

# Kustomize

- Kustomize lets us transform YAML files representing Kubernetes resources

- The original YAML files are valid resource files

  (e.g. they can be loaded with `kubectl apply -f`)

- They are left untouched by Kustomize

- Kustomize lets us define *overlays* that extend or change the resource files

---

## Differences with Helm

- Helm Charts use placeholders `{{ like.this }}`

- Kustomize "bases" are standard Kubernetes YAML

- It is possible to use an existing set of YAML as a Kustomize base

- As a result, writing a Helm Chart is more work ...

- ... But Helm Charts are also more powerful; e.g. they can:

  - use flags to conditionally include resources or blocks

  - check if a given Kubernetes API group is supported

  - [and much more](https://helm.sh/docs/chart_template_guide/)

---

## Kustomize concepts

- Kustomize needs a `kustomization.yaml` file

- That file can be a *base* or a *variant*

- If it's a *base*:

  - it lists YAML resource files to use

- If it's a *variant* (or *overlay*):

  - it refers to (at least) one *base*

  - and some *patches*

---

## An easy way to get started with Kustomize

- We are going to use [Replicated Ship](https://www.replicated.com/ship/) to experiment with Kustomize

- The [Replicated Ship CLI](https://github.com/replicatedhq/ship/releases) has been installed on our clusters

- Replicated Ship has multiple workflows; here is what we will do:

  - initialize a Kustomize overlay from a remote GitHub repository

  - customize some values using the web UI provided by Ship

  - look at the resulting files and apply them to the cluster

---

## Getting started with Ship

- We need to run `ship init` in a new directory

- `ship init` requires an URL to a remote repository containing Kubernetes YAML

- It will clone that repository and start a web UI

- Later, it can watch that repository and/or update from it

- We will use the [jpetazzo/kubercoins](https://github.com/jpetazzo/kubeercoins) repository

  (it contains all the DockerCoins resources as YAML files)

---

## `ship init`

.exercise[

- Change to a new directory:
  ```bash
  mkdir ~/kubercoins
  cd ~/kubercoins
  ```

- Run `ship init` with the kubercoins repository:
  ```bash
  ship init https://github.com/jpetazzo/kubercoins
  ```

]

---

## Access the web UI

- `ship init` tells us to connect on `localhost:8800`

- We need to replace `localhost` with the address of our node

  (since we run on a remote machine)

- Follow the steps in the web UI, and change one parameter

  (e.g. set the number of replicas in the worker Deployment)

- Complete the web workflow, and go back to the CLI

---

## Inspect the results

- Look at the content of our directory

- `base` contains the kubercoins repository + a `kustomization.yaml` file

- `overlays/ship` contains the Kustomize overlay referencing the base + our patch(es)

- `rendered.yaml` is a YAML bundle containing the patched application

- `.ship` contains a state file used by Ship

---

## Using the results

- We can `kubectl apply -f rendered.yaml`

  (on any version of Kubernetes)

- Starting with Kubernetes 1.14, we can apply the overlay directly with:
  ```bash
  kubectl apply -k overlays/ship
  ```

- But let's not do that for now!

- We will create a new copy of DockerCoins in another namespace


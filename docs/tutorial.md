# Ksonnet.next Tutorial

* [Overview](#overview)
* [0. Installation](#installation)
* [1. Out of the box](#1-out-of-the-box)
* [2. The power of prototypes](#2-the-power-of-prototypes)
  * [Conceptual overview](#conceptual-overview)
  * [An example in practice](#an-example-in-practice)
  * [Creating Guestbook](#creating-guestbook)
* [3. Using mixin libraries](#3-using-mixin-libraries)
* [4. Managing multiple cluster environments](#4-managing-multiple-cluster-environments)
  * [Conceptual overview](#conceptual-overview-2)
  * [Initializing your environments](#initializing-environments)
  * [Upgrading your app](#upgrading-your-app)


## Overview

This tutorial provides a quick demonstration of ksonnet.next's design patterns and how you might leverage them in your own setup.

It goes through the following:

* Initializes a **version-controlled** directory to keep track of the Kubernetes app in question ([Guestbook](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/))
* Sets up a frontend/PHP deployment and Redis backend via **prototypes**
* Sets up the [EFK](http://docs.heptio.com/content/tutorials/elastic.html) logging stack via **mixin libraries**
* Creates dev and prod **environments**, allowing you to develop a new version of Guestbook "locally" and easily promote this to production

## 0. Installation

### Prerequisites
*

### Install

[INSERT KUBECONFIG (ORDER OF OPERATIONS) HERE]

> How will default environment work? Will it use URI/etc from the $KUBECONFIG at creation? What happens if $KUBECONFIG is not set?

## 1. Out of the box

Start by initializing the workspace for your application, to be called `guestbook`. The following command creates this as a new subdirectory in your current directory:
```
ksonnet init guestbook
```
Your application directory is already auto-populated with some "scaffolding" files, which you can examine as follows:
```
cd guestbook
ls -l
```
The file structure should roughly correspond to the overview below:
```
guestbook/
  .gitignore    Default .gitignore; VCS configs are ignored due to potential customization
  .ksonnet/     Metadata for ksonnet (includes libsonnet for Kubernetes API)
  environments/         Environment specs
    params.yaml Specifies the schema of config obj (don't name an environment params.yaml)
    default     The generic environment, users can add their own
  components/   Top-level Kubernetes objects defining app
  lib/          user-written files (.libsonnet helper libraries/modules)
  vendor/       3rd-party files (mixin libraries, prototypes)
```
> BELOW NOT YET IMPLEMENTED

Note that `ksonnet init` also initializes `git` source control in your application directory, so that you can keep track of any changes to your manifest files.

## 2. The power of prototypes

### Conceptual overview
Ksonnet mixins are libraries containing "parts" that can be mixed and matched to create larger components. These components can be full-fledged apps like the EFK logging stack, or basic resources like Deployments. Mixins are modular--making it easy to reuse, customize, and extend code.

Where do prototypes come into play? Though mixins provide the full spectrum of customizability, this is not always necessary for simpler use cases. The majority of these cases may be covered by some common patterns. *Prototypes* codify such patterns into preassembled collections of mixin "parts" that make it easy to get off the ground and running. **Using simple CLI commands, you can quickly go from a blank workspace to multiple complete configurations.**

More specifically, using a prototype is like "filling-in-the-blank"--you provide some input parameters and the output is a self-contained Kubernetes manifest. This manifest can always be modified by additional jsonnet code, but is fully runnable on its own.

> POTENTIALLY INSERT VISUAL AID

### An example in practice

This section describes how to use ksonnet's default Service prototype to create the Guestbook's frontend service. The best practices demonstrated in the following series of commands can be generalized to the usage of any prototype. All commands are run from the app root directory.

1. **Find** the right prototypes.

  ```
  ksonnet prototype search service
  ```

  This yields `io.ksonnet.pkg.single-port-service`.

2. **Read** through the prototype spec to determine the parameters you need.

  ```
  ksonnet prototype describe io.ksonnet.pkg.single-port-service
  ```

3. **Invoke** the prototype, filling in the appropriate parameters.

  ```
  ksonnet prototype use io.ksonnet.pkg.single-port-service \
  --name frontend \
  --targetLabelSelector "{app:'guestbook', tier:'frontend'}" \
  --servicePort 80
  --type LoadBalancer
  ```

  The resulting jsonnet file is created in the `/components` subdirectory.

  > TODOS:
  > * Note that right now `prototype use` doesn't save any files
  > * Also note that "type" (ClusterIP, NodePort, LoadBalancer) is not part of the default Service prototype (is there a reason why they aren't?)
  > * Figure out how filenames will work for generated files

4. **View** the YAML that the newly-created jsonnet component will generate.

   ```
   ksonnet show default -f components/frontend-service.jsonnet
   ```

   Note that since you have not yet created an environment for your ksonnet app, you'll need to specify the `default` environment that is auto-generated during `ksonnet init`.

5. **Validate** the jsonnet component against the Kubernetes API spec.

  ```
  ksonnet validate default -f components/frontend-service.jsonnet
  ```

  This command doesn't return any results if the jsonnet component has an valid schema.

  *(Optional)*

  You can run the following commands to see what output looks like for an invalid schema:

  ```
  cp components/frontend-service.jsonnet components/oops.jsonnet
  echo " + {fakeField: "oops"}" >> components/oops.jsonnet
  ksonnet validate default -f components/oops.jsonnet
  rm components/oops.jsonnet
  ```

6. **Dry run** the jsonnet component against your remote cluster.

  ```
  ksonnet apply default -f components/frontend-service.jsonnet --dry-run
  ```

  The resulting output, shown below, details what Kubernetes resources your component will create.
  ```
  INFO  Updating services frontend (dry-run)
  ```

7. **Version control** your change before applying to your cluster, so that you maintain a record for debuggability.
  ```
  git add components/frontend-service.jsonnet
  git commit -m "add frontend service component"
  ```

8. **Apply** the jsonnet component to your cluster.

  ```
  ksonnet apply default -f components/frontend-service.jsonnet
  ```

  You can check that the service has actually been created:

  ```
  kubectl get svc frontend
  ```

*For the sake of brevity, the following sections only refer to step (3), `ksonnet prototype use`. However, it is recommended that you use follow the full process when developing your own apps.*

### Creating Guestbook

These commands first create the full set of Guestbook resources with `ksonnet prototype use`, before applying all of them to your cluster at once.

#### (1/3) Frontend Service

The previous section covers this.

#### (2/3) Frontend Deployment

Deployments are another standard prototype that ksonnet.next provides by default. You can use it as follows:

```
ksonnet prototype use io.ksonnet.pkg.single-port-deployment \
--name redis-master \
--image redis \
--port 6379
```

#### (3/3) Backend Service + Deployment (Redis)

In addition to using standard prototypes, you can import third-party prototypes via mixins. Ksonnet has a [public mixins repo](https://github.com/ksonnet/mixins), which includes a Redis mixin. The accompanying prototype provides a full Redis implementation--deployment, service, and optional network policy.

It is loaded into the `/vendor` directory with the following command:
```
ksonnet get https://github.com/ksonnet/mixins/tree/master/incubator/redis
```
You can confirm this is the case:
```
ksonnet prototype search redis
```
This should yield at least one result.

You can use it to create the Redis component the same way you would any standard prototype:
```
ksonnet prototype use redis
```

#### Applying to your cluster

You can apply your ksonnet components all at once rather than individually. To do so, specify the environment without a specific filename:
```
ksonnet prototype apply default
```
Once all your resources are finished creating, you should be able to access your fully-functional Guestbook using the URL output below:
```
kubectl get svc frontend -o jsonpath="{.status.loadBalancer.ingress[0].hostname}"
```

## 3. Using mixin libraries

If enough customization is required that a prototype is not sufficient (or would need too many parameters), you should use mixins instead.

This is the case for your EFK (Elasticsearch-Fluentd-Kibana) logging stack. Retrieve its mixin library the same way that you did for Redis previously:
```
ksonnet get <EFK-URL>
```

Since you're not using prototypes, manually create your own `components/efk-logging.jsonnet` file with the following code:

```
<CODE>
```

Apply it to your cluster:
```
ksonnet apply default components/efk-logging.jsonnet
```


## 4. Managing Multiple Cluster Environments

### Conceptual Overview

In a real-world scenario, you are more likely to be dealing with multiple environments, as opposed to a single local Minikube cluster. Some common distinctions may include:
* **Release Management** (Dev vs. Test vs. Prod)
* **Multi-version Support** (K8s 1.6 vs 1.7)
* **Multi-AZ Setup** (us-west-2 vs. us-east-1)
* **Multi-cloud Setup** (GCP vs. AWS vs. Azure)

Even if all of your environments run the same general set of Kubernetes resources, each likely requires some degree of parameter customization. You could set this up yourself with raw jsonnet, but it is much easier to use ksonnet.next's `env` command instead. This built-in CLI support keeps your environments organized in a clear way, and also sets the stage for CI/CD integration and other potential automation.

### Initializing your environments

For your Guestbook example, you can test this out by creating `dev` and `prod` environments. For simplicity, we are going to use the same cluster for both environments, based off of the same `$KUBECONFIG`. In practice, your dev and prod are likely segregated into separate clusters.

```
export CLUSTER_URI=$(kubectl config view -o jsonpath="{.clusters[0].cluster.server}")
ksonnet env add dev $CLUSTER_URI
ksonnet env add prod $CLUSTER_URI
```

Confirm that these environments have been created successfully:

```
ksonnet env list
```
Note that each `env` is tied to a single cluster. Any command that affects your cluster state will first check that the specified `env` matches the current kubeconfig file. If not, ksonnet raises an invalid configuration error.

Environments allow you to parameterize environment-dependent properties of your manifest files. For instance, you can increase the number of replicas in prod to handle high traffic, or specify a working container image for dev.

In any particular ksonnet app, you define these parameters in a consistent schema, `environments/params.yaml`. This file defines overridable default values and applies to all environments. For the Guestbook app, use the example below:
```
<PARAMS FILE WITH FIELD FOR THE CONTAINER IMAGE IN THE FRONTEND DEPLOYMENT>
```

### Upgrading your app

> WIP

Now you are going to test a new version of your Guestbook in your dev environment. Because of ksonnet's environment support, you can do this simply by changing the `frontend_container` parameter in your `environments/dev/params.yaml`.

1. Experimenting in dev

  can use `ksonnet diff` here

2. Promoting to prod

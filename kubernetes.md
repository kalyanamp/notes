# Notes on Kubernetes

Kubernetes [[1]][KubeIntro] is an open-source system for managing containerized
applications across multiple hosts in a cluster. Kubernetes is intended to
make deploying containerized/microservice-based applications easy but powerful.

Kubernetes is considered a scheduler with the broad sense. Other container
schedulers include Swarm, Fleet and Mesos [[2]][KubeVs]. Swarm is developed by
Docker and it essentially enables the operator to use a cluster as a single
Docker host throught the `docker` CLI. Fleet is developed by CoreOS and is
providing low level cluster management. Similarly, Mesos, originally developed
at Berkeley, is a powerful and battle tested cluster management system. It
can be used to deploy Swarm or Kubernetes and it is the only one of the four
that is proven to work with thousands of nodes.

Kubernetes provides mechanisms for application deployment, scheduling,
updating, maintenance, and scaling. A key feature of Kubernetes is that it
actively manages the containers to ensure that the state of the cluster
continually matches the user's intentions.

All containers run inside **pods**. A pod is a host that runs one or more
co-located containers. A pod can have multiple **volumes**. A volume is a
directory that can be shared between some or all the containers in a pod.

Kubernetes provides all the placement and health management functionality one
would expect. When an application is deployed it looks for pods with
sufficient capacity and uses them to run the application containers. At all
times, Kubernetes monitors the running containers to ensure that the user
requests are still met. But if the pod or its machine fails, it is not
automatically moved or restarted unless the user also defines a replication
controller.

## Setting it up in AWS

At first, we need to install Kubernetes [[3]][KubeAWS]:

```bash
$> export KUBERNETES_PROVIDER=aws; curl -sS https://get.k8s.io | bash
```

This usually takes about 5-10 minutes. By default Kubernetes is deployed in
the AWS AZ of Oregon (us-west-2) and it uses 5 minions of t2.micro. These
options are configurable once the Kubernetes release is downloaded locally:

```bash
$> wget https://github.com/kubernetes/kubernetes/releases/download/v1.1.3/kubernetes.tar.gz
$> tar -zxf kubernetes.tar.gz
```

In the `kubernetes/` directory we will find `cluster/config-default.sh`. It
contains the default options. In this example, to install Kubernetes we need
to run:

```bash
$> export KUBERNETES_PROVIDER=aws
$> ./cluster/kube-up.sh
Wrote config for aws_kubernetes to /Users/george/.kube/config
Sanity checking cluster...
[...]
... calling validate-cluster
Found 4 node(s).
[...]
Done, listing cluster services:
Kubernetes master is running at https://52.34.226.255
[...]
```

To **tear-down** a Kubernetes AWS cluster, run:

```bash
$> export KUBERNETES_PROVIDER=aws
$> ./cluster/kube-down.sh
[...]
All instances deleted
Deleting VPC: vpc-03777066
Cleaning up security group: sg-dbd0c6bf
Cleaning up security group: sg-a3d0c6c7
Cleaning up security group: sg-f6dacc92
Deleting security group: sg-dbd0c6bf
Deleting security group: sg-a3d0c6c7
Deleting security group: sg-f6dacc92
Done
```

## Using Kubernetes

After Kubernetes is installed, include the `kubectl` binary in the path:

```bash
$> export PATH="$PWD"/platform/darwin/amd64:"$PATH"
```

The `kubectl` program is the Kubernetes CLI. It provides an interface to,
Kubernetes functionality. The Kubernetes release comes with various examples
to play with. I used to `examples/celery-rabbitmq` example:

```bash
cd examples/celery-rabbitmq

# Add the RabbitMQ service and replication controller
$> kubectl create -f rabbitmq-service.yml
service "rabbitmq-service" created
$> kubectl create -f rabbitmq-controller.yml
replicationcontroller "rabbitmq-controller" created

# Add the Flower WebUI service and replication controller
$> kubectl create -f flower-service.yml
service "flower-service" created
$> kubectl create -f flower-controller.yml
replicationcontroller "flower-controller" created
```

The above commands will register the services (load balancers) and start the
replication controllers. By default, the provided replication controllers
run a single replica of the RabbitMQ and Flower services. To scale this up
run:

```bash
$> kubectl scale --replicas=5 -f flower-controller.yaml
replicationcontroller "flower-controller" scaled
```

It is interesting to look into the state of the cluster. We can do so by using
the following commands:

```bash
# Quick info about the cluster
$> kubectl cluster-info
Kubernetes master is running at https://52.34.226.255
[...]

# Returns lists of pods and replication controllers
$> kubectl get pods
NAME                        READY     STATUS    RESTARTS   AGE
flower-controller-g1igq     1/1       Running   0          1m
flower-controller-ophns     1/1       Running   0          1m
flower-controller-v73co     1/1       Running   0          1m
flower-controller-vnvaw     1/1       Running   0          1m
flower-controller-zvo5u     1/1       Running   0          1m
rabbitmq-controller-f8goq   1/1       Running   0          1m
$> kubectl get rc
CONTROLLER            CONTAINER(S)   IMAGE(S)          SELECTOR             REPLICAS   AGE
flower-controller     flower         endocode/flower   component=flower     5          1m
rabbitmq-controller   rabbitmq       rabbitmq          component=rabbitmq   1          2m

# Provides detailed information about this particular replication controller
$> kubectl describe rc flower-controller
Name:           flower-controller
Namespace:      default
Image(s):       endocode/flower
Selector:       component=flower
Labels:         name=flower
Replicas:       5 current / 5 desired
Pods Status:    5 Running / 0 Waiting / 0 Succeeded / 0 Failed
No volumes.
Events:
  FirstSeen     LastSeen        Count   From                            SubobjectPath   Reason                  Message
  ─────────     ────────        ─────   ────                            ─────────────   ──────                  ───────
  1m            1m              1       {replication-controller }                       SuccessfulCreate        Created pod: flower-controller-ophns
  1m            1m              1       {replication-controller }                       SuccessfulCreate        Created pod: flower-controller-vnvaw
  1m            1m              1       {replication-controller }                       SuccessfulCreate        Created pod: flower-controller-v73co
  1m            1m              1       {replication-controller }                       SuccessfulCreate        Created pod: flower-controller-zvo5u
  1m            1m              1       {replication-controller }                       SuccessfulCreate        Created pod: flower-controller-g1igq

```

## References

1. [Kubernetes vs Swarm, Mesos and Fleet][KubeVs]
2. [Introduction to Kubernetes][KubeIntro]
3. [Deploying Kubernetes in AWS][KubeAWS]

[KubeVs]: http://radar.oreilly.com/2015/10/swarm-v-fleet-v-kubernetes-v-mesos.html
[KubeIntro]: http://kubernetes.io/v1.1/docs/user-guide/overview.html
[KubeAWS]: http://kubernetes.io/v1.1/docs/getting-started-guides/aws.html

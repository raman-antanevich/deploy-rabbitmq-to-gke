# Deploy RabbitMQ cluster to GKE (Google Kubernetes Engine)

Step by step guide for deploying RabbitMQ cluster via Kubernetes Operator to GKE


## Prerequisites

- helm 3.2.0+
- gcloud 400.0.0+
- kubectl 1.19+
- Running GKE cluster (1.19+)


## Walkthrough

### Setup kubectl context

Before you're able to deploy to GKE, you should setup kubectl context. For that you need to know project, region/zone and cluster name.

If you don't know region/zone or cluster name, you can find these parameters to list existing clusters in the project.

```sh
gcloud container clusters list \
    --project my-project
```

Location column is region or zone of the cluster. If location looks like `europe-west2-a`, it is a zonal cluster. Otherwise, it will be a regional cluster (e.g. `europe-west2`).

Use one of the following commands to setup kubectl context:

Zonal cluster

```sh
gcloud container clusters get-credentials \
    --project my-project \
    --zone europe-west2-a \
    my-cluster
```

Regional cluster

```sh
gcloud container clusters get-credentials \
    --project my-project \
    --region europe-west2 \
    my-cluster
```


### Install Cert Manager

Kubernetes communicates with operators via HTTPS. RabbitMQ operator relies on Cert Manager to generate and distribute self-signed certificate. Let's install Cert Manager from official Helm chart.

Add Jetstack chart repository

```sh
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

Deploy Cert Manager

```sh
helm upgrade cert-manager jetstack/cert-manager \
		--version v1.10.0 \
		--install \
		--atomic \
		--wait \
		--timeout 300s \
		--cleanup-on-fail \
		--namespace cert-manager \
		--create-namespace \
		--values ./chart-values/cert-manager.yml
```


### Install RabbitMQ cluster and Message Topology operators

Now we have to install Kubernetes Operators which automates provisioning, management, and operations of RabbitMQ clusters running on Kubernetes.

Add Bitnami chart repository

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Deploy RabbitMQ operators

```sh
helm upgrade rabbitmq bitnami/rabbitmq-cluster-operator \
		--version v1.10.0 \
		--install \
		--atomic \
		--wait \
		--timeout 300s \
		--cleanup-on-fail \
		--namespace rabbitmq-system \
		--create-namespace \
		--values ./chart-values/rabbitmq-operator.yml
```


### Create a RabbitMQ cluster

The latest step is to create RabbitMQ cluster. For that, we will use Helm chart that locates in `charts/rabbitmq-cluster`. Default values are `charts/rabbitmq-cluster/values.yaml`.

Create or update the cluster

```sh
helm upgrade rabbitmq-cluster ./charts/rabbitmq-cluster \
    --install \
		--atomic \
		--wait \
		--timeout 300s \
		--cleanup-on-fail \
		--namespace default \
		--create-namespace
```


## Resources

- [Using RabbitMQ Cluster Kubernetes Operator](https://www.rabbitmq.com/kubernetes/operator/using-operator.html)
- [Using RabbitMQ Messaging Topology Kubernetes Operator](https://www.rabbitmq.com/kubernetes/operator/using-topology-operator.html)

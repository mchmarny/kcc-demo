# kcc-demo

Collection of Config Connector demos on GKE. More info about KCC [here](https://cloud.google.com/config-connector/docs/how-to/getting-started).

# Assumptions

* `gcloud` installed and authenticated
* `gcloud` defaults defined (example): 

```shell
gcloud config set project YOUR_PROJECT_ID
gcloud config set compute/region us-west1
gcloud config set compute/zone us-west1-c
```

# Setup 

This will create GKE cluster (`kcc-demo`) and configure KCC on namespace `demo`:

```shell
bin/setup
```

# Demo

> Use `bin/reset` to reset demo state after previous runs 

Show available GCP resources in [UI](https://cloud.google.com/config-connector/docs/reference/overview). And by querying the cluster CRDs:

```shell
kubectl get crds --selector cnrm.cloud.google.com/managed-by-kcc=true
```

> more info on 

## PubSub

Show API description for the PubSubTopic:

```shell
kubectl describe crd pubsubtopics.pubsub.cnrm.cloud.google.com
```

Enable PubSub service:

```shell
kubectl apply -f config/enable-pubsub.yaml
```

The YAML:

```yaml
apiVersion: serviceusage.cnrm.cloud.google.com/v1beta1
kind: Service
metadata:
  name: pubsub.googleapis.com
spec:
  projectRef:
    external: projects/PROJECT_ID
```

Create PubSub Topic (`kcc-demo-topic`):

```shell
kubectl apply -f config/pubsub-topic.yaml
```

The YAML:

```yaml
apiVersion: pubsub.cnrm.cloud.google.com/v1beta1
kind: PubSubTopic
metadata:
  annotations:
    cnrm.cloud.google.com/project-id: PROJECT_ID
  name: kcc-demo-topic
```

Once created, you can describe the created resources: 

```shell
kubectl describe pubsubtopics
```

To verify that the resource is created: 

```shell
kubectl wait --for=condition=READY pubsubtopics kcc-demo-topic
```

Returns 

```shell
pubsubtopic.pubsub.cnrm.cloud.google.com/kcc-demo-topic condition met
```

# Cleanup

To delete all resources created by this demo:

```shell
bin/cleanup
```

# Disclaimer

This is my personal project and it does not represent my employer. While I do my best to ensure that everything works, I take no responsibility for issues caused by this code.
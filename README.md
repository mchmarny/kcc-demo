# kcc-demo

Collection of Config Connector demos on GKE. More info about KCC [here](https://cloud.google.com/config-connector/docs/how-to/getting-started).

> For more on declarative management of Kubernetes objects see [docs here](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/).

## Assumptions

* `gcloud` installed and authenticated
* `gcloud` defaults defined (example): 

```shell
gcloud config set project YOUR_PROJECT_ID
gcloud config set compute/region us-west1
gcloud config set compute/zone us-west1-c
```

## Setup 

This will create GKE cluster (`kcc-demo`) and configure KCC on namespace `demo`:

```shell
bin/setup
```

Verify installation:

```shell
kubectl wait -n cnrm-system --for=condition=Ready pod --all
```

It may take a minute for the resources to be available. Eventually, if everything installed correctly, you should see at least 5 `condition met` statements:

```shell
pod/cnrm-controller-manager-0 condition met
pod/cnrm-deletiondefender-0 condition met
pod/cnrm-resource-stats-recorder-******-ztwpm condition met
pod/cnrm-webhook-manager-******-***** condition met
pod/cnrm-webhook-manager-******-***** condition met
```

## Demo

> Use `bin/reset` to reset demo state after previous runs 

Show available GCP resources in [UI](https://cloud.google.com/config-connector/docs/reference/overview). And by querying the cluster CRDs:

```shell
kubectl get crds --selector cnrm.cloud.google.com/managed-by-kcc=true # | grep pubsub
```

> more info on 

### PubSub

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

Once created, you can describe the created resource: 

```shell
kubectl describe pubsubtopics
```

You can also navigate to the list of PubSub Topics in [Console](https://console.cloud.google.com/cloudpubsub/topic/list), or verify (`condition met`) that the resource is created via `kubectl`: 

```shell
kubectl wait --for=condition=READY pubsubtopics kcc-demo-topic
```

### BigQuery 

KCC is also not only for service creation, you can also manage objects in those services, see BigQuery dataset example below: 

```yaml
apiVersion: bigquery.cnrm.cloud.google.com/v1beta1
  kind: BigQueryDataset
  metadata:
    name: bigquerydatasetsample
  spec:
    defaultTableExpirationMs: 3600000
    description: "BigQuery Dataset Sample"
    friendlyName: bigquerydataset-sample
    location: US
```

## Cleanup

To delete all resources created by this demo, including the GKE cluster service account:

```shell
bin/cleanup
```

To only reset demo state use `bin/reset`.

## Disclaimer

This is my personal project and it does not represent my employer. While I do my best to ensure that everything works, I take no responsibility for issues caused by this code.
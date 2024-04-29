# OTLP Kubernetes Ingest

This project contains Kubernetes manifests for self-deployed OTLP ingest on Kubernetes.

## Running on GKE

The recommended way to add the collector to your deployment is using `kubectl
apply`. If running on a GKE Autopilot cluster (or any cluster with Workload
Identity), you must follow the prerequisite steps to set up a Workload
Identity-enabled service account below. Otherwise, you can skip to the next
section.

### Workload Identity prequisites

Follow the [Workload Identity
docs](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity#authenticating_to)
to set up an IAM service account in your GCP project with permission to use
Workload Identity and write logs, traces, and metrics:

```console
export GCLOUD_PROJECT=<your project id>
```

```console
gcloud iam service-accounts create opentelemetry-collector \
    --project=${GCLOUD_PROJECT}
gcloud projects add-iam-policy-binding ${GCLOUD_PROJECT} \
    --member "serviceAccount:opentelemetry-collector@${GCLOUD_PROJECT}.iam.gserviceaccount.com" \
    --role "roles/logging.logWriter"
gcloud projects add-iam-policy-binding ${GCLOUD_PROJECT} \
    --member "serviceAccount:opentelemetry-collector@${GCLOUD_PROJECT}.iam.gserviceaccount.com" \
    --role "roles/monitoring.metricWriter"
gcloud projects add-iam-policy-binding ${GCLOUD_PROJECT} \
    --member "serviceAccount:opentelemetry-collector@${GCLOUD_PROJECT}.iam.gserviceaccount.com" \
    --role "roles/cloudtrace.agent"
gcloud iam service-accounts add-iam-policy-binding opentelemetry-collector@${GCLOUD_PROJECT}.iam.gserviceaccount.com \
    --role roles/iam.workloadIdentityUser \
    --member "serviceAccount:${GCLOUD_PROJECT}.svc.id.goog[opentelemetry-demo/opentelemetry-collector]"
```

### Install the manifests

First, make sure you have followed the Workload Identity setup steps above.
Update [`rbac.yaml`](rbac.yaml) to annotate the Kubernetes service account with
your project and namespace:

Note: If you don't have a namespace yet you can create one and use it as current
context like this:
```console
kubectl create namespace opentelemetry-demo
kubectl config set-context --current --namespace opentelemetry-demo
```

```console
export NAMESPACE=opentelemetry-demo
sed -i "s/%GCLOUD_PROJECT%/${GCLOUD_PROJECT}/g" rbac.yaml
sed -i "s/%NAMESPACE%/${NAMESPACE}/g" rbac.yaml
```

Install the manifests:

```console
kubectl apply -n ${NAMESPACE} -f .
```

### [Optional] Run the OpenTelemetry demo application alongside the collector

To test out and see the deployment in action, you can run the demo OpenTemetry application using
```console
kubectl apply  -f ./examples/opentelemetry-demo/. -n ${NAMESPACE}
```

### See Telemetry in Google Cloud Observability

Metrics, Log and Traces should be now available in your project in Cloud Observability.
You can see metrics under "Prometheus Target" in Cloud Monitoring.

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for details.

## License

Apache 2.0; see [`LICENSE`](LICENSE) for details.

## Disclaimer

This project is not an official Google project. It is not supported by
Google and Google specifically disclaims all warranties as to its quality,
merchantability, or fitness for a particular purpose.

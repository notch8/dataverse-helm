# Demo deployment with minikube

How to create a self-contained local Dataverse deployment using minikube with an in-cluster Solr.

## Prerequisites

1. Install minikube ([official docs](https://minikube.sigs.k8s.io/docs/start) or `brew install minikube`)
2. Start your cluster: `minikube start`
3. Verify kubectl: `kubectl version`
   If kubectl is not installed, you can use minikube's bundled copy:
   ```bash
   minikube kubectl -- get po -A
   ```

## Build the Dataverse Solr image

The minikube example uses a purpose-built Solr image that has the correct Dataverse
`schema.xml`, `solrconfig.xml`, and the full Solr `_default` configset (including `lang/`,
`stopwords.txt`, etc.) baked in. This keeps the Solr conf version-locked to the Dataverse
release and requires no manual ConfigMap setup.

Once CI publishes the image to GHCR you can skip this step. Until then, build it locally
inside the minikube Docker environment so minikube can pull it without a registry:

```bash
eval $(minikube docker-env)
docker build --build-arg DATAVERSE_VERSION=6.10.1 \
    -t ghcr.io/notch8/dataverse-solr:6.10.1-noble-r0 \
    docker/solr
```

## Deploy

Preview what the chart will render:
```bash
helm template minikube-dataverse charts -f minikube-example-values.yaml
```

Install (or upgrade) Dataverse in your minikube cluster:
```bash
helm upgrade --install minikube-dataverse charts -f minikube-example-values.yaml
```

## Access Dataverse

```bash
kubectl port-forward svc/minikube-dataverse-dataverseup 8080:80
```

Then open [http://localhost:8080](http://localhost:8080).

## External Solr (production / airgapped)

For a production deployment with an external or SolrCloud instance, leave `internalSolr`
disabled and use `solrInit.mode: cloud` with `solrInit.confConfigMap` pointing to a
ConfigMap built by `scripts/k8s/ensure-solr-conf-configmap.sh`.

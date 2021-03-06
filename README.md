Leveraging the sample dotnet value app from the container builder examples, deploy the app in GKE

## Getting started
Enable the appropriate APIs
```
gcloud services enable --async \
  container.googleapis.com \
  cloudapis.googleapis.com \
  cloudbuild.googleapis.com \
  sourcerepo.googleapis.com \
  compute.googleapis.com \
  storage-component.googleapis.com \
  containerregistry.googleapis.com \
  logging.googleapis.com
```

Grant the container builder service access to the GKE API's to apply changes to the cluster later
```
PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} --format='value(projectNumber)')

gcloud projects add-iam-policy-binding ${PROJECT_NUMBER} \
  --member=serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com \
  --role=roles/container.developer
```

Create a cluster
```
gcloud container clusters create test-cluster
```

Fork this repo

Associate your forked repo with [GC Source Repositories](https://cloud.google.com/source-repositories/docs/connecting-hosted-repositories)

* Create a build trigger under [GC Container Registry](https://cloud.google.com/container-builder/docs/running-builds/automate-builds)
  * Provide a name
  * Trigger Type: `branch`
  * Build configuration: `cloudbuild.yaml`
  * Substitution Variables:
  ```
  _IMAGE_NAME = test-app
  _ZONE = us-central1-a
  _CLUSTER = test-cluster
  ```

## Deployment
```
export PROJECT_ID=<YOUR_PROJECT_ID>
```

Replace PROJECT_ID
```
sed -i "s/##PROJECT_ID##/${PROJECT_ID}/g" k8s/k8s.yaml
```

Deploy onto your kubernetes instance
``` 
kubectl apply -f k8s/k8s.yaml
```

## Force k8s to rollout new image
```
gcloud container images list-tags gcr.io/$PROJECT_ID/testapp
```
Copy the short sha tag
```
kubectl set image deployment/dn-hello-world dn-hello-wolrd=gcr.io/$PROJECT_ID/testapp:$YOUR_SHORT_SHA_TAG
```

## Cleanup
```
kubectl delete all -l app=dn-hello-world
```
# Original Quick 'n Dirty: Kubed
 - cluster and app deployment

## Table of Contents:
* [Variables](#Variables)
* [gcloud & kubectl commands (Cluster & Hello App container)](#gcloud-and-kubectl-deployment-commands)
* [Keycloak deployment Readme](Keycloak_files/Keycloak_deployment.md)

## Variables
This script is based on the following variables and commands, which can be applied directly to the terminal.
The shell variables are prefixed with (DEF) to indicate they are default variables for gcloud and kubernetes deployments, and to avoid overwriting other environment variables.
```
export DEF_ZONE="us-east1-b"
export DEF_VERSION="v1"
export DEF_APP="myapp"
export DEF_CLUSTER="${DEF_APP}-cluster"
export DEF_DEPLOYMENT="${DEF_APP}-${DEF_VERSION}-deploy"
export DEF_SERVICE="${DEF_APP}-lb-service"
export DEF_MACHINE="g1-small"
export DEF_HD_SIZE="10G"
export DEF_EXT_PORT="80"
export DEF_INT_PORT="8080"         # Dependant on container image
export DEF_REPLICAS="3"            # Number of static replicas
export DEF_AUTOSCALE_OPTIONS=''    # Insert options to enable autoscale
export DEF_CONTAINER_IMAGE='gcr.io/google-samples/hello-app:2.0'
export DEF_KC_EXT_PORT='8080'
```


## gcloud and kubectl deployment commands
#### 1-6) Check API, create cluster, deploy Kubernetes.



- Command 1)  Enable API: check if the api is disabled, and either turn on the api or confirm it was enabled.
  - Simple version: "*gcloud services enable container.googleapis.com*"

```
#### 1) Enable API
[[ -z "$(gcloud services list --enabled | grep 'container.googleapis.com')" ]] && \
    gcloud services enable container.googleapis.com || \
    echo -e "Container API already enabled."

#### 2-3) Create cluster, get credentials, if necessary.
gcloud container clusters create ${DEF_CLUSTER} --machine-type ${DEF_MACHINE} --disk-size ${DEF_HD_SIZE} --zone ${DEF_ZONE}
gcloud container clusters get-credentials ${DEF_CLUSTER} --zone ${DEF_ZONE}

#### 4-6) Create & scale deployment, create load balancer.
kubectl create deployment ${DEF_DEPLOYMENT} --image ${DEF_CONTAINER_IMAGE}
kubectl scale deployment ${DEF_DEPLOYMENT} --replicas ${DEF_REPLICAS}
kubectl expose deployment ${DEF_DEPLOYMENT} --name=${DEF_SERVICE} \
    --type=LoadBalancer --port "${DEF_EXT_PORT}" --target-port "${DEF_INT_PORT}"
```

#### 7-9) Check Hello World app on Kubernetes cluster
```
kubectl get services                                                       # Wait for external IP
LB_IP=$(kubectl get services | grep "${DEF_SERVICE}" | awk '{ print $4 }') # Fetch IP
curl -m1 http://${LB_IP}:${DEF_EXT_PORT}                                   # curl IP & port with a 1 second timeout
```

#### Optional) Deploy Keycloak app, load balancer, then route.
see: [Keycloak deployment Readme](Keycloak_files/Keycloak_deployment.md) for details.
```
kubectl apply -f keycloak_app_service.yaml                                 # deploy app and load balancer
kubectl get services                                                       # fetch IP and add it to keycloak_ingress.yaml
kubectl apply -f keycloak_ingress.yaml                                     # create ingress
```

#### 10-14) Delete Kubernetes cluster
```
kubectl delete services ${DEF_SERVICE}
kubectl scale deployment ${DEF_DEPLOYMENT} --replicas=0
kubectl delete deployment ${DEF_DEPLOYMENT}
gcloud container clusters delete ${DEF_CLUSTER} --zone ${DEF_ZONE}
```

---

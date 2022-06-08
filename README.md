# Quick n Dirty: Kubed

#### qnd_kubed quickly sets up kubernetes clusters for testing, research and learning purposes.
The script originally was part of a series of [manual](#Manual-deployment) lists of commands to copy into the shell. It was designed to be quick, but polluted the shell history with excessive commenting.

qnd_kubed without any options will:
  1. create a zonal cluster (and tell you the duration of time it took).
  2. deploy 3 Hello World containers.
  3. check the containers with the curl command
  4. offers to create a script to be used later to delete the container and the cluster, saving resources.

qnd_kubed with options will:
  - install a custom yaml script (which cancels the Hello World deployment)
  - install a second container called Keycloak
  - change cluster zone, or make a regional cluster
  - let you choose machine types
  - for more: `./qnd_kubed -h`

## Installation
*Make sure you are logged in (gcloud auth login), and you have set your project (gcloud config set project <project>)*
*If a cluster with the same name already exists, you will be asked if you want to deploy Hello World onto it.*

```
curl https://raw.githubusercontent.com/nocyber/qnd_kubed/main/qnd_kubed  > qnd_kubed   # Download
chmod +x ./qnd_kubed                                                                   # make executable
```

## Usage
```
./qnd_kubed                                   # execute with defaults.
./qnd_kubed -a "myapp" -K                     # execute, name the container app[-a], install second container (Keycloak)[-K]
./qnd_kubed -c "vmgroup" -C                   # name the cluster[-c], install only a cluster[-C] and exit
```

```
./qnd_kubed -h                                # Help screen and examples
./qnd_kubed -V                                # Show default variables
./qnd_kubed -s                                # Show status
./qnd_kubed -S                                # Show status (expanded)
```


## Usage examples with options:
```
#  Launch a zonal cluster in the default script zone, deploy Hello World (default external port 80)
#  Give the app the default script app name. (Check default variables with ./qnd_kubed -V)
./qnd_kubed -a "myapp"

#  Launch a zonal cluster, deploy Hello World (default on port 80) and call it myapp[-a].
#  Also install Keycloak container[-K] (default external port 8080).
./qnd_kubed -a "myapp" -K

#  Launch a zonal cluster, call the app[-a] "myapp", deploy Hello World on external[-e] port 1234.
#  Also install Keycloak[-K] container on external Keycloak[-k] port 5678.
#     and output the Keycloak yaml deployment files[-Y].
./qnd_kubed -a "myapp" -e 1234 -K -k 5678 -Y

#  Launch a regional cluster in us-east1, do not deploy anything.
./qnd_kubed -C -z us-east1

#  Launch a regional cluster in us-central1, call the app[-a] "myapp",
#  deploy Hello World on external[-e] port 1234.
#  Also, enable and configure[-A] autoscaling.
./qnd_kubed -z us-central1 -a "myapp" -e 2222 -A '--cpu-percent=80 --min=2 --max=6'

#  Launch a zonal cluster in zone[-z] us-west2-a, call the app[-a] "myapp",
#  tag the deployment name with version[-v] "v2", deploy 6 replicas[-r] of Hello World on external[-e] port 1234.
#  Also, and install Keycloak[-K] container and make it accessible on port 5678[-k].
./qnd_kubed -z us-west2-a -a "myapp" -v 2 -r 6 -e 1234 -K -k 5678

# Output the default Keycloak yaml deployment files only.
./qnd_kubed -W
```
*You can delete containers *
*and redeploy them on top of the same cluster with the original command you used.*
(kubectl get deployments)
(kubectl delete deployment <deplyment name>)

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[GNE GPLv2](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)

---
---
  
# Manual deployment
* [Variables](#Variables)
* [gcloud & kubectl commands (Cluster & Hello App container)](#gcloud-and-kubectl-deployment-commands)
* [yaml deployment (Keycloak container)](#deploy-Keycloak-container-manually)
* [yaml reference files for Keycloak installation](#reference-files-for-yaml-Keycloak-deployment)

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
gcloud container clusters create "${DEF_CLUSTER}" --machine-type ${DEF_MACHINE} --disk-size ${DEF_HD_SIZE} --zone ${DEF_ZONE}
gcloud container clusters get-credentials ${DEF_CLUSTER} --zone ${DEF_ZONE}

#### 4-6) Create & scale deployment, create load balancer.
kubectl create deployment ${DEF_DEPLOYMENT} --image ${DEF_CONTAINER_IMAGE}
kubectl scale deployment ${DEF_DEPLOYMENT} --replicas ${DEF_REPLICAS}
kubectl expose deployment ${DEF_DEPLOYMENT} --name=${DEF_SERVICE} \
    --type=LoadBalancer --port "${DEF_EXT_PORT}" --target-port "${DEF_INT_PORT}"
```

#### 7-9) Check Hello World app on Kubernetes cluster
```
kubectl get services                                                            # Wait for external IP
LB_IP=$(kubectl get services | grep "${DEF_SERVICE}" | awk '{ print $4 }')      # Fetch IP
curl -m1 http://${LB_IP}:${DEF_EXT_PORT}                                        # curl IP & port with a 1 second timeout
```

#### Optional) Deploy Keycloak app, load balancer, then route.
see [Keycloak yaml files for reference](#reference)
```
kubectl apply -f keycloak_app_service.yaml                                      # yaml file is inside script
kubectl apply -f keycloak_ingress.yaml                                          # generate both yaml files: ./qnd_kubed -Y
```

#### 10-14) Delete Kubernetes cluster
```
kubectl delete services ${DEF_SERVICE}
kubectl scale deployment ${DEF_DEPLOYMENT} --replicas=0
kubectl delete deployment ${DEF_DEPLOYMENT}
gcloud container clusters delete ${DEF_CLUSTER} --zone ${DEF_ZONE}
```

---


## deploy Keycloak container manually
To deploy Keycloak manually:

#### Generate yaml files and connect to cluster, if not connected:
```
./qnd_kubed -W
gcloud container clusters get-credentials ${DEF_CLUSTER} --zone ${DEF_ZONE}
```

#### Deploy app and load balancer
```
kubectl apply -f keycloak_app_service.yaml
```

- ##### Or deploy app and load-balancer with qnd_kubed:
```
./qnd_kubed -y keycloak_app_service.yaml
```

#### Configure ingress file
```
# Copy the keycloak load-balancer external IP to variable
kubectl get services
export DEF_KEYCLOAK_IP=<10.10.10.10>

# edit the file to insert IP by replacing "<LB service IP>"
sed -i 's/<LB service IP>/${DEF_KEYCLOAK_IP}/g' ./keycloak_ingress.yaml
```

#### Deploy ingress yaml.
```
kubectl apply -f keycloak_ingress.yaml
```

- #### Or deploy ingress with qnd_kubed:
```
./qnd_kubed -y keycloak_ingress.yaml
```

#### Go to website
echo "http://${DEF_KEYCLOAK_IP}:${DEF_KC_EXT_PORT}"

---

## reference files for yaml Keycloak deployment:

cat ./keycloak_app_service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: keycloak
  labels:
    app: keycloak
spec:
  ports:
  - name: http
    port: 8080
    targetPort: 8080
  selector:
    app: keycloak
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloak
  labels:
    app: keycloak
spec:
  replicas: 1
  selector:
    matchLabels:
      app: keycloak
  template:
    metadata:
      labels:
        app: keycloak
    spec:
      containers:
      - name: keycloak
        image: quay.io/keycloak/keycloak:18.0.0
        args:
        - "start-dev"
        env:
        - name: KEYCLOAK_ADMIN
          value: "admin"
        - name: KEYCLOAK_ADMIN_PASSWORD
          value: "admin"
        - name: KC_PROXY
          value: "edge"
        ports:
        - name: http
          containerPort: 8080
        readinessProbe:
          httpGet:
            path: /realms/master
            port: 8080
      initContainers:
      - name: init-myservice
        image: busybox:1.28
        command: ['sh', '-c', "echo 'This is the init container'; sleep 10"]
```


cat ./keycloak_ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: keycloak
spec:
  tls:
    - hosts:
      - keycloak.<IP>.nip.io
  rules:
  - host: keycloak.<IP>.nip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: keycloak
            port:
              number: 8080
```

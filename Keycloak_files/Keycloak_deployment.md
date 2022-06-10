# Keycloak container deployment


## Keycloak yaml reference files
* [keycloak_app_service.yaml](keycloak_app_service.yaml)
* [keycloak_ingress.yaml](keycloak_ingress.yaml)

## To deploy Keycloak:

#### Generate yaml files and connect to cluster, if not connected:
(reference files can also be found in the next [section](yaml-files-for-Keycloak-deployment))
```
./qnd_kubed -W
gcloud container clusters get-credentials ${DEF_CLUSTER} --zone ${DEF_ZONE}
```

#### Deploy app and load balancer
```
kubectl apply -f keycloak_app_service.yaml
```

- Or deploy app and load-balancer with qnd_kubed:
```
./qnd_kubed -y keycloak_app_service.yaml
```

#### Configure ingress file

Copy the keycloak load-balancer external IP to variable
```
kubectl get services
export DEF_KEYCLOAK_IP="<INSERT THE IP HERE>"
```

Check ingress file:
```
cat ./keycloak_ingress.yaml
```

Edit the file to insert IP by replacing "\<LB service IP\>",
or use this sed command to do it for you:
```
sed -i 's/<LB service IP>/${DEF_KEYCLOAK_IP}/g' ./keycloak_ingress.yaml
```

#### Deploy ingress yaml.
```
kubectl apply -f keycloak_ingress.yaml
```

- Or deploy ingress with qnd_kubed:
```
./qnd_kubed -y keycloak_ingress.yaml
```

#### Go to website
```
echo "http://${DEF_KEYCLOAK_IP}:${DEF_KC_EXT_PORT}"
```

# Quick 'n Dirty: Kubed

#### qnd_kubed quickly sets up kubernetes clusters for testing, research and learning purposes.
The script originally was part of a series of [manual](https://github.com/nocyber/qnd_kubed/blob/main/Manual_QnD_Kubed.md) lists of commands to copy into the shell. It was designed to be quick, but polluted the shell history with excessive commenting.

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

##### Download & make executable:
```
curl https://raw.githubusercontent.com/nocyber/qnd_kubed/main/qnd_kubed > qnd_kubed
chmod +x ./qnd_kubed
```

## Usage
```
./qnd_kubed                     # execute with defaults.
./qnd_kubed -a "teacup" -K      # name the app[-a] "teacup", install Keycloak[-K]
./qnd_kubed -c "qacluster" -C   # name the cluster[-c] "qacluster", install only a cluster[-C] and exit
```

```
./qnd_kubed -h                 # Help screen and examples
./qnd_kubed -V                 # Show default variables
./qnd_kubed -s                 # Show status
./qnd_kubed -S                 # Show status (expanded)
```


## Usage examples with options:
  
By default, names are based on the app name [-a <name>] to be easy to identify.
For example, if you call the app: [-a canoe], you can expect these names:
 - cluster name: "canoe-cluster"
 - deployment name: "canoe-v1-deploy"
 - load-balancer: "canoe-lb-service"

The cluster name[-c] and deployment version ids[-v] can be changed independently.

Example 1:
- Name the cluster "[default app name]-cluster".
- Launch a zonal cluster in the [default] zone, 
- Name your Hello World container: [default app name]
- Use external port: [default port 80]
- (Check default variables with ./qnd_kubed -V)

```
./qnd_kubed
```

Example 2:
- Name the cluster "test-app-cluster".
- Launch a zonal cluster in the [default] zone, 
- Name your Hello World container: [-a] "test-app"
- Use external port: [default port 80]
- Install Keycloak container also, on external port: [default 8080]

```
./qnd_kubed -a "test-app" -K
```

Example 3:
- Name the cluster "cluterz".
- Launch a zonal cluster in the [default] zone, 
- Name your Hello World container: [-a] "test-app"
- Use external port: [-e] 1234
- Install Keycloak[-K] container also, on external port: [-k] 5678
- Create Keycloak yaml[-Y] deployment files in current directory.

```
./qnd_kubed -c "cluterz" -a "test-app" -e 1234 -K -k 5678 -Y
```

Example 4:
- Name the cluster: [-c] "cluterz".
- Launch a regional cluster in region: [-z] us-east1
- Cluster only[-C], no deployments.

```
./qnd_kubed -c "cluterz" -z us-east1 -C
```

Example 5:
- Name the cluster: [-c] "test-app-cluster".
- Launch a regional cluster in region: [-z] us-central1
- Name your Hello World container: [-a] "test-app"
- Use external port: [-e] 2222
- Enable autoscaling and configure: [-A] '--cpu-percent=80 --min=2 --max=6'

```
./qnd_kubed -z us-central1 -a "test-app" -e 2222 -A '--cpu-percent=80 --min=2 --max=6'
```

Example 6:
- Name the cluster "test-app-cluster".
- Launch a zonal cluster in zone: [-z] us-west2-a
- Name your Hello World container: [-a] "test-app"
- Tag the deployment name with a different version:[-v] "v2"
- Deploy 6 replicas: [-r] 6
- Use external port: [-e] 1234
- Install Keycloak[-K] container also, on external port: [-k] 5678

```
./qnd_kubed -z us-west2-a -a "test-app" -v v2 -r 6 -e 1234 -K -k 5678
```

Example 7:
- Output the default Keycloak yaml deployment files only.

```
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

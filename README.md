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
./qnd_kubed                    # execute with defaults.
./qnd_kubed -a "myapp" -K      # execute, name the Hello World app[-a], install Keycloak[-K]
./qnd_kubed -c "vmgroup" -C    # name the cluster[-c], install only a cluster[-C] and exit
```

```
./qnd_kubed -h                 # Help screen and examples
./qnd_kubed -V                 # Show default variables
./qnd_kubed -s                 # Show status
./qnd_kubed -S                 # Show status (expanded)
```


## Usage examples with options:
```
#  Launch a zonal cluster in the default script zone, name it <app name>-cluster.
#  Give the Hello World app the default <app name>. Use default external port 80
#  (Check default variables with ./qnd_kubed -V)
./qnd_kubed

#  Launch a zonal cluster, deploy Hello World (default on port 80) and name it[-a] test-app.
#  Also install Keycloak container[-K] (default external port 8080).
./qnd_kubed -a "test-app" -K

#  Launch a zonal cluster, call the app[-a] "test-app", deploy Hello World on external[-e] port 1234.
#  Also install Keycloak[-K] container on external Keycloak[-k] port 5678.
#     and output the Keycloak yaml deployment files[-Y] to the current directory.
./qnd_kubed -a "test-app" -e 1234 -K -k 5678 -Y

#  Launch a regional cluster in us-east1, do not deploy anything.
./qnd_kubed -C -z us-east1

#  Launch a regional cluster in us-central1, call the app[-a] "test-app",
#  deploy Hello World on external[-e] port 2222.
#  Also, enable and configure autoscaling[-A].
./qnd_kubed -z us-central1 -a "test-app" -e 2222 -A '--cpu-percent=80 --min=2 --max=6'

#  Launch a zonal cluster in zone[-z] us-west2-a, call the app[-a] "test-app",
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

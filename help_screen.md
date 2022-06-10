
## << qnd_kubed: Creating a Kubernetes cluster >>

## OPTIONS:
```
-h                                      # help: this screen
-V                                      # Show default variables
-s                                      # Display Current Cluster Status
-S                                      # Display Current Cluster Status, expanded
-F                                      # Fast install, skip status screens and curl check.

# Cluster:
-c <cluster name>                       # Give a custom name to the cluster
-z <zone or region>                     # Override default zone, or make a regional cluster by selecting a region
-C                                      # Create a (zonal) cluster only
-C -z <region>                          # Create a (regional) cluster only
-Z <cluster name>                       # "Get Credentials" for an existing cluster and exit
-Z <cluster name> -z <zone or region>   #   - Specify the zone or region if it is not the script default zone
-t <MACHINE TYPE> ex: "g1-small"        # Override default Machine type varible
-d <BOOT DISK SIZE   ex. "10GB">        # Override default boot disk size varible

# Hello World container: (installed by default)
-a <APPLICATION NAME>                   # Override default Application name varible
-v <VERSION>                            # Override default Version varible
-e <external port>                      # Change default external-facing port (80)
-r <static replica number>              # Set custom static scaling, the number of replicas (pods)
-A '--cpu-percent=80 --min=2 --max=6'   # Enable autoscale by inserting options
-H                                      # Do not install Hello World Container
-o                                      # Do not install check the app with curl

# Keycloak container:
-K                                      # Install Keycloak container (user: admin, password: admin)
-k <external port>                      # Change default keycloak external-facing port (8080)
-Y                                      # Generate Keycloak yaml files
-W                                      # Generate Keycloak yaml files and show status only
                                        #   (no cluster creation or Hello World deployment.)

# Deploy custom yaml file:
-y <yaml file>                          # deploy custom yaml file, do not deploy Hello World
```

## EXAMPLES:
```
#  Launch a zonal cluster in the default script zone, deploy Hello World (default external port 80)
#  Give the app the default script app name. (Check default variables with ./qnd_kubed -V)
./qnd_kubed -a "myapp" -K

#  Lqaunch a zonal cluster, deploy Hello World (default on port 80) and call it myapp[-a].
#  Also install Keycloak container[-K] (default external port 8080).
./qnd_kubed -a "myapp" -K

#  Launch a zonal cluster, call the app[-a] "myapp", deploy Hello World on external[-e] port 1234.
#  Also install Keycloak[-K] container on external Keycloak[-k] port 5678.
#     and output the Keycloak yaml deployment files.
./qnd_kubed -a "myapp" -e 1234 -K -k 5678 -Y

#  Launch a regional cluster in us-east1, do not deploy anything.
./qnd_kubed -C -z us-east1

#  Launch a regional cluster in us-central1, call the app[-a] "myapp",
#  deploy Hello World on external[-e] port 1234.
#  Also, enable and configure[-A] autoscaling.
./qnd_kubed -z us-central1 -a "myapp" -e 2222 -A '--cpu-percent=80 --min=2 --max=6'

#  Launch a zonal cluster in zone[-z] us-east1-b, call the app[-a] "myapp",
#  tag the deployment name with version[-v] "v2", deploy 6 replicas[-r] of Hello World on external[-e] port 1234.
#  Also, and install Keycloak[-K] container and make it accessible on port 5678[-k].
./qnd_kubed -z us-east1-b -a "myapp" -v 2 -r 6 -e 1234 -K -k 5678

# Output the default Keycloak yaml deployment files.
./qnd_kubed -W
```

# Docker-OpenWISP

[![Build Status](https://travis-ci.org/openwisp/docker-openwisp.svg?branch=master)](https://travis-ci.org/openwisp/docker-openwisp)
[![Gitter](https://img.shields.io/gitter/room/openwisp/general.svg)](https://gitter.im/openwisp/general)
[![Docker Hub](https://img.shields.io/badge/docker--hub-openwisp-blue.svg)](https://hub.docker.com/u/openwisp)
[![GitHub license](https://img.shields.io/github/license/atb00ker/docker-openwisp.svg)](https://github.com/atb00ker/docker-openwisp/blob/master/LICENSE)

This repository contains Official docker images of OpenWISP. Designed with horizontal scaling, easily replicable deployments and user customization in mind.

![kubernetes](https://i.ibb.co/rGpLq4y/ss1.png)
The sample files for deployment on kubernetes are available in the `kubernetes/` directory.

## TL;DR

Images are available on docker hub and can be pulled from the following links:
- OpenWISP Base - `openwisp/openwisp-base:latest`
- OpenWISP Dashboard - `openwisp/openwisp-dashboard:latest`
- OpenWISP Radius - `openwisp/openwisp-radius:latest`
- OpenWISP Controller - `openwisp/openwisp-controller:latest`
- OpenWISP Network Topology - `openwisp/openwisp-topology:latest`
- OpenWISP Nginx - `openwisp/openwisp-nginx:latest`

The configurations for openwisp images is available [here](https://bit.ly/2Wos8lP).

## Deployment

1. [Kubernetes](https://github.com/atb00ker/dockerize-openwisp#kubernetes)
2. [Docker Compose](https://github.com/atb00ker/dockerize-openwisp#docker-compose)

### Kubernetes

The following are steps of a sample deployment on a kubernetes cluster. All the files are present in `kubernetes/` directory of this repository.
The following assumes the reader knows basics of kubernetes.

1. (optional) Setup a Kubernetes Cluster: A guide for setting up the cluster on bare-metal machines is available [here](https://blog.alexellis.io/kubernetes-in-10-minutes/) and the guide to get started with kubernetes-dashboard (Web UI) is available [here](https://github.com/kubernetes/dashboard).

2. Changes for your cluster:

   2.1. `externalIPs` in `Service.yml` should to be your cluster's `externalIPs`

   2.2 `ingress` in `Ingress.yml` should to be your cluster's loadbalancer IPs.

   2.3 `<SERVICE-NAME>_DOMAIN` variables in `ConfigMap.yml` should to be your domain names.

3. If you are doing bare-metal setup, follow the steps below to setup nfs-provisioner. If you are using a provider like GKE or Amazon EKS your provider may have this ready out-of-the-box (If your provider doesn't provide it and you can't make these changes you need to alter the `PersistentVolumeClaim.yml` file):

   3.1. Install NFS requirements: `sudo apt install nfs-kernel-server nfs-common`

   3.2. Setup storage directory:

   ```bash
   sudo mkdir -p /mnt/kubes
   sudo chown nobody: /mnt/kubes
   ```

   3.3. Export the directory file system - inside the `/etc/exports` file add line: `/mnt/kubes    *(rw,sync,no_root_squash,no_subtree_check,no_all_squash,insecure)` and then export `sudo exportfs -rav`

4. [Setup helm](https://helm.sh/docs/using_helm/) and install the requirement(s):

   4.1. NFS Provisioner: `helm install --set storageClass.name=nfs-provisioner --set nfs.server=<ip-address> --set nfs.path=/mnt/kubes stable/nfs-client-provisioner`

   4.2. [Setup Cert-Manager](https://docs.cert-manager.io/en/latest/getting-started/install/) to take care of SSL certificates.

5. (optional) Customization: You can change any of the variables from the [list here](https://bit.ly/2Wos8lP) to trailer to your requirements. You need to change the values in `ConfigMap.yml`.

   - The ConfigMap with name `postgres-config` will pass the environment variables only to the postgresql container.
   - The ConfigMap with name `common-config` will pass the environment variables to all the openwisp containers where the values are applicable except the postgres instances.

6. Apply to Kubernetes Cluster: You need to apply all the files in the `kubernetes/` directory to your cluster. Some `ReplicationControllers` are dependant on other components, so it'll be helpful to apply them at last. This is the recommended order:

```bash
kubectl apply -f ConfigMap.yml
kubectl apply -f ClusterIssuer.yml
kubectl apply -f PersistentVolumeClaim.yml
kubectl apply -f Service.yml
kubectl apply -f Ingress.yml
kubectl apply -f ReplicationController.yml
```

NOTE: Containers will take a little while to start their work. You can see the status on the Web UI or on CLI by `kubectl get all` command.

### Docker Compose

Testing on docker-compose is relatively less resource and time consuming.

1. Install docker-compose: `pip install docker-compose`

2. Change the following options according to your system: `DJANGO_SECRET_KEY`, `DB_USER`, `DB_PASS`, `DJANGO_DEF_EMAIL`, `DASHBOARD_DOMAIN`, `CONTROLLER_DOMAIN`, `RADIUS_DOMAIN`, `TOPOLOGY_DOMAIN`. Optionally, you may change any other setting as well. A detailed document of the available variables can be found [here](https://bit.ly/2Wos8lP).

3. Pull all the required images to avoid building them. (Building images is a time consuming task.)

```bash
docker pull openwisp/openwisp-base:latest
docker pull openwisp/openwisp-dashboard:latest
docker pull openwisp/openwisp-radius:latest
docker pull openwisp/openwisp-controller:latest
docker pull openwisp/openwisp-topology:latest
docker pull openwisp/openwisp-nginx:latest
```

4. Run containers: Inside root of the repository, run `docker-compose up`. It will take a while for the containers to start up. (~1 minute)

5. When the containers are ready, you can test them out by going to the domain name that you've set for the modules.

Note:
   - Default username & password are `admin`.

**Note:** If you are using pipenv, remember that changing the values in `.env` file does nothing because `.env` is also a special file in `pipenv`, you need to change the values in `.env` file then re-activate environment to ensure that the changes reflect.

## Build (Development)

Guide to build images again with modification or with different environment variables.

1. Install docker-compose.
2. In the root of the repository, run `make`.
3. Run `docker-compose up`, when the containers are ready, you can test them out by going to the domain name of the modules.

Now you'll need to do steps (2) and (3) everytime you make a changes and want to build the images again. Consider using `docker-compose up -d` that may save you sometime if you need to build a container again and again.

**Note:**
   - Default username & password are `admin`.
   - Default domains are: dashboard.openwisp.org, controller.openwisp.org, radius.openwisp.org and topology.openwisp.org.
   - You may want to add the domains in your hosts file, command: `echo "127.0.0.1 dashboard.openwisp.org controller.openwisp.org radius.openwisp.org topology.openwisp.org" >> /etc/hosts/`
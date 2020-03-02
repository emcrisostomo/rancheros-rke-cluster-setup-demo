README
======

* Make sure `kubectl` is available, otherwise install it first.
* Create a virtual machine running RancherOS using the following command:

```shell script
$ docker-machine-create-rancher-vm \
    --name rancheros-rke-00 \
    --cpu-count 2 \
    --ram-size 4096
Running pre-create checks...
Creating machine...
(rancheros-rke-00) Downloading /Users/enrico/.docker/machine/cache/boot2docker.iso from /Users/enrico/.docker/machine/cache/rancheros-vmware-autoformat.iso...
(rancheros-rke-00) Creating SSH key...
(rancheros-rke-00) Creating VM...
(rancheros-rke-00) Creating disk '/Users/enrico/.docker/machine/machines/rancheros-rke-00/rancheros-rke-00.vmdk'
(rancheros-rke-00) Virtual disk creation successful.
(rancheros-rke-00) Starting rancheros-rke-00...
(rancheros-rke-00) Waiting for VM to come online...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with rancheros...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env rancheros-rke-00
```

* Check the status of the machine with `docker-machine ls`:

```shell script
$ docker-machine ls
NAME               ACTIVE   DRIVER         STATE     URL                        SWARM   DOCKER     ERRORS
rancheros-rke-00   -        vmwarefusion   Running   tcp://172.16.38.131:2376           v19.03.5
```

* Configure the current shell to connect the Docker client to the newly created
  machine:

```shell script
$ eval $(docker-machine env rancheros-rke-00)
```

* Run `rke` to create the initial Kubernetes cluster configuration file,
  specifying `docker` as SSH user (RancherOS uses `rancher` by default, but
  `docker-machine` overrides this setting) and
  `~/.docker/machine/machines/rancheros-rke-00/id-rsa` as cluster level SSH
  private key path.  Since this is a single node cluster, we also have to elect
  this node as control plane, worker and etcd host:

```
$ rke config
[+] Cluster Level SSH Private Key Path [~/.ssh/id_rsa]: ~/.docker/machine/machines/rancheros-rke-00/id_rsa
[+] Number of Hosts [1]:
[+] SSH Address of host (1) [none]: 172.16.38.131
[+] SSH Port of host (1) [22]:
[+] SSH Private Key Path of host (172.16.38.131) [none]:
[-] You have entered empty SSH key path, trying fetch from SSH key parameter
[+] SSH Private Key of host (172.16.38.131) [none]:
[-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.docker/machine/machines/rancheros-rke-00/id_rsa
[+] SSH User of host (172.16.38.131) [ubuntu]: docker
[+] Is host (172.16.38.131) a Control Plane host (y/n)? [y]: y
[+] Is host (172.16.38.131) a Worker host (y/n)? [n]: y
[+] Is host (172.16.38.131) an etcd host (y/n)? [n]: y
[+] Override Hostname of host (172.16.38.131) [none]:
[+] Internal IP of host (172.16.38.131) [none]:
[+] Docker socket path on host (172.16.38.131) [/var/run/docker.sock]:
[+] Network Plugin Type (flannel, calico, weave, canal) [canal]:
[+] Authentication Strategy [x509]:
[+] Authorization Mode (rbac, none) [rbac]:
[+] Kubernetes Docker image [rancher/hyperkube:v1.17.2-rancher1]:
[+] Cluster domain [cluster.local]:
[+] Service Cluster IP Range [10.43.0.0/16]:
[+] Enable PodSecurityPolicy [n]:
[+] Cluster Network CIDR [10.42.0.0/16]:
[+] Cluster DNS Service IP [10.43.0.10]:
[+] Add addon manifest URLs or YAML files [no]:
```

* Inspect the `cluster.yml` configuration file that has been created by `rke`.

* Run the following command to create the Kubernetes cluster in the same
  directory where the `cluster.yml` is located, or manually specify a location
  using the `--config` option:

```shell script
$ rke up --config path/to/cluster.yml
INFO[0000] Running RKE version: v1.0.4
INFO[0000] Initiating Kubernetes cluster
INFO[0000] [dialer] Setup tunnel for host [172.16.38.131]
INFO[0000] Checking if container [cluster-state-deployer] is running on host [172.16.38.131], try #1
INFO[0000] Image [rancher/rke-tools:v0.1.52] exists on host [172.16.38.131]
INFO[0000] Starting container [cluster-state-deployer] on host [172.16.38.131], try #1
INFO[0000] [state] Successfully started [cluster-state-deployer] container on host [172.16.38.131]
INFO[0000] [certificates] Generating CA kubernetes certificates
INFO[0000] [certificates] Generating Kubernetes API server aggregation layer requestheader client CA certificates
INFO[0000] [certificates] Generating Kubernetes API server certificates
INFO[0001] [certificates] Generating Service account token key
INFO[0001] [certificates] Generating Kube Controller certificates
INFO[0001] [certificates] Generating Kube Scheduler certificates
INFO[0001] [certificates] Generating Kube Proxy certificates
INFO[0001] [certificates] Generating Node certificate
INFO[0001] [certificates] Generating admin certificates and kubeconfig
INFO[0001] [certificates] Generating Kubernetes API server proxy client certificates
INFO[0001] [certificates] Generating kube-etcd-172-16-38-131 certificate and key
INFO[0001] Successfully Deployed state file at [./cluster.rkestate]
INFO[0001] Building Kubernetes cluster
...
INFO[0096] [ingress] ingress controller nginx deployed successfully
INFO[0096] [addons] Setting up user addons
INFO[0096] [addons] no user addons defined
INFO[0096] Finished building Kubernetes cluster successfully
```

* Use `kubectl` to operate on the cluster, specifying the kubeconfig file using
  the `--kubeconfig` option:

  ```shell script
  kubectl --kubeconfig ./kube_config_cluster.yml get all
  NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
  service/kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   6m35s
  ```

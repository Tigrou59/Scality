**Install minikube for a cluster with only one node (master and node)**

*Installation for minikube with option "none dirver" on Linux VM*

https://github.com/robertluwang/docker-hands-on-guide/blob/master/minikube-none-installation.md

*Verify the installation*
`minikube status`
>minikube: Running
cluster: Running
kubectl: Correctly Configured: pointing to minikube-vm at 10.0.2.15

`kubectl cluster-info`
>Kubernetes master is running at https://10.0.2.15:8443
KubeDNS is running at https://10.0.2.15:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

>To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

 `kubectl get pods --all-namespaces`

  NAMESPACE     NAME                                    READY   STATUS             RESTARTS   AGE
- kube-system   coredns-c4cffd6dc-slx45                 1/1     Running            2          5d
- kube-system   etcd-minikube                           1/1     Running            7          8d
- kube-system   heapster-p2f9x                          1/1     Running            6          5d
- kube-system   influxdb-grafana-kspgm                  2/2     Running            12         5d
- kube-system   kube-addon-manager-minikube             1/1     Running            7          8d
- kube-system   kube-apiserver-minikube                 1/1     Running            7          8d
- kube-system   kube-controller-manager-minikube        1/1     Running            7          8d
- kube-system   kube-dns-79f5cdddc5-79sht               3/3     Running            41         2d
- kube-system   kube-proxy-smk6n                        1/1     Running            7          8d
- kube-system   kube-scheduler-minikube                 1/1     Running            8          8d
- kube-system   kubernetes-dashboard-6f4cfc5d87-rs6dk   1/1     Running            13         8d
- kube-system   storage-provisioner                     1/1     Running            14         8d

*Verify that the ADDON dashboard is enabled and set the ADDON heapster at enabled, then start the k8s console*

  `minikube addons enable heapster`

  `minikube addons list`

- addon-manager: enabled
- coredns: enabled
- dashboard: enabled
- default-storageclass: enabled
- efk: disabled
- freshpod: disabled
- heapster: enabled
- ingress: disabled
- kube-dns: enabled
- metrics-server: disabled
- nvidia-driver-installer: disabled
- nvidia-gpu-device-plugin: disabled
- registry: disabled
- registry-creds: disabled
- storage-provisioner: enabled

*Start the k8s console*

  `minikube dashboard`

**In Dev environnment with k8s in cluster minikube**

*In production, we expect a cluster n nodes (3 nodes minimum) installed with kubeadm or provided by a cloud provider
we also expect that data will be persistent.
That you will use the multiple backends capabilities of Zenko CloudServer, and that you will have a custom endpoint for your local storage, and custom credentials for your local storage:*

*Many object k8s will be created, in logical order :*
**ConfigMap that stores ENV variables**

  `kubectl apply -f configmap-s3server.yml`

  `kubectl get cm -o wide`

  NAME            DATA   AGE
- config-map-s3   3      1d

**A Persistent Volumes for storage local or on any other types of storage suported by kubernetes (Volume Plugin)**

  `kubectl apply -f pv-s3server.yml`

  `kubectl get pv -o wide`

  NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                     STORAGECLASS   REASON   AGE
- s3-pv    1000Mi     RWO,ROX        Recycle          Bound    default/s3server-claim                            3d

**PersistentVolumeClaim for binding the PersistentVolume and then use refered this in pods**

  `kubectl apply -f pvc-s3server.yml`

  `kubectl get pvc -o wide`

  NAME              STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
- s3server-claim    Bound    s3-pv    1000Mi     RWO,ROX                       3d

*At the hight level, the Deployment wich including replicas, pods, labels, environment config, strategy of rolling update, claim for pvc...*

  `kubectl apply -f deploy-s3server-pv.yml --record`

  `kubectl get deploy -o wide`

  NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                    SELECTOR
- s3server          3         3         3            3           1d    s3server     scality/s3server:latest   app=s3server

 `kubectl get pods -l app=s3server -o wide`

  NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE
- s3server-6ffc956c55-hp5cc   1/1     Running   1          1d    172.17.0.11   minikube
- s3server-6ffc956c55-llld7   1/1     Running   1          1d    172.17.0.10   minikube
- s3server-6ffc956c55-xhxbm   1/1     Running   1          1d    172.17.0.9    minikube

*And finally the Service "NodeType" bring a stable and reliable network, DNS, load-balancing and expose your app outside of the kubernetes cluster, Cluster IP and Port (in our case : 10.109.72.126:30042 over TCP)*

>DNS-Based Service Recovery requires the DNS cluster--add-on,
The DNS add-on implements a POD-based DNS service in the cluster and configures all kubelets (nodes) to use it for DNS
ie: minikube addons list

  `kubectl get svc -l app=s3server -o wide`

  NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
- s3server-svc   NodePort   10.109.72.126   <none>        8000:30042/TCP   5d    app=s3server

> Behind the scene

**Public Docker repository for cloudserver Scality Zenko**
  * https://hub.docker.com/r/scality/s3server/

**Public GitHub source repo**
  * https://github.com/scality/cloudserver

**Zenko Cloud Server**
  * https://www.zenko.io/cloudserver/

**Install minikube on a VM**
  * https://github.com/robertluwang/docker-hands-on-guide/blob/master/minikube-none-installation.md

**Kubernetes documentation**
  * https://kubernetes.io/docs/home/?path=users&persona=app-developer&level=foundational

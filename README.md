# Installation Visual Flow to Local Minikube


1. [Installation prerequisites](#prerequisites)
    - [Setting up minikube cluster](#settingupcluster)
    - [Install Redis and PostgreSQL](#settingupadditionalsw) 
2. [Install Visual Flow application](#installvfapp)


## <a id="prerequisites">Installation prerequisites</a>

To install Visual Flow you should have software installed below. Please use official documentation to perform prerequisite installation.

- Git ([install](https://git-scm.com/downloads))
- kubectl ([install](https://kubernetes.io/docs/tasks/tools/))
- Helm CLI ([install](https://helm.sh/docs/intro/install/))
- Minikube ([install](https://minikube.sigs.k8s.io/docs/start/))

In case if you going pull\push images from AWS ECR - you need also install few AWS tools:
- AWS CLI ([install](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html))
- eksctl ([install](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html))

And if you have just installed the AWS CLI, then you need to perform initial configuration of AWS  command line interface using following command:

```bash
aws configure
```

> [!NOTE]
> The Visual Flow application has no formal hardware requirements, but Spark itself requires 4 CPUs and 6 GB of RAM to run at least one worker-pod.



## <a id="settingupcluster">Create minikube cluster</a>
>[!NOTE]
> 1. In this example we will use the HyperV VM driver, which is recommended for the Windows OS family. But depending on your system or requirements - you can also use Docker, VirtualBox, Podman, KVM2 etc.
> 2. The Visual Flow application has no formal hardware requirements, but Spark itself requires minimum 4 CPUs and 6 GB of RAM to run at least one worker-pod.


~~Also kubernetes version is 1.25.4, since current latest one (1.27.2) caused problem with GitOAuth Authentification (you may get issue like 'Failed to obtain access token'). So at least on this version with HyperV driver app was tested and works without any problem.~~

Now you can create simple cluster with Minikube using following commands:

```bash
minikube start --cpus 4 --memory 6g --disk-size 20g --delete-on-failure=true --driver hyperv --kubernetes-version=v1.25.4 -p visual-flow 
```
The cluster creatation duration about ~5-10min so please be patient.   
In case cluster creation failed you can delete cluster using following command and repeat cluster creation from beginning.
```bash
minikube delete -p visual-flow
```

Once cluster is ready you can change default profile to created cluster with command below. Setting up `visual-flow` as default minikube profile allow you to start and stop minikube without profile name.

```bash
minikube profile visual-flow
```
To check running pods of created cluster please use following command.
```bash
kubectl get pods -A
```

Olso please check the cluster IP address which is used to access Visual Flow application and other related services running on the same cluster.  
```bash
minikube ip
```


>[!TIP]
> Additional information about Minikube can be found by following guide: <https://minikube.sigs.k8s.io/docs/start>




## <a id="settingupadditionalsw">Install Redis & PostgreSQL (optional if need)</a>


Some functionality of Visual Flow application requires to have Redis & PosgreSQL dbs. Both of them with custom and default configs included in installation as a separate helm charts (values files with source from bitnami repo). 

<https://github.com/ibagroup-eu/Visual-Flow-deploy/tree/minikube/charts/dbs>

You can get them and install on you cluster using following steps.

#### 1. Add 'bitnami' repository to helm repo list
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
#### 2. Clone Visual Flow repo and navigate to Visual-Flow-deploy/charts/dbs directory
```bash
git clone -b minikube https://github.com/ibagroup-eu/Visual-Flow-deploy.git Visual-Flow-deploy

cd Visual-Flow-deploy/charts/dbs
```
#### 3. Redis (for Session and Job's execution history)
Using helm tool install `Redis` into the `visual-flow` cluster:
```bash
helm install redis -f bitnami-redis/values.yaml bitnami/redis
```
#### 4. PostgreSQL (History service)
Using helm tool install `PostgreSQL` into the `visual-flow` cluster:
```bash
helm install pgserver -f bitnami-postgresql/values.yaml bitnami/postgresql
```
To check that both services Ready and Running use following kubectl command.
```bash
> kubectl get pods
```
The command output shows you running pods with installed software.
`NAME                    READY   STATUS    RESTARTS   AGE
pgserver-postgresql-0   1/1     Running   0          2m59s
redis-master-0          1/1     Running   0          3m23s`


FYI: Just in case better to save output of these command (it contains helpful info with short guide how to get access to pod & dbs and show default credentials).



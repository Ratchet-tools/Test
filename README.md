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
>In this example we will use the HyperV VM driver, which is recommended for the Windows OS family. But depending on your system or requirements - you can also use Docker, VirtualBox, Podman, KVM2 etc.

*Also kubernetes version is 1.25.4, since current latest one (1.27.2) caused problem with GitOAuth Authentification (you may get issue like 'Failed to obtain access token'). So at least on this version with HyperV driver app was tested and works without any problem.*

You can create simple cluster in Minikube using following commands:

```bash
minikube start --cpus 4 --memory 6g --disk-size 20g --delete-on-failure=true --driver hyperv --kubernetes-version=v1.25.4 -p visual-flow 

# duration: ~5-10min

# if creation failed - delete cluster using following command and repeat from beginning:

#> minikube delete -p visual-flow
```

When cluster is ready - you can switch default profile to this cluster, check running pods and cluster IP:
```bash
minikube profile visual-flow

kubectl get pods -A

minikube ip
# on this IP will be available VF application and other services
```


For additional info about Minikube check following guide: 
<https://minikube.sigs.k8s.io/docs/start>


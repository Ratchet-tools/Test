# Installation Visual Flow to Local Minikube


1. [Installation prerequisites](#prerequisites)  
2. [Setting up minikube cluster](#settingupcluster)


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

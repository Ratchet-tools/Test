# Installation Visual Flow to Local Minikube


1. [Prerequisite Installation](#prerequisites)
    - [Setting up prerequisite tools](#prereqtools)
    - [Clone Visual Flow repository](#clonevfrepo)
    - [Setting up minikube cluster](#settingupcluster)
    - [Install Redis and PostgreSQL](#settingupadditionalsw) 
2. [Installation of Visual Flow application](#installvfapp)
    - [Update installation configuration file (values.yaml)](#valuescfg)
    - [Configure GitHub OAuth](#oauthsetup)
4. [Use Visual Flow](#usevf)
5. [Delete Visual FLow](#uninstallvf)
6. [Tips and Tricks](#tipsandtricks)


## <a id="prerequisites">Prerequisite Installation</a>

To install Visual Flow you should have software installed below. Please use official documentation to perform prerequisite installation.


## <a id="prereqtools">Setting up prerequisite tools</a>

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

## <a id="clonevfrepo">Clone Visual Flow repository</a>

  All instructions in this manual should be running in OS shell (either Windows command line or Linux bash/sh/ksh shells).
  Create <a id="workdir">__working directory__</a> on your target server where installation scripts will be downloaded to. The Visual Flow application will be installed on the same machine. 
  Once working directory created using OS shell commands navigate to it and perform Clone (or download) the [Minikube branch from Visual-Flow-deploy repository](https://github.com/ibagroup-eu/Visual-Flow-deploy/tree/minikube) repository using following command:

```bash
git clone -b minikube https://github.com/ibagroup-eu/Visual-Flow-deploy.git Visual-Flow-deploy
```
This `Visual-Flow-deploy` directory will be used in steps during application installation below. 


## <a id="settingupcluster">Create minikube cluster</a>
>[!NOTE]
> 1. In this example we will use the HyperV VM driver, which is recommended for the Windows OS family. But depending on your system or requirements - you can also use Docker, VirtualBox, Podman, KVM2 etc.
> 2. By default VF projects require 4 CPU and 6GB of RAM. We do not recommend to reduce these settings (to avoid job run issues), but you may increase these values based on your workload.


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

Olso please check the <a id="clusterIP">__cluster IP address__</a> which is used to access Visual Flow application and other related services running on the same cluster.  
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
#### 2. Navigate to your [working directory](#workdir).
#### 3. Navigate to Visual-Flow-deploy/charts/dbs directory

Go to the "[dbs](https://github.com/ibagroup-eu/Visual-Flow-deploy/blob/minikube/charts/dbs)" directory of the downloaded 
"[Visual-Flow-Deploy](#clonevfrepo)" repository with the following command:
    
```bash
cd Visual-Flow-deploy/charts/dbs
```
#### 4. Redis (for Session and Job's execution history)
Use helm tool to install `Redis` database service into the `visual-flow` cluster:
```bash
helm install redis -f bitnami-redis/values.yaml bitnami/redis
```
#### 5. PostgreSQL (History service)
Use helm tool to install `PostgreSQL` database server into the `visual-flow` cluster:
```bash
helm install pgserver -f bitnami-postgresql/values.yaml bitnami/postgresql
```
To check that both services Ready and Running use following kubectl command.
```bash
kubectl get pods
```
The command output shows you running pods with installed software.
```bash
NAME                    READY   STATUS    RESTARTS   AGE
pgserver-postgresql-0   1/1     Running   0          2m59s
redis-master-0          1/1     Running   0          3m23s
```

>[!TIP]
> It is recommended to save output of the installation commands which contains helpful info with short guide how to get access to pod & dbs and show default credentials.



## <a id="installvfapp">Installation of Visual Flow application</a>

### <a id="valuescfg">Update installation configuration file (values.yaml)</a>
VisualFlow installation process require some configuration values to be adjusted during some installation steps.
Configuration settings are located in `helm` configuration file called [values.yaml](./charts/visual-flow/values.yaml).
`values.yaml` contains following major configuration values you may need adjust during setting up process:


<table>
<tr>
        <th> Variable name </th>  <th> Type </th> <th>Value in template</th> <th> Note </th>
</tr>
   <tr>
   <td> install: </td>  <td> Bool </td> 
   <td> 

`install: true`

   </td> 
   <td>

The values.yam file contains several sections and they are representing particular application or its part. <BR> 
Yeach application section contains `install:` key which is define if application should be installed or not. <BR> 
Application where the key `install:` defined with `true` value will be installed during the setup process. <BR>
In case you need to exclude particular application from installation process just assign `false` value to `install:` key. 
<BR><b>Example</b> below shows how to disable installation of `argo` tools but allow `kube-metrics` installation. 

```yaml
argo:
  install: false
kube-metrics:
  install: true
```

   </td>
</tr>
<tr>
   <td>superusers:</td>
   <td>List</td>
   <td> 

`- your-github-nickname`

   </td>
   <td>

The pattern word "your-github-nickname" should be replaced <BR> 
withing real GitHub account owner full name. Also can contain several names. <BR>
<b>Example:</b>

```yaml
superusers:
   - John Dou
   - Obi Wan
```

   </td>
</tr>
<tr>
   <td> host: </td> <td> String </td> 
   <td> 

`https://<HOSTNAME_FROM_SERVICE>:30910/vf/ui/`

   </td> 
   <td> 
    This variable contains URL to access application frontend after installation. <BR> 
    The &lt;HOSTNAME_FROM_SERVICE&gt; pattern should be replaced with either valid IP or host name. <BR>
    In case you are going to use host name ensure that it exactly pointing to correct Kubernetes cluster IP address. <BR>
    <b>Example:</b> <BR>
              
```yaml            
host: https://192.168.49.2:30910/vf/ui/ <BR>
```
```yaml
host: https://visualflow.example.com:30910/vf/ui/
```
                
   </td>
</tr>
<tr>
   <td> STRATEGY_CALLBACK_URL:  </td>  <td> String </td> 
   <td> 

`"https://<HOSTNAME_FROM_SERVICE>:30910/vf/ui/callback"`

   </td>         
   <td> 
    
This variable contains callback URL address used by GitHub OAuth service <BR>
to return authentication token to. The &lt;HOSTNAME_FROM_SERVICE&gt; pattern <BR>
should be replaced the same way like for "host:" variable mentioned above. <BR>
<b>Example:</b>
```yaml
STRATEGY_CALLBACK_URL: "https://192.168.49.2:30910/vf/ui/callback" 
```
        
   </td>
</tr>
<tr>
   <td> GITHUB_APP_ID: </td> <td> String </td> <td> "DUMMY_ID"</td> 
   <td> 
The "DUMMY_ID" value should be replaced with the client ID <BR>
automatically generated by GitHub during OAuth application setup process. <BR>
b>Example:</b> <BR>
           
`GITHUB_APP_ID: "3cadfba32f47bcf9d623"`

   </td>
</tr>
<tr>
   <td> GITHUB_APP_SECRET: </td> <td> String </td> <td> "DUMMY_SECRET"</td>
   <td> 
The "DUMMY_SECRET" string should be replaced once you setup <BR>
GitHub OAuth application and generate client secret.<BR>
<b>Example:</b>

`GITHUB_APP_SECRET: "923adb2345bff88620cc797dff987a9889d987b897a6f5aada"`

   </td>
</tr>
</table> 

<b>Example of default values.yaml file content:</b>
  
```yaml
# release name: vf-app
vf-app:
  install: true
  backend:
    deployment:
      secretVariables:
        SLACK_API_TOKEN: ""
    configFile:
      superusers:
        - your-github-nickname
      host: https://<HOSTNAME_FROM_SERVICE>:30910/vf/ui/
  frontend:
    deployment:
      variables:
        STRATEGY_CALLBACK_URL: "https://<HOSTNAME_FROM_SERVICE>:30910/vf/ui/callback"
      secretVariables:
        GITHUB_APP_ID: "DUMMY_ID"
        GITHUB_APP_SECRET: "DUMMY_SECRET"
vf-secrets:
  install: true
argo:
  install: true
kube-metrics:
  install: true
```



1. Navigate to your [working directory](#workdir).

2. Go to the "[visual-flow](https://github.com/ibagroup-eu/Visual-Flow-deploy/blob/minikube/charts/visual-flow)" directory of the downloaded "[Visual-Flow-Deploy](#clonevfrepo)" repository with the following command:

    `cd Visual-Flow-deploy/charts/visual-flow`

3. *(Optional)* Configure Slack notifications in [values.yaml](./charts/visual-flow/values.yaml) using following guide:

    <https://github.com/ibagroup-eu/Visual-Flow-deploy/blob/main/SLACK_NOTIFICATION.md>

4. Define superusers in [values.yaml](./charts/visual-flow/values.yaml).

    New Visual Flow users will have no access in the app. The superusers(admins) need to be configured to manage user access. Specify the superusers real GitHub nicknames in [values.yaml](./charts/visual-flow/values.yaml) in the yaml list format:

    ```yaml
    superusers:
      - your-github-nickname
      # - another-superuser-nickname
    ```

5. If you have installed kube-metrics then update values.yaml file according to the example below.

    1. Check that the kube-metrics installed using the following command:

        ```bash
        kubectl top pods
        ```

        Output if the kube-metrics isn't installed:

        `error: Metrics API not available`

        If the kube-metrics isn't installed then go to step 6.

    2. Edit [values.yaml](./charts/visual-flow/values.yaml) file according to the example below:

        ```yaml
        ...
        kube-metrics:
          install: false
        ```

6. If you have installed Argo workflows then update values.yaml file according to the example below.

    1. Check that the Argo workflows installed using the following command:

        ```bash
        kubectl get workflow
        ```

        Output if the Argo workflows isn't installed:

        `error: the server doesn't have a resource type "workflow"`

        If the Argo workflows isn't installed then go to step 7.

    2. Edit [values.yaml](./charts/visual-flow/values.yaml) file according to the example below:

        ```yaml
        ...
        argo:
          install: false
        vf-app:
          backend:
            configFile:
              argoServerUrl: <Argo-Server-URL>
        ```

7. Install the app using the updated [values.yaml](./charts/visual-flow/values.yaml) file with the following command:

    `helm upgrade -i vf-app . -f values.yaml`

8. Check that the app is successfully installed and all pods are running with the following command:

    `kubectl get pods -A`



### <a id="oauthsetup">Configure GitHub OAuth</a>

>[!NOTE] Update [values.yaml](./charts/visual-flow/values.yaml) file and replace string  `<HOSTNAME_FROM_SERVICE>` with the [Cluster IP address](#clusterIP) received after cluster setup.  
>Also replace all occurences of `<HOSTNAME_FROM_SERVICE>` in below steps.

Create a GitHub OAuth app:

    1. Go to GitHub user's OAuth apps (`https://github.com/settings/developers`) or organization's OAuth apps (`https://github.com/organizations/<ORG_NAME>/settings/applications`).
    2. Click the **Register a new application** or the **New OAuth App** button.
    3. Fill the required fields:
        - Set **Homepage URL** to `https://<HOSTNAME_FROM_SERVICE>:30910/vf/ui/`
        - Set **Authorization callback URL** to `https://<HOSTNAME_FROM_SERVICE>:30910/vf/ui/callback`
    4. Click the **Register application** button.
    5. Replace "DUMMY_ID" with the Client ID value in [values.yaml](./charts/visual-flow/values.yaml).
    6. Click **Generate a new client secret** and replace in [values.yaml](./charts/visual-flow/values.yaml) "DUMMY_SECRET" with the generated Client secret value (Please note that you will not be able to see the full secret value later).

Update 'host' (`host: https://<HOSTNAME_FROM_SERVICE>/vf/ui/`) and 'STRATEGY_CALLBACK_URL' (`STRATEGY_CALLBACK_URL: https://<HOSTNAME_FROM_SERVICE>/vf/ui/callback`) values in [values.yaml](./charts/visual-flow/values.yaml). 

 

12. Upgrade release using updated 'values.yaml':

    `helm upgrade vf-app . -f values.yaml`

13. Wait until the update is installed and all pods are running:

    `kubectl get pods -A`

## <a id="usevf">Use Visual Flow</a>

1. All Visual Flow users (including superusers) need active Github account in order to be authenticated in application. Setup Github profile as per following steps:

    1. Navigate to the [account settings](https://github.com/settings/profile)
    2. Go to **Emails** tab: set email as public by unchecking **Keep my email addresses private** checkbox
    3. Go to **Profile** tab: fill in **Name** and **Public email** fields

2. Open the app's web page using the following link:

    `https://<HOSTNAME_FROM_SERVICE>:30910/vf/ui/`

3. See the guide on how to work with the Visual Flow at the following link: [Visual_Flow_User_Guide.pdf](https://github.com/ibagroup-eu/Visual-Flow/blob/main/Visual_Flow_User_Guide.pdf)

## <a id="uninstallvf">Delete Visual Flow</a>

1. If the app is no longer need, you can delete it using the following command:

    `helm uninstall vf-app`

2. Check that everything was successfully deleted with the command:

    `kubectl get pods --all-namespaces`

#### Delete additional components

If you do no need them anymore - you can also delete and these additional components:

- Redis & PostgreSQL databases

`helm uninstall redis`

`helm uninstall pgserver`

## Delete Minikube cluster and profile

If this cluster is no longer need - you can delete it using the following command:

`minikube delete -p visual-flow`

## <a id="tipsandtricks">Tips & Tricks</a>
Useful comments will be here soon.

### Helpful links about Minikube
- Minikube Start (https://minikube.sigs.k8s.io/docs/start)
- Minikube Basic Control (https://minikube.sigs.k8s.io/docs/handbook/controls)
- Minikube Dashboard (https://minikube.sigs.k8s.io/docs/handbook/dashboard)
- Minikube Tutorials (https://minikube.sigs.k8s.io/docs/tutorials)
- Minikube FAQ (https://minikube.sigs.k8s.io/docs/faq)



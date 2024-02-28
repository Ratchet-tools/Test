# OKD Install
## Docs 
* [OKD Docs: Installing a cluster on bare metal](https://docs.okd.io/latest/installing/installing_bare_metal/installing-bare-metal.html#installation-installing-bare-metal_installing-bare-metal)
* [OKD 4.4Bare Metal Install on VMWare Home Lab](https://medium.com/@craig_robinson/openshift-4-4-okd-bare-metal-install-on-vmware-home-lab-6841ce2d37eb)
* [andy-sokalau/okd-iba@GitHub](https://github.com/andy-sokalau/okd-iba)
## Install Process
> **Note**: The Ignition config files that the installation program generates 
> contain certificates that expire after 24 hours. You must complete your 
> cluster installation and keep the cluster running for 24 hours in a non-degraded 
> state to ensure that the first certificate rotation has finished. You can check 
> your certificate.
### Local Machine
 
* Generate ssh key pair and set OKD_PUBLIC_KEY env var:
  
  ```bash
  export OKD_PUBLIC_KEY=$(cat /home/sysauto/.ssh/id_rsa.pub)
  ```
* Obtain RHEL secret from [here](https://cloud.redhat.com/openshift/install/pull-secret)
* Install `openshift-install` (for OKD it is different than for OCP!) and oc/kubectl client:
  ```bash
  wget https://github.com/openshift/okd/releases/download/4.4.0-0.okd-2020-05-23-055148-beta5/openshift-client-linux-4.4.0-0.okd-2020-05-23-055148-beta5.tar.gz
  wget https://github.com/openshift/okd/releases/download/4.4.0-0.okd-2020-05-23-055148-beta5/openshift-install-linux-4.4.0-0.okd-2020-05-23-055148-beta5.tar.gz
  tar -zxvf openshift-client-linux-4.4.0-0.okd-2020-05-23-055148-beta5.tar.gz
  tar -zxvf openshift-install-linux-4.4.0-0.okd-2020-05-23-055148-beta5.tar.gz
  sudo mv kubectl oc openshift-install /usr/local/bin/
  oc version
  openshift-install version
  ```
* Get latest install script from GitHub
 
  ```bash
  git clone git@github.com:iba-gomel-devops/okd.git
  ```
 
* From the rep directory run:
  ```bash
  ansible-playbook generate-ignition-config.yml -e pull_secret_file=/home/sysauto/OKD/pull-secret.txt
  ```
  It will generate all necessary config files in `/tmp` directory. You can check 
  your certificate now:
  ```bash
  echo | openssl s_client -connect api.okd4.okd.gomel.iba.by:6443 | openssl x509 -noout -text
  ```
* SFTP **bootstrap.ign**, **master.ign** and **worker.ign** to web server (**10.224.0.22**)
  in `/usr/share/nginx/html` directory and set **755** permissions for them. Double check 
  that these 3 files are copied correctly.
### Cloud UI
All actions below are in SPICE console on each ICDC machine (https://manager.dc-iba.by/ovirt-engine/web-ui/):
* For bootstrap server (**10.224.0.20**) run installation from iPXE (option 4 
  in boot menu) and specify/adjust the following parameters:
  ```
  ip=dhcp rd.neednet=1 coreos.inst=yes coreos.inst.install_dev=sda coreos.inst.image_url=http://admin.okd.gomel.iba.by:8080/fedora-coreos-31.20200505.3.0-metal.x86_64.raw.xz coreos.inst.ignition_url=http://admin.okd.gomel.iba.by:8080/bootstrap.ign
  ```
  The console window shows many messages during the install and restarts several 
  times. After installation of the image, the VM restarts to the login screen 
  and begins to show the following:
  > _\<missing pucture\>_
  Fedore CoreOS release will be automatically updated to the latest based on ostree process.
* Do the same for all masters and workers, but adjust coreos.inst.ignition_url parameter 
  for them to `coreos.inst.ignition_url=http://admin.okd.gomel.iba.by:8080/master.ign` and 
  `coreos.inst.ignition_url=http://admin.okd.gomel.iba.by:8080/worker.ign` respectively.
### Cloud Machines
* Go to each node (`ssh core@<master or worker IP>`) and do the following:
  ```
  sudo vi /etc/hosts
  ```
  set entry for this particular host and restart a host. As a result, after the 
  reboot you should see the following:
  ```bash
  [core@okd4m2 ~]$ cat /etc/hosts
  10.224.0.19 okd4m2.okd.gomel.iba.by
  ```
  Do it for all cluster servers, except the bootstrap server.
* Go to `/tmp` directory on local pc (where ignition files are created) and run
  ```bash
  openshift-install wait-for bootstrap-complete --log-level=info
  ```
  Wait for it's completion. Ideally you should see the following output:
  ```
  INFO Waiting up to 30m0s for the cluster at https://api.okd4.okd.gomel.iba.by:6443 to initialize...
  INFO Waiting up to 10m0s for the openshift-console route to be created...
  INFO Install complete!
  INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/tmp/auth/kubeconfig'
  INFO Access the OpenShift web-console here: https://console-openshift-console.apps.okd4.okd.gomel.iba.by
  INFO Login to the console with user: kubeadmin, password:
  ```
  
  Wait some time and check that all is fine:
  ```bash
  export KUBECONFIG=/tmp/auth/kubeconfig
  oc get clusteroperators
  oc get nodes
  oc get pods -A
  ```
### Post-install
  
* Do post install action as needed (add persistence storage, configure OKD internal 
  registry, set identity provider and etc.)
* At this point, you can shutdown your bootstrap node. Now is a good time to edit the 
  `/etc/haproxy/haproxy.cfg`, comment out the bootstrap node, and restart the haproxy 
  service. It should be done on the LB machine (**10.224.0.22** in our case):
  ```bash
  sudo vi /etc/haproxy/haproxy.cfg
  sudo systemctl restart haproxy
  ```
 
> **Note**: See more docs regarding post-install actions under okd/docs/ directory: OKD_NFS_config.md, OKD_gitops_argocd.md and etc.

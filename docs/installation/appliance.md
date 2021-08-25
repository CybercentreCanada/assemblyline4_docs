# Assemblyline Appliance

This is the documentation for an appliance instance of the Assemblyline platform suited for smaller scale deployments. Since we've used microk8s as the backend for this, the appliance setup can later be scaled to multiple nodes.

## Setup requirements

Note: The documentation provided here assumes that you are installing your appliance on a Ubuntu based system and was only tested on Ubuntu 20.04. You might have to change the commands a bit if you use other linux distributions.

### Install pre-requisites:

1. Install microk8s: 
```
sudo snap install microk8s --classic
```
2. Install microk8s addons:  
```
sudo microk8s enable dns ha-cluster ingress storage metrics-server
```
3. Install Helm and set it up to use with microk8s:
```
sudo snap install helm --classic
sudo mkdir /var/snap/microk8s/current/bin
sudo ln -s /snap/bin/helm /var/snap/microk8s/current/bin/helm
```
4. Install git: 
```
sudo apt install git
```

### (Optional) Add more nodes!!!
**Note: This can be done before or after the system is live.**

From the master node, run:
```
sudo microk8s add-node
```

This will generate a command with a token to be executed on a standby node.

On your standby node, ensure the microk8s ```ha-cluster``` addon is enabled before
running the command from the master to join the cluster.

To verify the nodes are connected, run (on any node): 
```
sudo microk8s kubectl get nodes
```

Repeat this process for any additional standby nodes that's to be added.

For more details, see: [Clustering with MicroK8s](https://microk8s.io/docs/clustering)

## Get the Assemblyline chart to your computer

### Get the Assemblyline helm charts

```
mkdir ~/git && cd ~/git
git clone https://github.com/CybercentreCanada/assemblyline-helm-chart.git
```

### Create your personal deployment

```
mkdir ~/git/deployment
cp ~/git/assemblyline-helm-chart/appliance/*.yaml ~/git/deployment
```

### Setup the charts and secrets

The ```values.yaml``` file in your deployment directory ```~/git/deployment``` is already pre-configured for use with microk8s as a basic one node minimal appliance. Make sure you go through the file to adjust disk sizes and to turn on/off features to your liking.

The ```secret.yaml``` file in your deployment directory is preconfigured with default passwords, you should definitely change them. (NOTE: the secrets are used to setup during bootstrap so make sure you change them before deploy the al chart.)

## Deploy Assemblyline via Helm:

### Create a namespace for your Assemblyline install

For the purpose of this documentation we will use ```al``` as the namespace.

```
sudo microk8s kubectl create namespace al
```

### Deploy the secret to the namespace

```
sudo microk8s kubectl apply -f ~/git/deployment/secrets.yaml --namespace=al
```

From this point on, you don't need the secrets.yaml file anymore. You should delete it so there is no file on disk containing your passwords.

```
rm ~/git/deployment/secrets.yaml
```

### Finally, let's deploy Assemblyline's chart:

For the purpose of this documentation we will use ```assemblyline``` as the deployment name.

```
sudo microk8s helm install assemblyline ~/git/assemblyline-helm-chart/assemblyline -f ~/git/deployment/values.yaml -n al
```
## Updating the current deployment

### To get the latest chart changes (optional)
If you want to get the latest changes that we did to the chart, just pull the changes. (This could conflict with the changes you've made so be careful while doing this.)
```
cd ~/git/assemblyline-helm-chart && git pull
```

### Update the deployment
Once you have you're Assemblyline chart deployed throught helm, you can change any values in the ```values.yaml``` file and upgrade your deployment with the following command:
```
sudo microk8s helm upgrade assemblyline ~/git/assemblyline-helm-chart/assemblyline -f ~/git/deployment/values.yaml -n al
```

## Quality of life improvements

### Lens IDE
If the computer on which your microk8s deployment is install has a desktop interface, I strongly suggest that you use K8s Lens IDE to manage the system

#### Install Lens
```
sudo snap install kontena-lens --classic
```
#### Configure Lens
After you run Lens for the first time, click the "Add cluster" menu/button, select the paste as text tab and paste the output of the following command:
```
sudo microk8s kubectl config view --raw
```

### Alias to kubectl 

Since all is running inside microk8s you can create an alias to the kubectl addon in your bashrc to make your life easier
```
alias kubectl='sudo microk8s kubectl --namespace=al'
```

## Alternative Installations:

We will officially only support microk8s installations for appliances but you can technically install it on any local kubernetes frameworks (k3s, minikube. kind...). That said there will be no documentation for theses setups and you will have to modify the ```values.yaml``` storage classes to fit with your desired framework.
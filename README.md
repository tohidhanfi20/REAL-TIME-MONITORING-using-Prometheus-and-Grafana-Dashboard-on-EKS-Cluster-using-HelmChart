# REAL-TIME-MONITORING-using-Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-HelmChart

# Step 1 Setup EC2 Instance Instance Type as t2.medium AMIs as UbuntuUS-EAST-1


Step 1.1 – Create the IAM role having full access

Go to IAM -> Create role -> Select EC2 -> Give Full admin access

Step 1.2 – Attach the IAM role having full access

Go to EC2 -> Click on Actions on the left hands ide -> Security -> Modify IAM role

# Step 2 - Install AWS CLI and Configure.

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip"-o"awscliv2.zip"

sudo apt install unzip 
unzipawscliv2.zip
sudo./aws/install

# Step 3 - Install and Setup Kubectl

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl-s

https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x ./kubect

sudo mv ./kubectl /usr/local/bin

kubectl version

kubectl version --short

# Step 4- Install and Setup eksctl

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin

eksctl version

# Step 5 - Install Helm chart

curl -fsSL-o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh ./get_helm.sh

helm version

# Step 6 - Creating an AmazonEKS cluster using eksctl

    1. Name of the cluster: --eks2
    2. Version of Kubernetes: --version1.24
    3. Region: --region us-east-1
    4. Nodegroupname/workernodes: --nodegroup-nameworker-nodes
    5. NodeType: -- nodegroup-typet2.medium
    6. Numberofnodes: --nodes2
    7. MinimumNumberofnodes: --nodes-min2
    8. MaximumNumberofnodes: --nodes-max3

# Step 6.1 -IF ANY ERROR

aws eks update-kubeconfig --region <region-code> --name <cluster-name>

# Step 7 - Installing the Kubernetes Metrics Server

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Step 7.1 - Verify that the metrics-server deployment is running the desired number of pods with the following command

kubectl get deployment metrics-server -n kube-system

# Step 8 - Install Prometheus

Now install the Prometheus using the helm chart.
Add Prometheus helm chart repository

helm repo add prometheus-community 
https://prometheus-community.github.io/helm-charts

# Step 8.1 - Update helm chart repository

helm repo update 
helm repo list

# Step 8.2 Create prometheus namespace

kubectl create namespace prometheus

# Step 8.3 - Install prometheus

helm install prometheus prometheus-community/prometheus --namespaceprometheus --set alertmanager.persistentVolume.storageClass="gp2" --set server.persistentVolume.storageClass="gp2"


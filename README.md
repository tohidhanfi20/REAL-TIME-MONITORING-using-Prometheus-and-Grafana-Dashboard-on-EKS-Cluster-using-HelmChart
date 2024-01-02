# REAL-TIME-MONITORING-using-Prometheus-and-Grafana-Dashboard-on-EKS-Cluster-using-HelmChart



# Step 1 - Setup EC2 Instance Instance Type as t2.medium AMIs as UbuntuUS-EAST-1


Step 1.1 – Create the IAM role having full access

Go to IAM -> Create role -> Select EC2 -> Give Full admin access

Step 1.2 – Attach the IAM role having full access

Go to EC2 -> Click on Actions on the left hands ide -> Security -> Modify IAM role

# Step 2 - Install AWS CLI and Configure.

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip"-o"awscliv2.zip"

sudo apt install unzip 

unzip awscliv2.zip

sudo ./aws/install

# Step 3 - Install and Setup Kubectl

sudo apt update

sudo apt install curl -y

curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x ./kubectl

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

# Step 9 - Create IAM OIDC Provider

Your cluster has an Open IDConnect(OIDC) issuer URL associated with it. To use AWS Identity and Access Management (IAM) roles for service accounts, an IAMOIDC provider must exist for your cluster's OIDC issuer URL.

oidc_id=$(awseks describe-cluster --nameeks2 --regionus-east-1 --query"cluster.identity.oidc.issuer" --output text | cut-d '/' -f 5)

aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4

eksctl utils associate-iam-oidc-provider --cluster eks2 --approve --region us-east-1

# Step 10 – Create iam service account with role

eksctl create iamserviceaccount --name ebs-csi-controller-sa --namespace kube-system --cluster eks2 --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy --approve --role-only --role-name AmazonEKS_EBS_CSI_DriverRole --region us-east-1

# Step 10.1 - Then attach ROLE to eks by running the following command

Enter your account ID and cluster name.

eksctl create add on --name aws-ebs-csi-driver --clustereks2 --service-account-role-arn arn:aws:iam::164297528770:role/AmazonEKS_EBS_CSI_DriverRole --force--region us-east-1

# Step 10.2 - kubectl get pods -n prometheus

# Step 10.3 - View the Prometheus dashboard by forwarding the deployment ports

kubectl port-forward deployment/prometheus-server9090:9090 -n prometheus

# Step 10.4 - Open different browser and connect to your EC2 instance and run 

curl localhost:9090/graph

# Step 1 1 - Install Grafana

helm repo add grafana 
https://grafana.github.io/helm-chart

helm repo update

# Step 11.1 - Create a namespace Grafana

kubectl create namespace grafana

# Step 11.2 - Install the Grafana

helm install grafana grafana/grafana --namespacegrafana --setpersistence.storageClassName="gp2" --setpersistence.enabled=true --setadminPassword='EKS!sAWSome' --setservice.type=LoadBalancer

This command will create the Grafana service with an external load balancer to get the public view.

# Step 11.3 - Verify the Grafana installation by using the following kubectl command

kubectl get pods -n grafana

kubectl get service -n grafana

# Step 11.4 - Copy th EXTERNAL - IP and paste in browser

Password you mentioned as EKS!sAWSome while creating Grafana

# Step 11.5 - Add the Prometheus as the datasource to Grafana

Go to Grafana Dashboard -> Add the Data source -> Select the Prometheus

# Step 11.6 - Configure the endpoints of Prometheus and save

URL- http://prometheus-server.prometheus.svc.cluster.local

# Step 11.6 -Import Grafana dashboard from Grafana Labs

Now we have setup everything in terms of Prometheus and Grafana. For the custom Grafana Dashboard,
we are going to use the open source grafana dashboard. For this session, Iam going to import a dashboard 6417

Go to left side -> click on dashboards -> Click on New -> Import

Load and select the source as Prometheus

# Step 12 – Visualise the java application

# Step 13 - Deploya Java application and monitor it on Grafana

git clone https://github.com/tohidhanfi20/kubernetes_java_deployment

cd /kubernetes_java_deployment/Kubernetes/

kubectl apply -f shopfront-service.yaml
kubectl get deployment
kubectl get pods

# Step 14 - Clean Up

eksctl delete cluster --nameeks2 --region us-east-1


# THANK YOU

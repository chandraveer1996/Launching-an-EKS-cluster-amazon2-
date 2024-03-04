# Launching-an-EKS-cluster-amazon2-

Launching an EKS Cluster

Create an IAM User with Admin Permissions

    Navigate to IAM > Users.
    Click Add users.
    In the User name field, enter k8-admin.
    Click Next.
    Select Attach policies directly.
    Select AdministratorAccess.
    Click Next.
    Click Create user.
    Select the newly created user k8-admin.
    Select the Security credentials tab.
    Scroll down to Access keys and select Create access key.
    Select Command Line Interface (CLI) and checkmark the acknowledgment at the bottom of the page.
    Click Next.
    Click Create access key.
    Either copy both the access key and the secret access key and paste them into a local text file, or click Download .csv file. We will use the credentials when setting up the AWS CLI.
    Click Done.

Launch an EC2 Instance and Configure the Command Line Tools



    Navigate to EC2 > Instances.

    Click Launch Instance.

    At the Amazon Machine Image (AMI) dropdown, select the Amazon Linux 2 AMI.

    Leave t2.micro selected under Instance type.

    In the Key pair (login) box, select Create new key pair.

    Give it a Key pair name of mynvkp.

    Click Create new key pair. This will download the key pair for later use.

    Expand Network settings and click on Edit.

    In the Network settings box:
        Network: Leave as default.
        Subnet: Leave as default.
        Auto-assign Public IP: Select Enable.

    Click Launch instance.

    Click on the instance ID link (which looks like i-xxxxxxxxx), and give the new instance a few minutes to enter the running state.

    Once the instance is fully created, check the checkbox next to it and click Connect at the top of the window.

    In the Connect to your instance dialog, select EC2 Instance Connect (browser-based SSH connection).

    Click Connect.

    In the command line window, check the AWS CLI version:

    aws --version

    It should be an older version.

    Download v2:

    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

    Unzip the file:

    unzip awscliv2.zip

    See where the current AWS CLI is installed:

    which aws

    It should be /usr/bin/aws.

    Update it:

    sudo ./aws/install --bin-dir /usr/bin --install-dir /usr/bin/aws-cli --update

    Check the version of AWS CLI:

    aws --version

    It should now be updated.

    Configure the CLI:

    aws configure

    For AWS Access Key ID, paste in the access key ID you copied earlier.

    For AWS Secret Access Key, paste in the secret access key you copied earlier.

    For Default region name, enter us-east-1.

    For Default output format, enter json.

    Download kubectl:

    curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.16.8/2020-04-16/bin/linux/amd64/kubectl

    Apply execute permissions to the binary:

    chmod +x ./kubectl

    Copy the binary to a directory in your path:

    mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin

    Ensure kubectl is installed:

    kubectl version --short --client

    Download eksctl:

    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

    Move the extracted binary to /usr/bin:

    sudo mv /tmp/eksctl /usr/bin

    Get the version of eksctl:

    eksctl version

    See the options with eksctl:

    eksctl help

Provision an EKS Cluster

Provision an EKS cluster with three worker nodes in us-east-1:

eksctl create cluster --name dev --region us-east-1 --nodegroup-name standard-workers --node-type t3.medium --nodes 3 --nodes-min 1 --nodes-max 4 --managed
It will take 10–15 minutes since it's provisioning the control plane and worker nodes, attaching the worker nodes to the control plane, and creating the VPC, security group, and Auto Scaling group.



Navigate to EC2 > Instances, where you should see the instances have been launched.

Close out of the existing CLI window, if you still have it open.

Select the original t2.micro instance, and click Connect at the top of the window.

In the Connect to your instance dialog, select EC2 Instance Connect (browser-based SSH connection).

Click Connect.

In the CLI, check the cluster:

eksctl get cluster

Enable it to connect to our cluster:

aws eks update-kubeconfig --name dev --region us-east-1

Create a Deployment on Your EKS Cluster

cat nginx-deployment.yaml
cat nginx-svc.yaml



Create the service:

kubectl apply -f ./nginx-svc.yaml

Check its status:

kubectl get service
Copy the external DNS hostname of the load balancer, and paste it



Create the deployment:

kubectl apply -f ./nginx-deployment.yaml

Check its status:

kubectl get deployment

View the pods:

kubectl get pod

View the ReplicaSets:

kubectl get rs

View the nodes:

kubectl get node

Access the application using the load balancer, replacing <LOAD_BALANCER_DNS_HOSTNAME> with the IP you copied earlier (it might take a couple of minutes to update):

curl "<LOAD_BALANCER_DNS_HOSTNAME>"


=========================================

OPTIONAL

Test the High Availability Features of Your EKS Cluster

    In the AWS console, on the EC2 instances page, select the worker node instances.

    Click Instance state.

    Select Stop instance.

    In the dialog box, click Stop.

    After a few minutes, we should see EKS launching new instances to keep our service running.

    In the CLI, check the status of our nodes:

    kubectl get node

    All the nodes should be down (i.e., display a NotReady status).

    Check the pods:

    kubectl get pod

    We'll see a few different statuses — Terminating, Running, and Pending — because, as the instances shut down, EKS is trying to restart the pods.

    Check the nodes again:

    kubectl get node

    We should see a new node, which we can identify by its age.

    Wait a few minutes, and then check the nodes again:

    kubectl get node

    We should have one in a Ready state.

    Check the pods again:

    kubectl get pod

    We should see a couple pods are now running as well.

    Check the service status:

    kubectl get service

    Copy the external DNS Hostname listed in the output.

    Access the application using the load balancer, replacing <LOAD_BALANCER_DNS_HOSTNAME> with the DNS Hostname you just copied:

    curl "<LOAD_BALANCER_EXTERNAL_IP>"

    We should see the Nginx web page HTML again. (If you don't, wait a few more minutes.)

    In a new browser tab, navigate to the same IP, where we should again see the Nginx web page.

    In the CLI, delete everything:

    eksctl delete cluster dev --region us-east-1



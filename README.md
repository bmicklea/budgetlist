# Building the Application Locally

### Pre-requisites
- Kubernetes
- AWS Account Access
- Optional awscli
- Git repo connection
- Connection to private container registry
- Connection to private artifact repository

```
curl -sSL https://get.docker.com/ | sh
apt-get install -yq jq python-pip curl unzip build-essential python-dev
pip install awscli
curl -o /tmp/terraform.zip https://releases.hashicorp.com/terraform/0.7.11/terraform_0.7.11_linux_amd64.zip
unzip /tmp/terraform.zip -d /usr/bin
```

```
git clone https://github.com/microservices-demo/microservices-demo
cd microservices-demo
```
### Setup Kubernetes
Begin by setting the appropriate AWS environment variables.

```
export AWS_ACCESS_KEY_ID=[YOURACCESSKEYID]
export AWS_SECRET_ACCESS_KEY=[YOURSECRETACCESSKEY]
export AWS_DEFAULT_REGION=[YOURDEFAULTREGION]
export TF_VAR_aws_region=$AWS_DEFAULT_REGION
```

Next we’ll create a private key for use during this demo.

```
aws ec2 create-key-pair --key-name deploy-docs-k8s --query 'KeyMaterial' --output text > ~/.ssh/deploy-docs-k8s.pem
chmod 600 ~/.ssh/deploy-docs-k8s.pem
```

Finally run terraform.

`terraform apply deploy/kubernetes/terraform/`

Our master node makes use of some of the files in this repo so lets securely copy those over.

```
master_ip=$(terraform output -json | jq -r '.master_address.value')
scp -i ~/.ssh/deploy-docs-k8s.pem -o StrictHostKeyChecking=no -rp deploy/kubernetes/manifests ubuntu@$master_ip:/tmp/
```

### Post-Install Configuration
Run the following commands to setup Kubernetes and Weave Net on the master instance

```
master_ip=$(terraform output -json | jq -r '.master_address.value')
ssh -i ~/.ssh/deploy-docs-k8s.pem ubuntu@$master_ip sudo kubeadm init > k8s-init.log
ssh -i ~/.ssh/deploy-docs-k8s.pem ubuntu@$master_ip "sudo cp /etc/kubernetes/admin.conf /home/ubuntu/"
ssh -i ~/.ssh/deploy-docs-k8s.pem ubuntu@$master_ip "sudo chown \$(id -u):\$(id -g) \$HOME/admin.conf"
grep -e --token k8s-init.log > join.cmd
ssh -i ~/.ssh/deploy-docs-k8s.pem ubuntu@$master_ip KUBECONFIG=\$HOME/admin.conf kubectl apply -f https://git.io/weave-kube-1.6
```

Run the following commands to SSH into each node_addresses and run the `kubeadm join --token <token> <master-ip>` command from before.
    
```
node_addresses=$(terraform output -json | jq -r '.node_addresses.value|@sh' | sed -e "s/'//g" )
for node in $node_addresses; do
    ssh -i ~/.ssh/deploy-docs-k8s.pem -o StrictHostKeyChecking=no ubuntu@$node sudo `cat join.cmd`
done
```

### Setup Scope
There are two options for running Weave Scope, either you can run the UI locally, or using the hosted provider at `cloud.weave.works`

#### Locally
- SSH into the master node
- Start weave scope on the cluster

```
master_ip=$(terraform output -json | jq -r '.master_address.value')
ssh -i ~/.ssh/deploy-docs-k8s.pem ubuntu@$master_ip KUBECONFIG=\$HOME/admin.conf kubectl apply -f 'https://cloud.weave.works/launch/k8s/weavescope.yaml'
```

### Deploy the App
- SSH into the master node
- Deploy the app

```
master_ip=$(terraform output -json | jq -r '.master_address.value')
ssh -i ~/.ssh/deploy-docs-k8s.pem ubuntu@$master_ip KUBECONFIG=\$HOME/admin.conf kubectl apply -f /tmp/manifests/sock-shop-ns.yaml -f /tmp/manifests
```

### View the results
Run terraform output command to see the load balancer and node URLs

The app is available at the sock_shop_address as displayed below. The scope app is accessible via the master and any of the node urls on port 30001, while the same applies for Kibana if you deployed it, but using port 31601. It may take a few moments for the apps to get running.

Outputs:

```
master_address = ec2-52-213-213-161.eu-west-1.compute.amazonaws.com
node_addresses = [
    ec2-52-213-136-12.eu-west-1.compute.amazonaws.com,
    ec2-52-208-64-132.eu-west-1.compute.amazonaws.com,
    ec2-52-48-129-206.eu-west-1.compute.amazonaws.com
]
sock_shop_address = MD-k8s-elb-sock-shop-1211989270.eu-west-1.elb.amazonaws.com
```

### Run tests
There is a separate load-test available to simulate user traffic to the application. For more information see Load Test. This will send some traffic to the application, which will form the connection graph that you can view in Scope or Weave Cloud.

```
    elb_url=$(terraform output -json | jq -r '.sock_shop_address.value')
    docker run --rm weaveworksdemos/load-test -d 300 -h $elb_url -c 2 -r 100
```

### Opentracing
Zipkin is part of the deployment and has been written into some of the services. While the system is up you can view the traces in Zipkin at http://<loadbalancer>:9411. Currently orders provide the most comprehensive traces.


# Developing using CodeReady Workspaces

To contribute to this project without having to install or configure software, we use Red Hat CodeReady Workspaces on Red Hat OpenShift Kubernetes.

Just click the button to jump into the preconfigured project in the IDE:
<a href="http://codeready-crw.apps.ocp4.home.ocpcloud.com/f?name=RHTE2019&user=bmicklea">
    <img src="http://beta.codenvy.com/factory/resources/codenvy-contribute.svg" width="130" alt="Push" align="top">
</a>

Alternatively click this link: https://che.openshift.io/f?id=factorylztfoa6yozjl88k9

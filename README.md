# Simple Kubernetes Cluster

## This kubernetes cluster configuration enables 3 nodes, one master and two workers.

###Prerequisites
* SSH key pair on your local machine to access your cluster's node.
* Three servers runing Ubuntu 18.04 with at 2GB RAM and 2 CPU's.
* Ansible on your local machine to run playbooks.

### CONFIGURATION

#### Step 1 -   Setting up the workspace
* In the file "hosts", replace 'master_ip_address', 'worker01_ip_address' and 'worker02_ip_address' with the machine ip addresses.

#### Step 2 - Create non-root user on all remote servers
Run the following command:
> ansible-playbook -i hosts ~/kubenetes-cluster/initial.yml

#### Step 3 - Installing kubernetes dependencies
Run the following command:
> ansible-playbook -i hosts ~/kubenetes-cluster/kubernetes-dependencies.yml

#### Step 4 - Setting up master node
Run the following command:
> ansible-playbook -i hosts ~/kubenetes-cluster/master.yml

Now we can check the status of the master node:
> $ ssh ubuntu@master_ip_address
kubectl get nodes

You will now see the following output:
> NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    1d        v1.14.0

#### Step 5 - Setting up worker nodes
Run the following command:
> ansible-playbook -i hosts ~/kubenetes-cluster/workers.yml

#### Step 6 - Veritying the cluster
SSH into the master node
> $ ssh ubuntu@master_ip_address

And run the command below:
> kubectl get nodes

You should see the output below:
> NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    1d        v1.14.0
worker1   Ready     <none>    1d        v1.14.0
worker2   Ready     <none>    1d        v1.14.0

#### Step 7 - Running an application on the cluster
At thisn point you're able to run a containerized application on the cluster. In this case we're going to deploy NGINX using *Deployments* and *Services*

On the master node run the following commands:
> $ kubectl create deployment nginx --image=nginx

> $ kubectl expose deploy nginx --port 80 --target-port 80 --type NodePort

> $ kubectl get services

You should see something familiar to this output:
NAME | TYPE | CLUSTER-IP | EXTERNAL-IP | PORT(S) | AGE
---- | ----- | --------- | ----------- | ------- | ---
kubernetes | ClusterIP | 10.96.0.1 | <none> | 443/TCP | 1d
nginx | NodePort | 10.109.228.209 | <none> | 80:nginx_port/TCP | 40m

To test that everything is working, visit http://worker_1_ip:nginx_port or http://worker_2_ip:nginx_port through a browser on your local machine. You will see Nginx’s familiar welcome page.

**EXTRAS**
If you would like to remove the Nginx application, first delete the nginx service from the master node:

> $ kubectl delete service nginx

> $ kubectl get services

You will see the following output:

Output
>NAME         TYPE        CLUSTER-IP       EXTERNAL-IP           PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>                443/TCP        1d

Then delete the deployment:

> $ kubectl delete deployment nginx

Run the following to confirm that this worked:

> $ kubectl get deployments

Output:

>No resources found.
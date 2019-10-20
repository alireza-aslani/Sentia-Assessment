# Prerequisites
- [EC2 Key Pair](https://console.aws.amazon.com/ec2/v2/home)
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)
- [kubectl & aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)


# Stacks

## VPC
This [kubernetes-vpc-stack-public-only](kubernetes-vpc-stack-public-only.yaml) creates the [Amazon Virtual Private Cloud](https://aws.amazon.com/vpc/) that our Kubernetes cluster will run inside. you can config from 2 to 4 different public subnet in a vpc. there is an extended version that has more features which you can use that for more customization .
> if you want to just see how it works just leave the defaul values select to AZs and set the stack name to `eks-vpc` because they are needed in the next few steps.

## Cluster
This [kubernetes-eks-cluster](/kubernetes-eks-cluster.yaml) creates the [AWS Kubernetes EKS Cluster](https://aws.amazon.com/eks/) that our worker nodes will be associated with.
> Copy the `VPCStack` parameter from the previous step into the corresponding `VPCStack` parameter ,for example I used`eks-vpc`, next create the stack .
Record the `ClusterName` and `ClusterEndpoint` outputs because they are needed in the next few steps.

## Nodes
This [kubernetes-eks-nodes](/kubernetes-eks-nodes.yaml) creates the [EC2](https://aws.amazon.com/ec2/) nodes that will run our Kubernetes containers.
>Amazon Elastic Compute Cloud (Amazon EC2) is a web service that provides secure, resizable compute capacity in the cloud. It is designed to make web-scale cloud computing easier for developers.

Copy the `ClusterName` output from the previous step into the corresponding `ClusterName` parameter . Then Copy the `ClusterStackName` from the previous step into the corresponding `ClusterStack` parameter ,for example I used`eks-cluster`, The same applies to `VPCStack`. then create the stack .

# Client Setup
Once all of your stacks are up it's time to configure your local environment to connect to your new Kubernetes cluster.  We also have to configure your worker nodes and associate them with your cluster.

use `aws configure` in your terminal and make sure that you have the credentials and then use this comman to generate the ${HOME}.KUBE/CONFIG file `aws eks update-kubeconfig --name <ClusterName>`
check the cluster now `kubectl --kubeconfig=/home/alireza/.kube/config cluster-info`

## Enable worker nodes to join your cluster
Download, edit, and apply the AWS authenticator configuration map:

1.) Download the configuration map.
```
curl -O https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-06-05/aws-auth-cm.yaml
```

2.) Open the file with your favorite text editor. Replace the <ARN of instance role (not instance profile)> snippet with the `NodeInstanceRole` value that you recorded in the previous procedure, and save the file.

This will be the `NodeInstanceRole` output from the nodes stack.

Important
> Do not modify any other lines in this file.

3.) Apply the configuration. This command may take a few minutes to finish.
```
kubectl apply -f aws-auth-cm.yaml
```

4.) Watch the status of your nodes and wait for them to reach the Ready status.
```
kubectl get nodes --watch
```
# Application & Database Setup

## MySQL
This [mysql.yaml](/mysql.yaml) creates the database for our application . use valid password because we are going to use it later . fill the VPCSTACK parameter with `eks-vpc`.
> this database can be used only from inside of vpc so do not try to connect to its endpoint from your system .

## Wordpress Application (Simple Version)
clone my [WP-Sample](https://github.com/alireza-aslani/WP-Sample) repo for deploying first wordpress website in your eks cluster. 
1.) Create database password
```
kubectl create secret generic mysql-pass --from-literal=password=1234qwer
```
2.) Create the [kubernetes storage class](https://github.com/alireza-aslani/WP-Sample/blob/master/kubernetes-storage-class.yaml)
```
kubectl apply -f kubernetes-storage-class.yaml
```
3.) Open [wordpress-deployment](https://github.com/alireza-aslani/WP-Sample/blob/master/wordpress-deployment.yaml) and set `WORDPRESS_DB_HOST` with the `DatabaseEndpoint` output from the previous step, now apply:
```
kubectl apply -f wordpress-deployment.yaml
```
4.) use `a3a994402f38411e9887912f5f382f18-377045863.us-east-1.elb.amazonaws.com` this command and you should see the ALB url under the EXTERNAL-IP . open the url in your browser :)

# Prerequisites
- [EC2 Key Pair](https://console.aws.amazon.com/ec2/v2/home)
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)
- [kubectl & aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)


# Stacks

## VPC
This [kubernetes-vpc-stack-public-only](kubernetes-vpc-stack-public-only.yaml) creates the [Amazon Virtual Private Cloud](https://aws.amazon.com/vpc/) that our Kubernetes cluster will run inside. You can configure from 2 to 4 different public subnets in a vpc. An extended version exists that has more features which you can use for more customizations.
> If you want to just see how it works, just leave the default values and select two different AZs (prefarably us-east-1a,us-east-1b) and set the stack name to `eks-vpc` because they are needed in the next few steps.

## Cluster
This [kubernetes-eks-cluster](/kubernetes-eks-cluster.yaml) creates the [AWS Kubernetes EKS Cluster](https://aws.amazon.com/eks/) that our worker nodes will be associated with.
> Copy the `VPCStack` parameter from the previous step into the corresponding `VPCStack` parameter. For example I used`eks-vpc`. Next create the stack.
Record the `ClusterName` and `ClusterEndpoint` outputs because they are also needed in the next few steps.

## Nodes
This [kubernetes-eks-nodes](/kubernetes-eks-nodes.yaml) creates the [EC2](https://aws.amazon.com/ec2/) nodes that will run our Kubernetes containers.
> Amazon Elastic Compute Cloud (Amazon EC2) is a web service that provides secure, resizable computing capacity in the cloud. It is designed to make web-scale cloud computing easier for developers.

Copy the `ClusterName` output from the previous step into the corresponding `ClusterName` parameter. Then Copy the `ClusterStackName` from the previous step into the corresponding `ClusterStack` parameter. For example I used`eks-cluster`, The same applies to `VPCStack`. Afterwards, create the stack .

# Client Setup
Once all of your stacks are up and running, it's time to configure your local environment to connect to your new Kubernetes cluster. We also have to configure the worker nodes and associate them with your cluster.

Use `aws configure` in your terminal and make sure that you have the credentials and then use this command to generate the ${HOME}.KUBE/CONFIG file `aws eks update-kubeconfig --name <ClusterName>`
Check the cluster info by printing it to the console using: `kubectl --kubeconfig=/home/alireza/.kube/config cluster-info`

## Enable worker nodes to join your cluster
Download, edit, and apply the AWS authenticator configuration map:

1.) Download the configuration map.

```bash
curl -O https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-06-05/aws-auth-cm.yaml
```

2.) Open the file with your favorite text editor. Replace the `<ARN of instance role (not instance profile)>` snippet with the `NodeInstanceRole` value that you recorded in the previous procedure and save the file.

This will be the `NodeInstanceRole` output from the nodes stack.

Important
> Do not modify any other lines in this file.

3.) Apply the configuration. This command may take a few minutes to finish.

```bash
kubectl apply -f aws-auth-cm.yaml
```

4.) Watch the status of your nodes and wait for them to reach the **ready** status.

```bash
kubectl get nodes --watch
```

# Application & Database Setup

## MySQL
The [mysql.yaml](/mysql.yaml) file creates the database for our application. Use a valid password because we are going to use it later. Fill the VPCSTACK parameter with `eks-vpc`.
> This database can be used only from inside the vpc so do not try to connect to its endpoint from your localhost .

## Wordpress Application (Simple Version)

Clone the [WP-Sample](https://github.com/alireza-aslani/WP-Sample) repo for deploying a Wordpress website in your eks cluster.

1.) Create the database password

```bash
kubectl create secret generic mysql-pass --from-literal=password=1234qwer
```

2.) Create the [kubernetes storage class](https://github.com/alireza-aslani/WP-Sample/blob/master/kubernetes-storage-class.yaml)

```bash
kubectl apply -f kubernetes-storage-class.yaml
```

3.) Open [wordpress-deployment](https://github.com/alireza-aslani/WP-Sample/blob/master/wordpress-deployment.yaml) and set `WORDPRESS_DB_HOST` with the `DatabaseEndpoint` output from the previous step, now apply:

```bash
kubectl apply -f wordpress-deployment.yaml
```

4.) Use the `kubectl get svc` command and you should see the `ELB` generated url under the EXTERNAL-IP. Open the url in your browser :)

## Wordpress Application (Extended Version With CI/CD)

At first I decided to use everything in a cloud native way. Therefore, I wrote the AWS code pipline and AWS code build templates for cloudformation and I created a [buildspec.yml](https://github.com/alireza-aslani/WP-Sample/blob/master/buildspec.yml) file but unfortunately my AWS account has some limitations and I faced this error `User: arn:aws:iam ... is not authorized to perform: codebuild:CreateProject on ....` during the test proccess. So i decided to implement the CI/CD my way. Open the [WP-Sample-Extended-Project](https://github.com/alireza-aslani/WP-Sample-Extended). There is a python code in the root folder. What it does is it listens to requests which come from the GitLab runner. This is a three level ci. If you look at [gitlab-ci.yml](https://github.com/alireza-aslani/WP-Sample-Extended/blob/master/.gitlab-ci.yml) you can see: First I build the docker image that contains all of the wordpress files and then I push the image into my private registry with two different tags (latest and versioned). For rollback purposes, you can deploy with the healthy version of the tag. At last you can leave the rest of the pipline to [python-ci-server.py](https://github.com/alireza-aslani/WP-Sample-Extended/blob/master/python-ci-server.py). This code can connect to the `kube-api-server` using `kube/config` (you need to configure AWS and all the credentials from `Enable worker nodes to join your cluster` steps on `Stacks` section into the GitLab runner first). The `apply_deployment` function is used to replace the docker image tag with the new generated docker image tag (you need to put your deployment manifest on the GitLab runner) and then apply it to the cluster. The `get_rollout_status` function is used for geting the latest status of deployment proccess. 

curl -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
aws-iam-authenticator help



pip3 install awscli --upgrade --user
aws configure
aws eks update-kubeconfig --name Cluster-dny86QGnO79k
kubectl --kubeconfig=/home/alireza/.kube/config cluster-info
kubectl --kubeconfig=/home/alireza/.kube/config apply -f aws-auth-cm.yaml
kubectl --kubeconfig=/home/alireza/.kube/config get nodes

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

Copy the `ClusterName` output from the previous step into the corresponding `ClusterName` parameter .

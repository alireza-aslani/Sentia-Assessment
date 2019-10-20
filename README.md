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
This [kubernetes-vpc-stack-public-only](https://github.com/alireza-aslani/Sentia-Assessment/blob/master/kubernetes-vpc-stack-public-only.yaml) creates the [Amazon Virtual Private Cloud](https://aws.amazon.com/vpc/) that our Kubernetes cluster will run inside. you can config from 2 to 4 different public subnet in a vpc. there is an extended version that has more features which you can use that for more customization .
> if you want to just see how it works just leave the defaul values and set the stack name to "eks-vpc" . because we are going to use this later 

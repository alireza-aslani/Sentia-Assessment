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

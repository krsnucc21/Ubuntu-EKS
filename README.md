# Ubuntu-EKS


I put the link to a more updated guide for custom AMI
Amazon EKS Sample Custom AMIs: https://github.com/aws-samples/amazon-eks-custom-amis
 
After downloading the code from GitHub, the following changes are needed to build your AMI for Graviton2:
Update VPC and subnet IDs in Makefile
Change the VPC_ID, SUBNET_ID, and AWS_REGION properly. Note that the subnet for a packer instance must support automatic public IP assignment. Please check if the automatic public IP assignment is ON.
Here is an example of the head of Makefile
VPC_ID := vpc-0eccdebe81b47954f
SUBNET_ID := subnet-048d6fdc454a3c3c5
AWS_REGION := us-east-1
The packer configuration files (*.json) support the x86 processor. For Graviton, you need to change the arch and ami variables:
Edit “amazon-eks-node-ubuntu2004.json” as follows:
22,23c22,23
<     "source_ami_arch":"x86_64",
<     "source_ami_name":"ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*",
---
>     "source_ami_arch":"arm64",
>     "source_ami_name":"ubuntu/images/hvm-ssd/ubuntu-focal-20.04-arm64-server-*",
44c44
<       "instance_type":"m5.xlarge",
---
>       "instance_type":"c6g.large",
And, finally run the following make command:
make build-ubuntu2004-1.19
 
I found a more fundamental issue regarding Ubuntu-EKS AMI. The packer script puts ‘jq’ for json data but it installs Intel-based jq which doesn’t work when ‘/etc/eks/ bootstrap.sh’ runs. The bootstrap fails and can’t register this instance to a cluster.
 
To fix this, you can change the code lines in ‘files/functions.sh’ as follows:
vi files/functions.sh
---
"files/functions.sh" 485 lines --54%--
install_jq() {
    #curl -sL -o /usr/bin/jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
    #chmod +x /usr/bin/jq
    apt-get install -y jq
}
---
 
Then, the AMI created by the packer can run and be used to launch a worker node at EKS console.
make build-ubuntu2004-1.19
 

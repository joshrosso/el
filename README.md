## Prerequisites

### Build Lab AMI using Wardroom

Refer to [Wardroom]() repository for updated docs. These steps are known good as of May 8, 2019. You must have packer (1.4.0), ansible (2.5.0), and the AWS CLI installed and configured.

```
git clone https://github.com/heptiolabs/wardroom.git

cd wardroom/packer

packer build \
-var-file aws-us-west-2.json \
-var kubernetes_cni_version=0.7.5-00 \
-var kubernetes_version=1.14.1-00 \
-var build_version=`git rev-parse HEAD` \
--only=ami-ubuntu \
packer.json
```

Example output:

```
==> Builds finished. The artifacts of successful builds are:
--> ami-ubuntu: AMIs were created:
us-west-2: ami-0c893ed431d7cbd55
```

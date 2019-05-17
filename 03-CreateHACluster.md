# Create a HA cluster

## Launch Instances 

Launch three more instances using your SSH keypair and the AMI you created with Packer.

```
aws ec2 run-instances \
--image-id AMI \
--count 3 \
--instance-type m5.large \
--key-name AWS_KEY
```

## Join a worker node

SSH to the worker node and use the kubeadm join command, using the IP address and token from your first controller node:

```
sudo kubeadm join ${CONTROLLER_1_IP}:6443 \
  --token ${KUBEADM_TOKEN} 	\
  --discovery-token-ca-cert-hash ${APISERVER_CA_CERT_HASH}
```

## Join two controllers

SSH to each controller node and use the kubeadm join command, using the IP address, certificate key, and token from your first controller node:

```
sudo kubeadm join ${CONTROLLER_1_IP}:6443 \
    --experimental-control-plane \
    --certificate-key=${ENCRYPTION_KEY} \
    --token ${KUBEADM_TOKEN} \
    --discovery-token-ca-cert-hash ${APISERVER_CA_CERT_HASH}
```

Add the new controllers to your first node's HAProxy configuration, and restart HAproxy so the changes take effect.

## Verify your HA cluster

On the initial controller node:

`kubectl get nodes`

There should be three controllers marked "master" and one worker with no role.
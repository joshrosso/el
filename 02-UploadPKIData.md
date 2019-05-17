# Uploading PKI Data

## Use Kubeadm to upload PKI data

```
sudo kubeadm init phase upload-certs --experimental-upload-certs
```

Make note of the certificate key provided. This will be required when joining another controller to the cluster later.

## Validate the PKI Data

Confirm the PKI data was uploaded to Kubernetes:

`kubectl get secrets kubeadm-certs -n kube-system`

## Verify the TTL

The PKI data expires after two hours. To confirm this, look at the owner of the `kubeadm-certs` secret, which will be a `bootstrap-token`:

```
kubectl get secrets kubeadm-certs \
-n kube-system \
-o jsonpath='{.metadata.ownerReferences[*].name}'

bootstrap-token-2la36n
```

Confirm the expiration, replacing the token in this command with the one present on your system:

```
kubectl get secrets bootstrap-token-2la36n -n kube-system -o jsonpath='{.data.expiration}' | base64 -d
```

The timestamp should be about 2 hours from now.
# Crossplane Environment Composition for platform building

This tutorial focus on creating a basic AWS EKS cluster

This Crossplane Composite resource creates the following resources:
- EKS Cluster
- NodeGroup
<!-- - Helm Provider Config
- Helm Release inside the created cluster -->

## Installation

In an EKS Kubernetes Cluster install the following components.

Let's install [Upbound's Crossplane CLI](https://docs.upbound.io/cli/):

```
curl -sL "https://cli.upbound.io" | sh

```

Download [Upbound Universal Crossplane](https://docs.upbound.io/uxp/install/?ref=upbound-blog#install-upbound-universal-crossplane)

```
# Make sure your ~/.kube/config file points to your cluster

up uxp install

```

Then install the Crossplane Helm provider: 
```
up controlplane \
provider install \
crossplane/provider-helm:v0.10.0
```

<!-- We need to get the correct ServiceAccount to create a new ClusterRoleBinding so the Helm Provider can install Charts on our behalf. 

```
SA=$(kubectl -n crossplane-system get sa -o name | grep provider-helm | sed -e 's|serviceaccount\/|crossplane-system:|g')
kubectl create clusterrolebinding provider-helm-admin-binding --clusterrole cluster-admin --serviceaccount="${SA}"
```

```
kubectl apply -f crossplane/config/helm-provider-config.yaml
```

We also need to install the Crossplane Kubernetes Provider if we want to install custom resources. 

```
kubectl crossplane install provider crossplane/provider-kubernetes:v0.6.0
```

And then configure it (this is only necessary if we are planning to install a kubernetes resource in the cluster where crossplane is installed):

```
kubectl apply -f crossplane/config/kubernetes-provider-config.yaml
``` -->



## Install & Configure GCP provider

I've used the GCP getting started, but GCP provider can be installed in the same way. 


```
up controlplane \
provider install \
xpkg.upbound.io/upbound/provider-aws:v0.21.0
```

Generate an AWS key-pair file: Create a text file (aws-credentials.txt) containing the AWS account aws_access_key_id and aws_secret_access_key

```
[default]
aws_access_key_id = <aws_access_key>
aws_secret_access_key = <aws_secret_key>
```

Create a Kubernetes secret with the AWS credentials

```
kubectl create secret \
generic aws-secret \
-n upbound-system \
--from-file=creds=./aws-credentials.txt
```

Finally

```
cat <<EOF | kubectl apply -f -
apiVersion: aws.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: upbound-system
      name: aws-secret
      key: creds
EOF

```

## Install Environment Composite Resource (XRD)

```
kubectl apply -f eks-claim.yaml
```
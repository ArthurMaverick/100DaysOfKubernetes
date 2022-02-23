# Crossplane

## Compose cloud infrastructure and services into custom platform APIs.


**Crossplane is a open source add-on that enables platform teams to assemble infrastructure from multiples vendors, and expose higer level-service APIs for apllication teams to consume, without habing to write any code.**


## How install with helm

```
kubectl create namespace crossplane-system

helm repo add crossplane-stable https:chars.crossplane.io/stable
helm update

helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
```
## Check Crossplane Status
```
helm list -n crossplane-system
kubectl get all -n crossplane-system
```

## How Install Crossplane CLI
### The Crossplane CLI extends kubectl with funtionality to build, push and intall Crossplane

```
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
```
## Install Crossplane configuration pkg
### this case we use aws configuration
```
kubectl crossplane install configuration registry.upbound.io/xp/getting-started-with-aws:v1.6.3
``` 
**wait until all packages have been installed**
```
watch kubectl get pkg
```

## Get AWS Account Keyfile
```
AWS_PROFILE=default && echo -e "[default]\naws_access_key_id = $(aws configure get aws_access_key_id --profile $AWS_PROFILE)\naws_secret_access_key = $(aws configure get aws_secret_access_key --profile $AWS_PROFILE)" > creds.conf

```

## Create a Provider Secret
```
kubectl create secret generic aws-creds -n crossplane-system --from-file=creds=./creds.conf
```

## Configure the Provider
### we will creathe following ProfiderConfig object to configure credentials for AWS Provider:
```
apiVersion: aws.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials: 
    source: Secret
    secretRef:
      namespace: crossplane-system
      name:aws-creds
      key: creds
```
```
kubectl apply -f https://raw.githubusercontent.com/crossplane/crossplane/release-1.6/docs/snippets/configure/aws/providerconfig.yaml
```

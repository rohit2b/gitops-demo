Demo of managing Confluent Platform with Confluent Operator 2.0 using GitOps.

## Set up Flux CD

On your local dev machine, install Flux

```
brew install fluxcd/tap/flux
```

Once installed, check that the Kubernetes cluster matches the pre-reqs

```
flux check --pre
```

Create namespace

```
 k create ns flux
```

Check out this Github repo

```
git clone git@github.com:rohit2b/gitops-demo.git
```

Set up the Github SSH access

```
# Create known_hosts

ssh-keyscan github.com > ./known_hosts

# Use own private key (./demo-rohit)  and public key (./demo-rohit.pub)
kubectl create secret generic ssh-credentials -n flux \
    --from-file=./demo-rohit \
    --from-file=./demo-rohit.pub \
    --from-file=./known_hosts
```

Deploy Flux Operator for QA

```
flux bootstrap github \
    --context=demo-qa \
    --personal \
    --owner=rohit2b \
    --repository=gitops-demo \
    --branch=main \
    --path=cluster/qa \
    --namespace=flux
```

Deploy Flux Operator for Production

```
flux bootstrap github \
    --context=demo-prod \
    --personal \
    --owner=rohit2b \
    --repository=gitops-demo \
    --branch=main \
    --path=cluster/prod \
    --namespace=flux
```

### Set up Sealed Secrets

https://toolkit.fluxcd.io/guides/sealed-secrets/

```
brew install kubeseal
```

```
flux create source helm sealed-secrets -n flux \
--interval=1h \
--url=https://bitnami-labs.github.io/sealed-secrets
```

```
flux create helmrelease sealed-secrets -n flux \
--interval=1h \
--release-name=sealed-secrets \
--target-namespace=flux \
--source=HelmRepository/sealed-secrets \
--chart=sealed-secrets \
--chart-version="1.13.x"
```

```
kubeseal --fetch-cert \
--controller-name=sealed-secrets \
--controller-namespace=flux \
> pub-sealed-secrets.pem
```

## Deploy Confluent

### Create encrypted secrets

Create a secret for the Root Certificate Authority for TLS certificates

```
kubectl create -n confluent secret tls ca-pair-sslcerts \
--cert=$TUTORIAL_HOME/ca.pem \
--key=$TUTORIAL_HOME/ca-key.pem --dry-run=true -o yaml > ca-pair-sslcerts.yaml

kubeseal --format=yaml --cert=pub-sealed-secrets.pem < ca-pair-sslcerts.yaml > ca-pair-sslcerts-sealed.yaml
```

Create a secret with authentication credentials for components

```
kubectl create secret generic credential \
  --from-file=plain-users.json=$TUTORIAL_HOME/creds-kafka-sasl-users.json \
  --from-file=digest-users.json=$TUTORIAL_HOME/creds-zookeeper-sasl-digest-users.json \
  --from-file=digest.txt=$TUTORIAL_HOME/creds-kafka-zookeeper-credentials.txt \
  --from-file=plain.txt=$TUTORIAL_HOME/creds-client-kafka-sasl-user.txt \
  --from-file=basic.txt=$TUTORIAL_HOME/creds-control-center-users.txt --dry-run=true -o yaml > credential.yaml

kubeseal --format=yaml --cert=pub-sealed-secrets.pem < credential.yaml > credential-sealed.yaml
```

### Deploy Confluent components





------------------

## References

Flux official example: https://github.com/fluxcd/flux2-kustomize-helm-example

Example with CR yamls: https://github.com/gbaeke/realtimeapp-infra

Use SSH access to Github repo:
https://toolkit.fluxcd.io/components/source/gitrepositories/#ssh-authentication


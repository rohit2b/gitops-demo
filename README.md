Demo of managing Confluent Platform with Confluent Operator 2.0 using GitOps.

## Set up

On your local dev machine, install Flux

```
brew install fluxcd/tap/flux
```

Once installed, check that the Kubernetes cluster matches the pre-reqs

```
flux check --pre
```

Check out this Github repo

```
git clone git@github.com:rohit2b/gitops-demo.git
```

Deploy Flux Operator

```
flux bootstrap github \
    --context=demo-qa \
    --owner=rohit2b \
    --repository=gitops-demo \
    --branch=main \
    --path=cluster/qa
```

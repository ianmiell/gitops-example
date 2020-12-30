# Install / Set Up

## Pre-Requisites

You will need:

- A GitHub account

- A Kubernetes cluster, with admin `kubectl` access (see below, 'Kubernetes Cluster')

- `kubectl` installed to your client. See [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

- `fluxctl` installed to your client. See [here](https://github.com/fluxcd/flux/blob/master/docs/references/fluxctl.md)

- `gcloud` installed to your client. See [here](https://cloud.google.com/sdk/install)

Notes:

- Kubernetes: in this walkthrough we assume you are using a GCP Kubernetes cluster. Other options include: minikube, microk8s, EKS, or AKS.

- GitHub: will use the [GitHub Packages](https://github.com/features/packages) service, which has a free tier which should be sufficient for this.

## Walkthrough

1. Get personal access token `GITOPS_EXAMPLE_DEPLOY_TRIGGER`: On GitHub, go to `Settings => Developer Settings => Tick: public_repo`. Note the value, which will be referred to as: `GITOPS_EXAMPLE_DEPLOY_TRIGGER` below
1. Get personal access token `GITOPS_EXAMPLE_DEPLOY_DOCKER_LOGIN`: On GitHub, go to: `Settings => Developer Settings => Tick: read:packages`. Note the value, which will be referred to as: `GITOPS_EXAMPLE_DEPLOY_DOCKER_LOGIN` below
1. Fork the repos: https://github.com/ianmiell/gitops-example-app and https://github.com/ianmiell/gitops-example-deploy (and, optionally, https://github.com/ianmiell/gitops-example-infra if you do not have kubectl access to a cluster, or want that provisioned in a GitOps manner also - see 'Kubernetes Cluster' below)
1. Update `.github/workflows/main.yml` in the `gitops-example-app` and replace `ianmiell` with your GitHub username
1. Create secret `GITOPS_EXAMPLE_DEPLOY_TRIGGER` in your gitops-example-app fork with the value you noted above
1. Create `gitops-example` namespace in your Kubernetes cluster: `kubectl create namespace gitops-example`
1. Set up Docker registry secret in your Kubernetes cluster, remembering to replace `GITOPS_EXAMPLE_DEPLOY_DOCKER_LOGIN` with the personal access token created above: `kubectl create -n gitops-example secret docker-registry regcred --docker-server=docker.pkg.github.com --docker-username=ianmiell --docker-password=GITOPS_EXAMPLE_DEPLOY_DOCKER_LOGIN --docker-email=ian.miell@gmail.com`
1. Set up FluxCD: Instructions are [here](https://github.com/fluxcd/flux/blob/master/docs/tutorials/get-started.md) OR see next section for instructions specific for this workflow

### FluxCD Setup

1. ONLY ON GCP, set up a cluster admin role binding: `kubectl create clusterrolebinding "cluster-admin-$(whoami)" --clusterrole=cluster-admin --user="$(gcloud config get-value core/account)"`
1. Create a flux namespace: `kubectl create ns flux`
1. Set up your GitHub username in a variable: `export GHUSER="YOURGITHUBUSERNAME"`
1. Install flux, check that the `--git-branch` is correct for your repo: `fluxctl install --git-branch main --git-user=${GHUSER} --git-email=${GHUSER}@users.noreply.github.com --git-url=git@github.com:${GHUSER}/gitops-example-deploy --git-path=namespaces,workloads --namespace=flux | kubectl apply -f -
1. Wait for the flux deployment to complete: `kubectl -n flux rollout status deployment/flux`
1. Go to: `https://github.com/YOURGITHUBUSERNAME/gitops-example-deploy/settings/keys/new` and put the output of: `fluxctl identity --k8s-fwd-ns flux` in a key called `flux`. Allow write access.
1. Wait up to 5 minutes
1. Check namespace and app have been deployed

Now you have the infrastructure set up, you can proceed to make a change on the app repo and then wait for it to deploy automatically. It can take up to 5 minutes.

## Check it worked

```
kubectl -n gitops-example deployment/gitops-example-deployment 8000:8000 &
curl localhost:8080
```

OR

Check the flux application logs in the `flux` namespace.

## Kubernetes Cluster

If you want to use the (optional) gitops-example-infra repository to set up a Kubernetes cluster in a GitOps manner, then instructions are below.

### Branches

There are separate branches per cloud provider

`aws` - terraform for AWS

`gcp` - terraform for GCP

### To Run This Up On AWS

- Set up AWS account with appropriate privileges (admin is fine for this example's purposes) to create and administer the resources in this terraform module

- Create AWS key

- `aws configure`, using AWS key created above

- `terraform init`

- `terraform plan`

- `terraform apply`

- Configure your `kubectl` to use the config output by terraform, eg:

```
KUBECONFIG=newfile:~/.kube/config kubectl config view --merge --flatten > newkubeconfig
cp ~/.kube/config ~/.kube/config.$(date +%s)
mv newkubeconfig ~/.kube/config
kubectx   # choose aws context
```

### To Run This Up On GCP

- Create a project, and note the project-id

- Replace the `project` setting in `connections.tf` with your project-id (NOTE: this is NOT the project 'Name', rather the project 'ID')

- `gcloud init` and set up your gcloud client

- `terraform init`

- `terraform apply`

- Go to cluster in console, and click on 'Connect' to get the 'Command-line access' command, which will look similar to: `gcloud container clusters get-credentials cluster-1 --zone ZONE --project PROJECT-ID`

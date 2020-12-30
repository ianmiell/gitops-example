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

## Walkthrough

1. Get personal access token `EXAMPLE_GITOPS_DEPLOY_TRIGGER`: On GitHub, go to `Settings => Developer Settings => Tick: public_repo`. Note the value, which will be referred to as: `EXAMPLE_GITOPS_DEPLOY_TRIGGER` below
1. Get personal access token `EXAMPLE_GITOPS_DEPLOY_DOCKER_LOGIN`: On GitHub, go to: `Settings => Developer Settings => Tick: read:packages`. Note the value, which will be referred to as: `EXAMPLE_GITOPS_DEPLOY_DOCKER_LOGIN` below
1. Fork the repos: https://github.com/ianmiell/example-gitops-app and https://github.com/ianmiell/example-gitops-deploy (and, optionally, https://github.com/ianmiell/example-gitops-infra if you do not have kubectl access to a cluster, or want that provisioned in a GitOps manner also - see 'Kubernetes Cluster' below)
1. Update `.github/workflows/main.yml` in the `example-gitops-app` and replace `ianmiell` with your GitHub username
1. Create secret `EXAMPLE_GITOPS_DEPLOY_TRIGGER` in your example-gitops-app fork with the value you noted above
1. Set up Docker registry secret in your Kubernetes cluster, remembering to replace `EXAMPLE_GITOPS_DEPLOY_DOCKER_LOGIN` with the personal access token created above: `kubectl create -n example-gitops secret docker-registry regcred --docker-server=docker.pkg.github.com --docker-username=ianmiell --docker-password=EXAMPLE_GITOPS_DEPLOY_DOCKER_LOGIN --docker-email=ian.miell@gmail.com`
1. Set up FluxCD: Instructions are [here](https://github.com/fluxcd/flux/blob/master/docs/tutorials/get-started.md)

Now you have the infrastructure set up, you can proceed to make a change on the app repo and then wait for it to deploy automatically.

## Check it worked

```
kubectl -n example-gitops deployment/example-gitops-deployment 8000:8000 &
curl localhost:8080
```

## Kubernetes Cluster

If you want to use the (optional) example-gitops-infra repository to set up a Kubernetes cluster in a GitOps manner, then instructions are below.

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

# Kubestack

Provision a Kubernetes cluster with [Packer](https://packer.io) and [Terraform](https://www.terraform.io) on Google Compute Engine.

## Status

Ready for testing. Over the next couple of weeks the repo should be generic enough for reuse with complete documentation.

## Prep

- [Install Packer](https://packer.io/docs/installation.html)
- [Install Terraform](https://www.terraform.io/intro/getting-started/install.html)
- [Setup an Authentication JSON File](https://www.terraform.io/docs/providers/google/index.html#account_file)

The Packer and Terraform configs assume your authentication JSON file is stored under `/etc/kubestack-account.json`

## Packer Images

Immutable infrastructure is the future. Instead of using cloud-init to provision machines at boot we'll create a custom image using Packer.

Run the packer commands below will create the following image:

```
kubestack-0-0-1-v20150518
```

### Create the Kubestack Base Image

```
cd packer
packer build kubestack.json
```

## Terraform

Terraform will be used to declare and provision a Kubernetes cluster.

### Prep

Generate an [etcd discovery](https://coreos.com/docs/cluster-management/setup/cluster-discovery/) token:

```
curl https://discovery.etcd.io/new?size=3
https://discovery.etcd.io/465df9c06a9d589...
```

Edit `terraform/terraform.tfvars`. Add the required values:

```
discovery_url = "https://discovery.etcd.io/465df9c06a9d589..."
project = "kubestack"
sshkey_metadata = "core: ssh-rsa AAAAB3NzaC1yc2EA..."
```

- Add API tokens to `terraform/secrets/tokens.csv`. See [Kubernetes Authentication Plugins](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/authentication.md) for more details.
- Ensure your local ssh-agent is running and your ssh key has been added. This step is required by the terraform provisioner.

```
ssh-add ~/.ssh/id_rsa
```


### Provision the Kubernetes Cluster

```
cd terraform
terraform plan
terraform apply
```

### Resize the number of worker nodes

Edit `terraform/terraform.tfvars`. Set `worker_count` to the desired value:

```
worker_count = 3
```

Apply the changes:

```
terraform plan
terraform apply
```

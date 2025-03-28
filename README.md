# go-operator
Logs all the event changes in a kubernetes namespace, operator is used debug specific issue or study a particular changes done to the cluster namespace.
## Repository
Create a github repo and clone the repo.
```sh
git clone https://github.com/ManojChandran/chowki
cd go-operator
go mod init github.com/ManojChandran/chowki
go mod tidy
```
## Get started
Run the init command inside of it to initialize a new project, Init command will create basic project structure with all the configurations for a base CRD.
```sh
kubebuilder init --domain  devops.tools --repo devops.tools/controller
```
### config
All our launch configurations under the config/ directory and it contains Kustomize YAML definitions required to launch our controller on a cluster.
* `config/default` contains a Kustomize base for launching the controller in a standard configuration.
* `config/manager` contains a Kustomize base for launching as pods in the cluster.
* `config/rbac`    contains a Kustomize base for required permissions to run your controllers under their own service account

### Makefile
In addition to the configuration `Makefile` does some heavy lifting for us:
* make (Default Target)
* make run (Run Operator Locally)
* make manifests (Generate CRDs & RBAC)
* make install (Apply CRDs to Cluster)
* make build (Build Operator Binary)
* make docker-build (Build Docker Image)
* make docker-push (Push Image to Registry)
* make deploy (Deploy Operator to Cluster)
* make uninstall (Remove CRDs from Cluster)
* make test (Run Unit Tests)
* make fmt (Format Code)
* make vet (Static Analysis)
* make help (Show Available Commands)

### main.go
Our entry point to the operator is `main.go`
* Core controller-runtime library
* Default controller-runtime logging, Zap (more on that a bit later)
* Manager to track of running all of our controllers
* Scheme, which provides mappings between Kinds and their corresponding Go types.
* Basic health checks

### Kubernetes API
Determine group/version for our API.
* Kind - A schema for an object, mapping to a Go type (Capital Letters). 
* Resource - An HTTP enpoint/Path (Lower case plurals)
* Group -  A set of resources that are exposed together
* Version - Unique versions to a group

> API Endpoint - /apis/apps/v1/namespaces/default/deployemnts/nginx
```sh
kubectl api-resources
kubectl api-versions
kubectl get --raw /apis |jq '.'  # api resources with versions
```
Build API and for our, run below command.
```sh
kubebuilder create api --group crd --version v1 --kind Chowki
```
This will create API and Controller for our operator.

## Set up kind cluster for running kubernetes
Manifest file
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
```
Cluster creation 
```sh
kind create cluster --name my-cluster --config kind-cluster.yaml
```
Cluster Deletion
```sh
kind delete my-cluster
```
## Testing
To test the controller built using Kubebuilder with this Makefile, follow these steps:

### Run Static Code Checks
Before running the controller, ensure that the code is formatted correctly and free from obvious issues.
```sh
make fmt       # Format the code
make vet       # Run static analysis on the code
make lint      # Run linting checks
```

### 2. Generate Required Code and Manifests
Generate necessary Kubernetes manifests, RBAC rules, CRDs, and DeepCopy functions.
```sh
make manifests  # Generate CRDs, RBAC, and Webhooks
make generate   # Generate DeepCopy methods
```
> Install custom resource 
> kubectl apply -k config/samples/

### 3. Build and Run the Controller Locally
To test the controller on your host machine (without deploying it in Kubernetes):
```sh
make build  # Build the controller binary
make run    # Run the controller locally
```
> **Note:** Running locally requires access to a Kubernetes cluster, usually via `~/.kube/config`.

### 4. Run Unit Tests
Run tests using `envtest`:
```sh
make test  # Run unit tests with envtest
```

### 5. Deploy the Controller in Kubernetes
If you want to test the controller inside a Kubernetes cluster:
```sh
make docker-build  # Build the Docker image
make docker-push   # Push the image to a registry (if required)

# Deploy CRDs
make install  

# Deploy the controller
make deploy
```
> **Note:** Ensure that the Kubernetes cluster is running and accessible.

## Basic Working
![Alt Text](./img/operator-basic.png)
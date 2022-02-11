# running locally
go run hello.go
http://localhost:8080/api/hello

# building docker image
docker build -t hello-world .

# tagging docker image
docker tag a23196d090a9 gornyj/hello-world:test

# pushing docker image
docker push gornyj/hello-world:test

# applying to cluster
kubectl apply -n demo -f ./k8s/hello-app-deployment.yaml

# connect
kubectl port-forward hello-app-deployment 8080:8080


# flux notes
Prerequisites:
* Kubernetes Cluster
* Valid GitHub personal access token (PAT)

# clear k8s context
unset KUBECONFIG

# create cluster
kind create cluster --name gornyj-k8s
kubectl cluster-info

# Install flux cli
brew install fluxcd/tap/flux

# Export github PAT as env var
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>

# Bootstrap
flux bootstrap github \
--owner=jtgorny \
--repository=flux-infra \
--branch=master \
--path=./k8s/ \
--personal

git clone flux-infra
cd flux-infra

# Create the source to monitor & set interval
flux create source git hello-world \
--url=https://github.com/jtgorny/hello-world-k8s \
--branch=master \
--interval=1m \
--export > ./k8s/sample-source.yaml

git add -A && git commit -m "Add sample app GitRepository"
git push

# Create the kustomize deployment file
flux create kustomization sample \
--target-namespace=default \
--source=hello-world \
--path="./k8s" \
--prune=true \
--interval=5m \
--export > ./k8s/sample-app-kustomization.yaml

git add -A && git commit -m "Add sample app Kustomization"
git push

# Apply a change to the hello-world-k8s deployment on master branch
git add -A && git commit -m "Add sample app Kustomization"
git push

# Remove cluster when finished
kind delete cluster --name gornyj-k8s

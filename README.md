# running locally
go run hello.go
http://localhost:8080/api/hello

# building docker image
docker build -t hello-world .

# tagging docker image
docker tag a23196d090a9 gornyj/hello-world:test

# pushing docker image
docker push gornyj/hello-world:test

# flux notes
Prerequisites:
* Kubernetes Cluster
* Valid GitHub personal access token (PAT)

# Install flux cli
brew install fluxcd/tap/flux

# Export github PAT as env var
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>

# Bootstrap
flux bootstrap github \
--owner=jtgorny \
--repository=hello-world-k8s \
--branch=master \
--namespace=flux-system \
--personal

# Create the source to monitor & set interval
flux create source git sample-app \
--url=https://github.com/jtgorny/hello-world-k8s \
--branch=master \
--interval=1m \
--export > ./out/sample-app-source.yaml

git add -A && git commit -m "Add podinfo GitRepository"
git push

# Create the kustomize deployment file
flux create kustomization sample-app \
--target-namespace=default \
--source=sample-app \
--path="./kustomize" \
--prune=true \
--interval=5m \
--export > ./out/sample-app-kustomization.yaml

# 

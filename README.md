# Tekton setup

```sh
brew install tektoncd-cli

https://medium.com/nontechcompany/build-docker-image-using-tekton-pipeline-buildah-fe62a8f70a75


# pipelines
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

# dashboard
kubectl apply --filename https://github.com/tektoncd/dashboard/releases/latest/download/tekton-dashboard-release.yaml

# triggers
kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml

```

```sh
kubectl create ns cicd

export DOCKER_USERNAME=<DOCKERUSER>
export DOCKER_PASSWORD=<DOCKERPASSWORD>
kubectl create secret generic -n cicd image-push-secrets "--from-literal=username=$DOCKER_USERNAME" "--from-literal=password=$DOCKER_PASSWORD"

```

```sh
# create GITHUB secret for the backend to push changed files
export GITHUB_ORG=thindexed
export GITHUB_REPO=shapes
export GITHUB_TOKEN=<GITHUB-TOKEN>

kubectl create secret generic \
    github-secrets \
    "--from-literal=GITHUB_ORG=$GITHUB_ORG" \
    "--from-literal=GITHUB_REPO=$GITHUB_REPO" \
    "--from-literal=GITHUB_TOKEN=$GITHUB_TOKEN"

```


Install required tasks

```sh
kubectl apply -n cicd -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-cli/0.3/git-cli.yaml
kubectl apply -n cicd -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-version/0.1/git-version.yaml
kubectl apply -n cicd -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/kubernetes-actions/0.2/kubernetes-actions.yaml

```


```
kubectl create secret generic -n cicd kubeconfig --from-file=kubeconfig=./kubeconfig.yaml

```

# open dashboard

```
kubectl proxy 

```


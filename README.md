# Tekton setup

```
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

```
export DOCKER_USERNAME=<DOCKERUSER>
export DOCKER_PASSWORD=<DOCKERPASSWORD>
kubectl create secret generic image-push-secrets "--from-literal=username=$DOCKER_USERNAME" "--from-literal=password=$DOCKER_PASSWORD"

```

Install required tasks

```
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-cli/0.3/git-cli.yaml
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-version/0.1/git-version.yaml
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/buildah/0.2/buildah.yaml
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/kubernetes-actions/0.2/kubernetes-actions.yaml

```


```
kubectl create secret generic kubeconfig --from-file=kubeconfig=./kubeconfig.yaml

```

# open dashboard

```
kubectl proxy 

```

[http://127.0.0.1:8001/api/v1/namespaces/tekton-pipelines/services/tekton-dashboard:http/proxy/#/namespaces/default/pipelineruns](http://127.0.0.1:8001/api/v1/namespaces/tekton-pipelines/services/tekton-dashboard:http/proxy/#/namespaces/default/pipelineruns)
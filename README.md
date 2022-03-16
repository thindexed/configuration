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
kubectl create ns cicd

export DOCKER_USERNAME=<DOCKERUSER>
export DOCKER_PASSWORD=<DOCKERPASSWORD>
kubectl create secret generic -n cicd image-push-secrets "--from-literal=username=$DOCKER_USERNAME" "--from-literal=password=$DOCKER_PASSWORD"

```

Install required tasks

```
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

[http://127.0.0.1:8001/api/v1/namespaces/tekton-pipelines/services/tekton-dashboard:http/proxy/#/namespaces/default/pipelineruns](http://127.0.0.1:8001/api/v1/namespaces/tekton-pipelines/services/tekton-dashboard:http/proxy/#/namespaces/default/pipelineruns)


# Install Istio

```sh
# Create namespace istio-system
#
kubectl create namespace istio-system

# Install Istio operator
#
istioctl operator init --watchedNamespaces=istio-system

``` 


```sh
# Deploy separate IstioOperator CR for istiod
#
kubectl apply -f - <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istio-base
spec:
  profile: minimal
  meshConfig:
    defaultConfig:
      gatewayTopology:
        forwardClientCertDetails: APPEND_FORWARD
      holdApplicationUntilProxyStarts: true
  components:
    pilot:
      k8s:
        replicaCount: 1
EOF

```

```sh
GARDENER_DOMAIN=$(kubectl get configmap -n kube-system shoot-info -o=jsonpath={.data.domain})

# Deploy separate IstioOperator CR ingress gateway
#
kubectl apply -f - <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istio-gateway
spec:
  profile: empty
  components:
    ingressGateways:
      - name: istio-ingressgateway
        enabled: true
        k8s:
          replicaCount: 1
          serviceAnnotations:
              dns.gardener.cloud/class: garden
              dns.gardener.cloud/dnsnames: "*.$GARDENER_DOMAIN, $GARDENER_DOMAIN"
              dns.gardener.cloud/ttl: "600"
              service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: '*'
              service.beta.kubernetes.io/aws-load-balancer-type: nlb    
EOF

kubectl label namespace default istio-injection=enabled --overwrite

kubectl apply -f - <<EOF
apiVersion: cert.gardener.cloud/v1alpha1
kind: Certificate
metadata:
  name: gateway-service-tls
  namespace: istio-system
spec:
  commonName: $GARDENER_DOMAIN
  dnsNames:
    - app.$GARDENER_DOMAIN
  secretRef:
    name: gateway-service-tls
    namespace: istio-system
EOF

kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: gateway-app
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 443
        name: https
        protocol: HTTPS
      tls:
        mode: SIMPLE
        credentialName: gateway-service-tls
      hosts: ["app.$GARDENER_DOMAIN"]
EOF

```

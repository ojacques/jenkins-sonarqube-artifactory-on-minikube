# jenkins-sonarqube-artifactory-on-minikube
An experiment: Jenkins, SonarQube, Artifactory on Minikube, for Katacoda

Install helm 3:

```shell
cd /tmp && wget https://get.helm.sh/helm-v3.5.4-linux-amd64.tar.gz && tar -xzvf helm-v3.5.4-linux-amd64.tar.gz && cp linux-amd64/helm /usr/bin/ && cd -
helm repo add jenkins https://charts.jenkins.io && helm repo update
```

```shell
git clone https://github.com/ojacques/jenkins-sonarqube-artifactory-on-minikube.git
cd jenkins-sonarqube-artifactory-on-minikube
helm install jenkins jenkins/jenkins -f jenkins-values.yaml
```

Get Admin Password:

```shell
kubectl exec --namespace jenkins-project -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/chart-admin-password && echo
```

Get URL:

```shell
export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services jenkins)
export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT/login
```

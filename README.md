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
kubectl exec --namespace default -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/chart-admin-password && echo
```

Get URL:

```shell
export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services jenkins)
export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT/login
```

Test pipeline:

```groovy
podTemplate(label: 'mypod', containers: [
    containerTemplate(name: 'git', image: 'alpine/git', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]
  ) {
    node('mypod') {
        stage('Check running containers') {
            container('docker') {
                // example to show you can run docker commands when you mount the socket
                sh 'hostname'
                sh 'hostname -i'
                sh 'docker ps'
            }
        }
        
        stage('Clone repository') {
            container('git') {
                sh 'whoami'
                sh 'hostname -i'
                sh 'git clone -b master https://github.com/lvthillo/hello-world-war.git'
            }
        }

        stage('Maven Build') {
            container('maven') {
                dir('hello-world-war/') {
                    sh 'hostname'
                    sh 'hostname -i'
                    sh 'mvn clean install'
                }
            }
        }
    }
}
```

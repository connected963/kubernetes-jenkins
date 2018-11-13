# Vagrant environment with Kubernetes and Jenkins

##### This environment was only proposed for development/test purposes

To execute this environment just run
```
vagrant up
```

During build, you can get vm ip on console ```This VM has IP address VM-IP```

In this build process vagrant will install and setup Kubernetes, provide a kubeconfig to access kubernetes without necessity to use ```vagrant ssh```, setup a docker registry, build a Jenkins image based on jdk 11 and deploy it on kubernetes.

## Docker registry

The docker registry is running on port 5000. To deploy a image on kubernetes, it should be on this registry. To push a image to registry you can:

##### Build a image on vagrant using the tag "vagrant:5000/IMAGE-NAME" and push to the registry ```docker push vagrant:5000/IMAGE-NAME```.

##### Create or modify ```/etc/docker/daemon.json``` adding the registry address on "insecure-registries"
```
{
    "insecure-registries":
    [
	"VM-IP:5000"
    ], 
}
```
And then you can push a image builded in your machine to registry running on vagrant ```docker push VM-IP:5000/IMAGE-NAME```.

## Kubernetes credentials

When build finished in the directory of Vagrantfile there will be a credentials.conf to access kubernetes without ssh.

```
kubectl --kubeconfig credentials.conf get pods
```

## Jenkins

The jenkins image is builded with Dockerfile that is inside ```jenkins``` directory, this Dockerfile's image base is ```jenkins/jenkins-experimental:latest-jdk11```

Beside that both jenkins-deployment.yaml and jenkins-service.yaml used to deploy this image on Kubernetes are in ```jenkins``` directory.

##### Jenkins home was mapped to directory ```jenkins/jenkins_home```
 

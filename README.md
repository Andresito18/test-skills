# TEST-Skills

It is an example of an app with orchestration of kubernetes and deployment jenkins (CI/CD).

that is why it is evident in the JENKINSFILE that it uses the following services.

- Container Registry
- Kubernetes Engine
- GCP IAM

Is recommended use minikube in the test the code.

## APP - FLASK

This is a simple app with framework Flask, for run app, run the next command.

```bash
python app/main.py
```

### UNIT TEST

For run unit test you need execute the next command.

```bash
python app/test.py
```

## Work of Docker

This project count with a Dockerfile for work with containers.
For use build image and create container you must run the next commands.

#### Build image

```bash
docker build --tag=flaskservice:1 .
```

#### Create Container

```bash
docker run flaskservice:1
```

## Kubernetes

This project be found a folder with a content deployment of Kubernetes.
For run commands in kuberenetes you start all commands with word

##### kubectl

#### command

```bash
kubectl apply -f ./k8s/deployment.yaml
```

## Jenkins

![jenkins](https://www.google.com/url?sa=i&url=https%3A%2F%2Fwww.pngwave.com%2Fpng-clip-art-fkaag&psig=AOvVaw1vG4ksQNDQu1oQJAosPY6n&ust=1594250262164000&source=images&cd=vfe&ved=0CAIQjRxqFwoTCOCy7JajvOoCFQAAAAAdAAAAABAK)

With jenkins the idea is have three environments like example it was used the cloud GCP. For this the deployment be management starting with metology gitflow of git then in jenkins if deploy a branch to be develop ,qa and master.

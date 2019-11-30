# Monolith to Microservices at Scale


![](https://travis-ci.com/MMSaad/Monolith-to-Microservices-at-Scale.svg?branch=master)



# Build project via docker

Make sure to setup AWS configuration
```bash
aws configure
```

Setup environment varriables file (.env)
```
POSTGRESS_USERNAME=**********
POSTGRESS_PASSWORD=**********
POSTGRESS_DB=**********
POSTGRESS_HOST=**********
AWS_REGION=**********
AWS_PROFILE=**********
AWS_BUCKET=**********
JWT_SECRET=**********
URL=http://localhost:8080
```

import environment varriables
```bash
source .env
```

Build all system docker images

```bash
docker-compose -f udacity-c3-deployment/docker/docker-compose-build.yaml build --parallel
```
Run all system docker images locally
```bash
cd udacity-c3-deployment
docker-compose up
```

# Deploy project to AWS with Kuberenetes & CubeOne


install Terraform 
https://learn.hashicorp.com/terraform/getting-started/install.html

Clone CubeOne repository
https://github.com/kubermatic/kubeone/blob/master/docs/quickstart-aws.md

Unzip CubeOne repository and move to example folders
```bash
cd ./examples/terraform/aws
```

update variiables file as needed (varriables.tf)



Build and publish kuberentes cluster
```bash
source .env
terraform init
terraform plan
terraform apply
terraform output -json > tf.json
kubeone install config.yml --tfjson tf.json

kubectl --kubeconfig=../mustafa-kubeconfig apply -f env-config.yml
kubectl --kubeconfig=../mustafa-kubeconfig apply -f env-secret.yml
kubectl --kubeconfig=../mustafa-kubeconfig apply -f aws-secret.yml

kubectl --kubeconfig=../mustafa-kubeconfig apply -f feed-deployment.yml
kubectl --kubeconfig=../mustafa-kubeconfig apply -f backend-feed-service.yaml

kubectl --kubeconfig=../mustafa-kubeconfig apply -f user-deployment.yml
kubectl --kubeconfig=../mustafa-kubeconfig apply -f backend-user-service.yaml

kubectl --kubeconfig=../mustafa-kubeconfig apply -f reverseproxy-deployment.yml
kubectl --kubeconfig=../mustafa-kubeconfig apply -f reverseproxy-service.yaml

kubectl --kubeconfig=../mustafa-kubeconfig apply -f frontend-deployment.yml
kubectl --kubeconfig=../mustafa-kubeconfig apply -f frontend-service.yaml
```

To forward cluster port to local pc for test
```bash
kubectl --kubeconfig=../mustafa-kubeconfig port-forward service/reverseproxy 8080:8080
kubectl --kubeconfig=../mustafa-kubeconfig port-forward service/frontend 8100:80
```

To scale deployments
```bash
kubectl --kubeconfig=../mustafa-kubeconfig scale deployment/backend-feed --replicas=2
```

To destroy kuberentes cluster
```bash
kubeone reset config.yml --tfjson tf.json
terraform destroy
```



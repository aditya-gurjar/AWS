# AWS
Steps to replicate some of my work as an intern at AWS.

To get familiar with the project details, we will be using [this](https://github.com/GoogleCloudPlatform/microservices-demo) application by deploying it on kubernetes clusters and performing scans on it. This is an optimal application for our use since it uses multiple microservices that interact with each other and the source code of these services is also present in the repository.

The work was done in two parallel tracks, DevSecOps and the Kubernetes Deployment as described below.

## DevSecOps

First we need to install the docker engine. This document describes the work using [docker desktop](https://www.docker.com/products/docker-desktop) but you can replicate the results using other products as well. We aim to deploy kubernetes clusters so we need to [enable kubernetes] in the docker application. More details can be found in [the official documentation](https://docs.docker.com).

Next, we will deploy the microservices-demo application on kubernetes by following the steps given [here](https://github.com/GoogleCloudPlatform/microservices-demo/blob/master/docs/development-guide.md). Note that if you are connected to corporate VPN, then replicating some of the steps may be unsuccessful. 

### Static Application Security Testing (SAST)

Now we will perform some SAST tests on the source code files in the demo application.

#### Node.js

For the source code written in node.js (currencyservice, paymentservice), we will use [njsscan](https://github.com/ajinabraham/njsscan) to scan for vulnerabiliites.
Enter the following commands on terminal. 

```
pip install njsscan
```

```
njsscan filename.js
```

The result will look something like this:

njsscan
<img width="1792" alt="image" src="https://user-images.githubusercontent.com/69450024/115213795-7f7eb300-a11f-11eb-9e68-186512cd80c4.png">

#### Go

For the source code written in Go (frontend, checkoutservice, shippingservice, productservice), we will use [gosec](https://github.com/securego/gosec) to scan for vulnerabiliites. You can follow the multiple methods described in the docs to install and run gosec. 

The results will look something like this:

gosec_1<img width="1614" alt="image" src="https://user-images.githubusercontent.com/69450024/115214080-cbc9f300-a11f-11eb-9109-3a10db5a20bc.png">

gosec_2<img width="1614" alt="image" src="https://user-images.githubusercontent.com/69450024/115214114-d2f10100-a11f-11eb-9c45-9c92c79c2456.png">

gosec_3<img width="1614" alt="image" src="https://user-images.githubusercontent.com/69450024/115214127-d5ebf180-a11f-11eb-9b90-7de6c35c8a36.png">

gosec_4<img width="1614" alt="image" src="https://user-images.githubusercontent.com/69450024/115214143-d8e6e200-a11f-11eb-90f5-0e0d3058baa4.png">

#### Python

For the source code written in Python (emailservice, recommendservice, loadgenerator), we will use [bandit](https://bandit.readthedocs.io/en/latest/) to scan for vulnerabiliites. You can follow the multiple methods described in the docs to install and run bandit. 

The result will look something like this:

bandit_1<img width="1614" alt="image" src="https://user-images.githubusercontent.com/69450024/115214634-590d4780-a120-11eb-93db-ac29aaa10067.png">

bandit_2<img width="1614" alt="image" src="https://user-images.githubusercontent.com/69450024/115214656-5f032880-a120-11eb-844f-e8b3f256782c.png">


#### Java

For the source code written in Python (adservice), we will use [ck](https://github.com/mauricioaniche/ck) to scan for vulnerabiliites. You can follow the methods described in the docs to run the vulnerability scans.


### Dynamic Application Security Testing (DAST)

For DAST scans, we will make use of Zed Attack Proxy (ZAP).
Zed Attack Proxy (ZAP) is a free, open-source penetration testing tool being maintained under the umbrella of the Open Web Application Security Project (OWASP). ZAP is designed specifically for testing web applications and is both flexible and extensible.

You can follow the instructions [here](https://www.zaproxy.org/getting-started/) to install and run the ZAP tool.

The result will look something like this:

ZAP
<img width="1904" alt="image" src="https://user-images.githubusercontent.com/69450024/115223863-e6a16500-a129-11eb-98cb-c696f04c6a49.png">


### Image Signing and Vulnerability Scanning Using Harbor Registry

We will now upload docker images on harbor registry that allows vulnerability scans for images and signing the images for security.

To install harbor registry on your local machine, follow the instructions given [here](https://goharbor.io/docs/2.2.0/install-config/).

If you are using Docker for Mac as your docker engine then you need to do run some additional commands. Docker for Mac runs [differently](https://collabnix.com/how-docker-for-mac-works-under-the-hood/) on the system than the windows version.
The directory structure for the certificates folder will be different and is described in the docs [here](https://docs.docker.com/docker-for-mac/#add-tls-certificates) in the user manual section. [Adding Self-signed Registry Certs to Docker & Docker for Mac](https://blog.container-solutions.com/adding-self-signed-registry-certs-docker-mac) should be followed to add self signed certs for the harbor registry.

Once you are able to host the registry, you can [upload docker images](https://goharbor.io/docs/1.10/working-with-projects/working-with-images/pulling-pushing-images/) and sign them by implementing content trust as described [here](https://goharbor.io/docs/2.2.0/working-with-projects/project-configuration/implementing-content-trust/).

You can perform [vulnerability scans](https://goharbor.io/docs/2.2.0/administration/vulnerability-scanning/) on the image using harbor as well. 

This concludes our work on the DevSecOps track.

## Kubernetes Deployment

We will spin AWS EKS clusters and deploy the [microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo) application. 

First, login to your AWS account and go to Elastic Kubernetes Service. Follow the steps provided [here](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html) to create the clusters using `eksctl`.

Then, deploy the demo app by running the following command on terminal:
```
kubectl apply -f ./release/kubernetes-manifests.yaml
```

You can access the web frontend in a browser using the frontend's `EXTERNAL_IP`
```
kubectl get service frontend-external | awk '{print $4}'
```

*Example output*
```
EXTERNAL-IP
<your-ip>
```

### Polaris

We can now run polaris, it runs a variety of checks to ensure that Kubernetes pods and controllers are configured using best practices, helping you avoid problems in the future. 

We will run it in the dashboard mode. You can choose to run it in any of the methods described [here](https://polaris.docs.fairwinds.com/dashboard/).

The dashboard scores the cluster deployment and gives a grade as well.

### kube-audit

To run kubeaudit on your deployment, first we can install it using Homebrew,
```
brew install kubeaudit
```

Next, we can run
```
kubeaudit all -f "/path/to/manifest.yml"
```
to obtain the auditing reports.

### kube-bench

We can run kube-bench on the EKS cluster by following the steps described [here](https://github.com/aquasecurity/kube-bench#running-in-an-eks-cluster). However, the process is tedious and requires pushing an image to AWS Elastic Container Registry (ECR). 

We can also deploy it using yaml templates by utilising kubernetes jobs by following the steps described [here](https://itnext.io/aws-eks-and-kube-bench-a7ae840f0f1)


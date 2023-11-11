# Kubernetes Deployments

This repository explores the multiple options we have available for managing Kubernetes workloads.

It uses a simple Go application to illustrate how to deploy applications to Kubernetes in various ways. For more information about the application, please refer to my [go-lang-microservices](github.com/guirgouveia/devops-stack/tree/main/dockerize) repository.

The [application's Kubernetes manifests and Helm charts](https://github.com/guirgouveia/go-vuejs-microservices/tree/main/kubernetes) make extensive use of Secrets Operations ( [SOPS](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwiYubm2n7uCAxUGLrkGHYLVBdcQFnoECBQQAQ&url=https%3A%2F%2Fgithub.com%2Fgetsops%2Fsops&usg=AOvVaw1VL2ENXs82bAZnq5jAzeH_&opi=89978449) ) to encrypt the secrets before commiting to the repository, so we will use different techniques to decrypt the secrets either before or during the installation.

This document provides instructions on how to use the Kubernetes configuration files developed for deploying the Go application.

## Kubernetes Resources

The [manifests](./manifests/stack-io/) declare the following Kubernetes resources:

- [Namespace](./manifests/namespace.yaml) The namespace where the application will be deployed.
- [Deployment](./manifests/deployment.yaml) The deployment of the application.
- [Service](./manifests/service.yaml) The service that exposes the application.
- [Ingress](./manifests/ingress.yaml) The ingress that exposes the application to the outside world.
- [Secret](./manifests/secret.yaml) The secret of the application.
- [ConfigMap](./manifests/configmap.yaml) The configuration of the application.
- [PersistentVolumeClaim](./manifests/persistentvolumeclaim.yaml) The persistent volume claim of the application.
- [Job](./manifests/job.yaml) The job that runs the database migrations.
- [CronJob](./manifests/cronjob.yaml) The cron job that runs the database backups.
- [HorizontalPodAutoscaler](./manifests/horizontalpodautoscaler.yaml) The horizontal pod autoscaler that scales the application based on CPU usage.
- [PodDisruptionBudget](./manifests/poddisruptionbudget.yaml) The pod disruption budget that ensures that at least one pod is available at all times.
- [NetworkPolicy](./manifests/networkpolicy.yaml) The network policy that restricts access to the application.
- [PodSecurityPolicy](./manifests/podsecuritypolicy.yaml) The pod security policy that restricts the privileges of the application.
- [Role](./manifests/role.yaml) The role that defines the permissions of the application.
- [RoleBinding](./manifests/rolebinding.yaml) The role binding that binds the role to the service account.
- [ServiceAccount](./manifests/serviceaccount.yaml) The service account that is used by the application.
- [PodSecurityPolicy](./manifests/podsecuritypolicy.yaml) The pod security policy that restricts the privileges of the application.

The Deployment includes:

- **Init Containers:** perform setup tasks before the application starts.
- **Readiness Probe:** checks if the application is ready to receive traffic.
- **Liveness Probe:** checks if the application is alive.
- **Resource Requests and Limits:** defines the minimum and maximum amount of resources that the application can use.
- **Lifecycle Hooks:**
    - **Post Start Hook:** executed immediately after the container is created.
    - **Pre Stop Hook:** executed immediately before the container is terminated.

## Getting Started

The instructions below provide a step-by-step guide for setting up the application in a local Kubernetes cluster.

Both Helm Charts and Kubernetes Manifests are declared to deploy the application. The Helm Charts are located in the [helm](./helm) folder and the Kubernetes Manifests are located in the [manifests](./manifests) folder for each app of the stack.

## Prerequisites

- [Minikube](https:**//minikube.sigs.k8s.io/docs/start/) ( optional )
- [kubectl](https:**//kubernetes.io/docs/tasks/tools/install-kubectl/)
- [MySQL Server](https:**//artifacthub.io/packages/helm/bitnami/mysql)
- [SOPS](https:**//github.com/getsops/sops)
- [Helm](https:**//helm.sh/docs/helm/helm_install/)
- [Helmfile](https:**//helmfile.readthedocs.io)
- [Skaffold](https:**//skaffold.dev/docs/install/#standalone-binary)
- [Kustomize](https:**//kubectl.docs.kubernetes.io/installation/kustomize/)
- [Docker](https:**//docs.docker.com/engine/install/)
- [Docker Compose](https:**//docs.docker.com/compose/install/linux/)
- [Go](https:**//go.dev/doc/install)

## Instructions

> Remember to use [Minikube Docker Daemon](https://minikube.sigs.k8s.io/docs/handbook/pushing/) and `imagePullPolicy: IfNotPresent`, before building images locally, by running `eval $(minikube docker-env)`, to make them available to the cluster. 
> 
> Or use Docker-Desktop instead, as it has a built-in Kubernetes cluster.

1. **Start the cluster**: Start your local Minikube cluster by running the command `minikube start`, or allow the Docker Desktop to use the built-in Kubernetes cluster.

2. **Deploy the reources**:

    The resources can be deployed both with kubectl or Helm with the following instructions:
    #### **2.1 Deploying with Helm**
 
    Here we have many options to deploy the Helm Chart, defining the credentials for MySQL server safely, without exposing them in the values.yaml. Only one of them is used in this repository, but I will list all of them below:
    
    - Deploy the [Bitnami MySQL Helm Chart](https:**//artifacthub.io/packages/helm/bitnami/mysql) manually encrpyting/decrypting SOPS to encrypt/decrypt the file and store it safely in the repository.
    - Deploy the Helm chart with `helm secrets` plugin to decrypt the secrets before deploying the Helm Chart.
    - Use Kustomize with `helm secrets` plugin to download the Helm Charts and then patch the templated Helm Chart, for example, to add environment overlays or Secret and ConfigMap generators.
    - Deploy with Skaffold and `helm secrets` plugin
    - Define a helmfile.yaml with `helm secrets` to declare a Helm releases desired state in a declarative way.
    - Use Kustomize with `helmfile` to download the Helm Charts and then patch the templated Helm Chart, for example, to add environment overlays or Secret and ConfigMap generators.
    - Use Kustomize with Helm to download the Helm Charts and then patch the templated Helm Chart, for example, to add environment overlays or Secret and ConfigMap generators.

    1. **Deploy the manifests**: Analogous to the previous step, you can deploy the Helm Charts from my OCI registry with the following command: 

        ```bash
        helm upgrade --install stack-io oci://registry-1.docker.io/guirgouveia/stack-io -v ./mysql-values.yaml --create-namespaces -n stack-io
        ```
        or deploy it with `kubectl kustomize` running:
        
        ```bash
        kubectl apply -k stack-io
        ```

    The Helm Charts are located in the [helm](./helm) folder and the Kubernetes Manifests are located in the [manifests](./manifests) folder for each app of the stack. The `kustomization.yaml` files are responsible for declaring the resources to be deployed and the automatic generation of Secrets and ConfigMaps. 

    The Helm Charts are continuously built, packaged and pushed to my OCI registry at DockerHub using GitHub Actions, in order to provide the latest stable version of the application. 

    In the Development stage, you can install the Helm Charts directly from the local files by running the following command:

            helm install stack-io stack-io/helm/stack-io

    2. **Verify the Deployment and Service**: You can verify that the Deployment and Service were created successfully by running the following commands:

        ```bash
        kubectl get deployments -n stack-io
        kubectl get services -n stack-io
        ```

    3. **Access the Application**: If everything was set up correctly, you should be able to access the application by forwarding a port from your local machine to the Service in the cluster:

        ```
        kubectl port-forward svc/stack-io 8083:8080 -n stack-io
        ```

    The application should now be accessible at http://localhost:8083.

    ### Using Skaffold for Development

    To use Skaffold for development, make sure to install [Skaffold](https://skaffold.dev/docs/install/) and [kubectl 1.14 or higher](https://kubernetes.io/docs/tasks/tools/install-kubectl/) first.

    Then, run the following command to create the pipeline that will do the local CI/CD:

        ```bash
        skaffold dev --keep-running-on-failure=true
        ```

    This will create a pipeline that will watch for changes in the source code, as well as in the Kubernetes manifests, and continuously build and push the image to the remote registry, and deploy the app to the specified Kubernetes cluster.

    After making changes to the Kubernetes manifests or source code, hit Enter in the terminal to restart the pipeline.

    The application should now be accessible at http://localhost:8084, as Skaffold automatically creates a port-forward.

## Prerequisites

Before you begin, you will need to install the following tools:

- **-[Minikube](https:**//minikube.sigs.k8s.io/docs/start/)
- **-[kubectl](https:**//kubernetes.io/docs/tasks/tools/install-kubectl/)

## Installation

To install the application, follow these steps:

1. Start your local Minikube cluster by running the command `minikube start`.
2. Apply the `namespace.yaml` file to create a new namespace in your cluster:

    ```bash
    kubectl apply -f namespace.yaml
    ```

3. Apply the `app.yaml` file to create a new Deployment and Service in your cluster:

    ```bash
    kubectl apply -f app.yaml
    ```

4. Verify that the Deployment and Service were created successfully by running the following commands:

    ```bash
    kubectl get deployments -n stack-io
    kubectl get services -n stack-io
    ```

5. Forward a port from your local machine to the Service in the cluster:

    ```bash
    kubectl port-forward svc/stack-io 8083:8080 -n stack-io
    ```

6. Access the application by navigating to http://localhost:8083 in your web browser.

## Usage

To use the application, simply navigate to http://localhost:8083 in your web browser. The application should be up and running.

## Future Development

In the future, Kubernetes configuration files will be updated to improve the application's performance and scalability.

# Task 2: Kubernetes
### Exercise Goals

* Install minikube;
* Create namespace;
* Create deployment;
  * Use the golang webserver image you built in the previous step;
  * Add readiness/livess probe;
  * Add prestophook;
  * add init container that sleep for 30 seconds;
* Create service to expose your pod;

### Expected Output

Please, provide us with a file named `namespace.yaml` you are going to create. Your `namespace.yaml` is supposed to:
* Contain the following Kubernetes Resources you are going to create in your `minikube` cluster:
  * Namespace specification;

Please, provide us with a file named `app.yaml` you are going to create. Your `app.yaml` is supposed to:
* Contain the following Kubernetes Resources you are going to create in your `minikube` cluster:
  * Deployment specification;
    * Use your new image created on the [Task 1](../dockerize) in your deployment;
  * Service specification;

[Optional] You can also share screenshots of your progress.

### Next steps?

Once you complete this task, you can proceed to the [Terraform](../terraform) task;

# SOPS ( Secrets OPerationS )

SOPS is an editor of encrypted files that supports YAML, JSON, ENV, INI and BINARY formats and encrypts with AWS KMS, GCP KMS, Azure Key Vault and PGP.

# Go Application

This repository uses a sample go application to illustrate how to deploy applications to Kubernetes in various ways.

You can find the source code of this application at [golang-webapp](https://github.com/guirgouveia/devops-stack/tree/main/dockerize).

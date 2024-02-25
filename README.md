<br />
<p align="center">
    <img src="/mars.png"/>
    <h3 align="center">Kuiper Deployment</h3>
    <p align="center">
        Kuiper microservice for deploying workstations.
    </p>
</p>

## Dependencies

- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/)

## Usage

Kuiper allows for the deployment of workstations dynamically on a kubernetes cluster. It is designed to be a horizontally
scaled microservices that provides High Availability and Redundancy. Deployment is handled by ArgoCD and Helm. The configuration
is handled by a values file that is stored in a private repository. This allows for the configuration to be managed separately
per project.

```yaml
mars:
  image:                            # Registry to pull the Kuiper server image from
  namespace: production             # Namespace to deploy the Kuiper server to

  node_selector:                   # Node selector to deploy the Kuiper server to
    enable: true                   # Enable node selector
    labels:                        # Node selector to deploy the Kuiper server to
      kubernetes.io/arch: amd64


  autoscaling:
    min_replicas: 1                 # Minimum number of replicas to run
    max_replicas: 3                 # Maximum number of replicas to run

  workstations:                     # List of workstation templates to deploy
#    - registry:                    # Registry to pull the workstation image from
#      icon:                        # Icon to display for the workstation
#      label:                       # Label to display for the workstation
#      group:                       # Group to display the workstation under
#      tag:                         # Tag to pull the workstation image from
#      repo:                        # Repository to pull the workstation image from
#      cpu:                         # CPU to allocate to the workstation
#      ram:                         # RAM to allocate to the workstation
#      name:                        # Name of the workstation
#      gpu:                         # GPU to allocate to the workstation

```

### ArgoCD Application

Ideally, you will use an Application in ArgoCD to deploy Kuiper. Below is an example of how to set one up.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: kuiper
  namespace: argocd
spec:
  project: <your project>
  destination:
    server: <target cluster>
    namespace: argocd
  
  # We use the multi source approach to allow for the use of a private values repo
  # https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/
  sources:
    - path: .
      repoURL: https://github.com/juno-fx/Kuiper-Deployment.git
      targetRevision: <the version of Kuiper you want to use>
      helm:
        valueFiles:
          - $values/kuiper.yaml

    - repoURL: https://<your values repo>.git
      targetRevision: <branch>
      ref: values
```

Once ArgoCD has synchronized the application, you should see the Kuiper server ready to go. 
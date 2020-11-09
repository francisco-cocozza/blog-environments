# blog-environments
Repository declaratively defining the state of the environments

This repo follows the Helm Dependency Patter
[https://github.com/argoproj/argocd-example-apps/tree/master/helm-dependency](https://github.com/argoproj/argocd-example-apps/tree/master/helm-dependency)

It proposes this structure **per environment**:
```
├── app-1
│   ├── Chart.yaml
│   └── values.yaml
├── app-2
│   └── ...
├── ...
└── app-N
    └── ...
```

Each environment will N directories, each directory represents an app in that environment
Inside each *"App directory"*, there will be a Helm Chart. It will be just *proxy-chart* pointing to the real Helm Chart to deploy. This levarages the dependency resolution capabilities of Helm.
The `values.yaml` file will have the values to apply for that instance of the app in specific

There is no specific approach to define multiple environments, following that structure.
Some possible options could be:
- One repo per environment
- One repo for all environments and a directory per environment
- One repo for all the environments and a branch per environment.

This repo implements the second strategy.
The repo will 

In general:
```
├── env-1
|   ├── app-1
|   │   ├── Chart.yaml
|   │   └── values.yaml
|   ├── app-2
|   │   └── ...
|   ├── ...
|   └── app-N
|       └── ...
├── env-2
│   └── ...
├── ...
└── env-N
    └── ...
```

For this case in specific:

```
├── production
│   ├── Chart.yaml
│   ├── templates
│   │   ├── app-1.yaml
│   │   ├── app-2.yaml
│   └── values.yaml
└── staging
    ├── Chart.yaml
    ├── templates
    │   ├── app-1.yaml
    │   └──  app-2.yaml
    └── values.yaml
```

## How to start
For each environment `<env>` create a new dir and Create the proxy-chart with the values to apply

## How to add an app
Create a new ArgoCD app manifest in `/<env>/templates/`. To override the values used for the App Helm Chart, add them to `/<env>/templates/values.yaml`. In this way:
```yaml
<argocd-app-name>:
  <values-here>
```

For example, to a new app called `my-new-app`:
```yaml
...
my-new-app:
  attibute1: value1
...
```

## How to add an environment (`<env>`)
Create a new directory at the root, with the name of the environment (e.g.: `staging`) and follow the same structure of files:
```
my-new-env
├── argocd-app-of-apps.yaml
├── argocd-project.yaml
└── helm
    ├── Chart.yaml
    ├── templates
    │   ├── app-x.yaml
    │   └── app-y.yaml
    └── values.yaml
```

## How to remove the objects created
For each environment `<env>`
1. Delete the App of Apps (the main ArgoCD app) for `<env>`. Since the ArgoCD app and the apps inside it have `prune` enabled. 
2. Delete the Argo CD project for `<env>`
3. Delete the namespaces: `<env>`
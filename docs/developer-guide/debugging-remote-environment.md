# Debugging Remote ArgoCD Environment
In this artical we will describe how to debug a remote environment with [Telepresence](https://telepresence.io/).
Telepresence allows to connect & debug a service deployed in a remote environment and to "cherry-pick" one service to run locally, staying connected to the remote cluster. This will:
* Reduce resource footprint on local machine
* Increase feedback loop time
* Gain more confidance about the delivered code.
To read more about it refer to official documentation on [telepresence.io](https://telepresence.io/) or [Medium](https://medium.com/containers-101/development-environment-using-telepresence-634bd7210c26)

## Install ArgoCD
First of all install ArgoCD on you cluster
```shell
kubectl create ns argocd
curl -sSfL https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml | k apply -n argocd -f -
```

## Connect
Connect to one of the services, for example debug the main ArgoCD server run:
```shell
telepresence --swap-deployment argocd-server --namespace argocd --env-file .envrc.remote --expose 8080:8080 --expose 8083:8083 --run bash
```
* `--swap-deployment` changes the argocd-server deployment
* `--expose` forwarf trafic comes to ports 8080 and 8083 to the same ports locally
* `--env-file` write all the environment variables of the remote pod into local file, the variables also set on the subprocess of the `--run` command
* `--run` a command to run once a connected established, use `bash`, `zsh` or others


## Debug
Once a connection established, use favourite tools to start the server locally.

### Terminal
* Compile `make server`
* Run `./dist/argocd-server`

### VSCode
In VSCode use the integrated terminal to run Telepresence command to connect. Then, to run argocd-server service use below configuration.
Make sure to run `packr` before starting the debug to generated the assest. 
Update the configuration file ot point to kubeconfig file: `KUBECONFIG=` (required)
```json
        {
            "name": "Launch",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${workspaceFolder}/cmd/argocd-server",
            "envFile": [
                "${workspaceFolder}/.envrc.remote",
            ],
            "env": {
                "CGO_ENABLED": "0",
                "KUBECONFIG": "/path/to/kube/config"
            }
        }
```
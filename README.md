# run-rabbitmq

Requirements:

* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
* [Operator Lifecycle Manager](https://github.com/operator-framework/operator-lifecycle-manager)
* [yq](https://github.com/mikefarah/yq?tab=readme-ov-file#install)

That last one is more of a recommendation than a hard requirement, but anyway.

Get an appropriate
[kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
file, then switch to appropriate Kubernetes context. Here, our imaginary Kubernetes cluster is named
"yoyodyne-ops":

```console
kubectl config get-contexts
kubectl config use-context yoyodyne-ops
kubectl config get-contexts
kubectl cluster-info
kubectl get all -A
```

Install OLM:

```console
operator-sdk olm install --version v0.28.0
operator-sdk olm status
kubectl events -A -w
```

Create a Namespace, and install the operator:

```console
kubectl create namespace rabbitmq
kubectl apply -f operatorgroup.yaml
kubectl apply -f subscription.yaml
```

> At this point it should be mentioned that this kind of "manual" approach to installation
> of anything is likely not how things should look like in environments even remotely
> resembling "production".

Look at those Kubernetes events, then get the RabbitMQ installation going, and watch
more events:

```console
kubectl events -A -w
kubectl apply -f rabbitmq.yaml
kubectl events -A -w
```

Check out the Pods and follow RabbitMQ's logs:

```console
kubectl get pods -n rabbitmq
kubectl logs -n rabbitmq -l app.kubernetes.io/name=rabbitmq --follow
```

Fish out the credentials for RabbitMQ's default user, then forward local ports to RabbitMQ proper
and its web management console, through RabbitMQ's service:

```console
kubectl get secret -n rabbitmq rabbitmq-default-user -o yaml | yq '.data.username' | base64 -d
kubectl get secret -n rabbitmq rabbitmq-default-user -o yaml | yq '.data.password' | base64 -d
kubectl port-forward -n rabbitmq service/rabbitmq 5672:5672 15672:15672
```

Then for example access the web management console by pointing your browser to:

> http://localhost:15672/

Note: with those `kubectl port-forward`s in place, RabbitMQ clients should talk either to `localhost:5672`,
in case they can access the host's network directly, or to `host.docker.internal:5672` in case they're
running in a Docker container.

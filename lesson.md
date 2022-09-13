# Kubernetes Networking Lab

Welcome to the Kubernetes Networking lab exercise. The following steps will
provide a tour of the necessary features of Kubernetes networking and review
the important details to remember.

## Assumptions and Preconditions

For this exercise, we assume that you have access to a Kubernetes cluster with
basic functionality.

A cloud provider Kubernetes or a local installation of
[Kind](https://github.com/kubernetes-sigs/kind) will work fine for this lab.

## A sample application

In order to demonstrate networking functionality, we will start with a simple
application to have something to access.  The current repository includes a
basic web server (Nginx) that can serve up a sample page.
We will first deploy the sample application to Kubernetes. This deployment
includes a utility sidecar container we can
use for some additional steps.

```shell
kubectl apply -f nginx-deployment.yaml
```

Fetch your deployment and pods to examine the deployment as it comes up.

```shell
kubectl get pods,deployments
```

When the pods have reached ```Running``` status you are ready to continue.

## Exploring Pod Networking

Let's open a shell into the utility container and check the web content. In the
following snippet, we extract the name of the pod to a variable, but you can
also just use your pod name in the following exercises (which you can obtain
with a simple ```kubectl get pods``` command).

```shell
export MYPOD="$(kubectl get pods -l app=nginx | tail -1 | cut -d' ' -f 1)"
```

We execute a shell in the utility container with an exec command.

```shell
kubectl exec -it $MYPOD -c utility -- /bin/sh
# ( from the resulting shell prompt)
# Request via localhost:
curl http://localhost
# Then exit the shell with Ctrl+D or:
exit
```

Note that the utility container can refer to the Nginx server directly as
localhost.

## Creating a Service

A sample of a service to expose the application we deployed previously is
included in this repo.  Check out the service definition:

```shell
cat frontend-service.yaml
```

Now, let's expose the Nginx webserver as a service.

```shell
kubectl apply -f frontend-service.yaml
```

You can view the details of the created service with a get command:

```shell
kubectl describe svc frontend
```

Also, to ensure that the pods are aware of the service, let's recreate the
pods to force them to inherit new environment variables.

```shell
kubectl scale deployment nginx-deployment --replicas=0; kubectl scale deployment nginx-deployment --replicas=1;
```

## Accessing a Service

After the deployment has finished the scaling operation, you will likely need
to recreate the variable that points to the pod.

```shell
export MYPOD="$(kubectl get pods -l app=nginx | tail -1 | cut -d' ' -f 1)"
 ```

You can then fetch the environment from the utility container to see the
service is now available to the pod. (Outside of a lab, we would have created
the service before the dependent pods to avoid the need to recreate pods above.)

```shell
export MYPOD="$(kubectl get pods -l app=nginx | tail -1 | cut -d' ' -f 1)"
kubectl exec -it $MYPOD -c utility -- /bin/sh
# ( from the resulting shell prompt)
echo "The IP of service is $FRONTEND_SERVICE_HOST"
curl http://$FRONTEND_SERVICE_HOST
 ```

## In Conclusion

In this lab, you have deployed an application and created a simple service that
targets that application.  You can see how services can be used to expose and
find running containers wherever they may be needed.

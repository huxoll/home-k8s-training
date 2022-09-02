# Kubernetes Networking Lab

Welcome to the Kubernetes Networking lab exercise. The following steps will
provide a tour of the necessary features of Kubernetes networking and review
the important details to remember.

## A sample application

Deploy the sample application included in this repository. This deployment
includes a standard Nginx web server and a utility sidecar container we can
use for some additional steps.

```shell
kubectl apply -f nginx-deployment.yaml
```

Shell into the utility container and check the web content. In the following
snippet, we extract the name of the pod to a variable, but you can also just
use your pod name here.

```shell
export MYPOD="$(kubectl get pods -l app=nginx | tail -1 | cut -d' ' -f 1)"
kubectl exec -it $MYPOD -c utility -- /bin/sh
# ( from the resulting shell prompt)
curl http://localhost
 ```

Note that the utility container can refer to the Nginx server directly as
localhost.

Now, let's expose the Nginx webserver as a service.

```shell
kubectl apply -f frontend-service.yaml
```

Finally, to ensure that the pods are aware of the service, let's recreate the
pods to force them to inherit new environment variables.

```shell
kubectl scale deployment nginx-deployment --replicas=0; kubectl scale deployment nginx-deployment --replicas=1;
```

After the deployment has finished the scaling operation, you will likely need
to recreate the variable that points to the pod.  You can then fetch the
environment from the utility container to see the service is now available to
the pod. (Outside of a lab, this would be more likely useful to other pods that
need to consume this service.)

```shell
export MYPOD="$(kubectl get pods -l app=nginx | tail -1 | cut -d' ' -f 1)"
kubectl exec -it $MYPOD -c utility -- /bin/sh
# ( from the resulting shell prompt)
curl http://$FRONTEND_SERVICE_HOST
 ```


# Lab 1. Deploy with Helm

Let's investigate how Helm can help us focus on other things by letting a chart do the work for us. We'll first deploy an application to a Kubernetes cluster by using `kubectl` and then show how we can offload the work to a chart by deploying the same app with Helm.

The application is the [Guestbook App](https://github.com/IBM/guestbook), which is a sample multi-tier web application.

## Deploy the application using Helm

In this part of the lab, we will deploy the application by using Helm. We will set a release name of `guestbook-demo`. The Helm chart is available [here](../../charts/guestbook). Clone the [Helm 101](https://github.com/remkohdev/helm101) repo to get the files:

```console
git clone https://github.com/remkohdev/helm101
```

A chart is defined as a collection of files that describe a related set of Kubernetes resources. We probably then should take a look at the the files before we go and install the chart. The files for the `guestbook` chart are as follows:

```text
.
├── Chart.yaml    \\ A YAML file containing information about the chart
├── LICENSE       \\ A plain text file containing the license for the chart
├── README.md     \\ A README providing information about the chart usage, configuration, installation etc.
├── templates     \\ A directory of templates that will generate valid Kubernetes manifest files when combined with values.yaml
│   ├── _helpers.tpl               \\ Template helpers/definitions that are re-used throughout the chart
│   ├── guestbook-deployment.yaml  \\ Guestbook app container resource
│   ├── guestbook-service.yaml     \\ Guestbook app service resource
│   ├── NOTES.txt                  \\ A plain text file containing short usage notes about how to access the app post install
│   ├── redis-master-deployment.yaml  \\ Redis master container resource
│   ├── redis-master-service.yaml     \\ Redis master service resource
│   ├── redis-slave-deployment.yaml   \\ Redis slave container resource
│   └── redis-slave-service.yaml      \\ Redis slave service resource
└── values.yaml   \\ The default configuration values for the chart
```

Note: The template files shown above will be rendered into Kubernetes manifest files before being passed to the Kubernetes API server. Therefore, they map to the manifest files that we deployed when we used `kubectl` or `oc` (minus the helper and notes files).

Let's go ahead and install the chart now. If the `helm-demo` namespace does not exist, you will need to create it using:

```console
oc new-project helm-demo
```

1. Install the app as a Helm chart:

   ```console
   $ cd helm101/charts

   $ helm install guestbook-demo ./guestbook/ 
    NAME: guestbook-demo
    LAST DEPLOYED: Thu Mar 25 20:54:23 2021
    NAMESPACE: helm-demo
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    1. Get the application URL by running these commands:
      NOTE: It may take a few minutes for the LoadBalancer IP to be available.
            You can watch the status of by running 'kubectl get svc -w guestbook-demo --namespace helm-demo'
      export SERVICE_IP=$(kubectl get svc --namespace helm-demo guestbook-demo -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
      echo http://$SERVICE_IP:3000
   ```

   The chart install performs the Kubernetes deployments and service creations of the redis master and slaves, and the guestbook app, as one. This is because the chart is a collection of files that describe a related set of Kubernetes resources and Helm manages the creation of these resources via the Kubernetes API.

   Check the deployment:

   ```console
   oc get deployment guestbook-demo
   ```

   You should see output similar to the following:

   ```console
   $ oc get deployment guestbook-demo
   NAME             READY   UP-TO-DATE   AVAILABLE   AGE
   guestbook-demo   2/2     2            2           51m
   ```

   To check the status of the running application pods, use:

   ```console
   oc get pods
   ```

   You should see output similar to the following:

   ```console
   $ oc get pods
   NAME                            READY     STATUS    RESTARTS   AGE
   guestbook-demo-6c9cf8b9-jwbs9   1/1       Running   0          52m
   guestbook-demo-6c9cf8b9-qk4fb   1/1       Running   0          52m
   redis-master-5d8b66464f-j72jf   1/1       Running   0          52m
   redis-slave-586b4c847c-2xt99    1/1       Running   0          52m
   redis-slave-586b4c847c-q7rq5    1/1       Running   0          52m
   ```

   To check the services, use:

   ```console
   oc get services
   ```

   ```console
   $ oc get services
    NAME             TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)          AGE
    guestbook-demo   LoadBalancer   172.21.53.13     169.46.30.44   3000:30701/TCP   3m6s
    redis-master     ClusterIP      172.21.242.242   <none>         6379/TCP         3m6s
    redis-slave      ClusterIP      172.21.238.104   <none>         6379/TCP         3m6s
   ```

1. Test the guestbook:

   1. Locate the external IP and the port of the load balancer by following the "NOTES" section in the install output. The commands will be similar to the following:

      ```console
      export SERVICE_IP=$(oc get svc guestbook-demo -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
      export NODEPORT=$(oc get svc guestbook-demo -o jsonpath='{.spec.ports[0].nodePort}')
      curl http://$SERVICE_IP:$NODEPORT/rpush/guestbook/hi2
       ```

   2. Navigate to the output given (for example `http://50.23.5.136:31367`) in your browser. You should see the guestbook now displaying in your browser:

       ![Guestbook](../images/guestbook-page.png)

## Conclusion

Congratulations, you have now deployed an application by using two different methods to Kubernetes! From this lab, you can see that using Helm required less commands and less to think about (by giving it the chart path and not the individual files) versus using `kubectl`. Helm's application management provides the user with this simplicity.

Move on to the next lab, [Lab2](../Lab2/README.md), to learn how to update our running app when the chart has been changed.

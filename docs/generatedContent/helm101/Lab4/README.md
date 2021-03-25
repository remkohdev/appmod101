# Lab 4. Share Helm Charts

A key aspect of providing an application means sharing with others. Sharing can be direct counsumption (by users or in CI/CD pipelines) or as a dependency for other charts. If people can't find your app then they can't use it.

A means of sharing is a chart repository, which is a location where packaged charts can be stored and shared. As the chart repository only applies to Helm, we will just look at the usage and storage of Helm charts.

## Using charts from a public repository

Helm charts can be available on a remote repository or in a local environment/repository. The remote repositories can be public like [Bitnami Charts](https://github.com/bitnami/charts) or [IBM Helm Charts](https://github.com/IBM/charts), or hosted repositories like on Google Cloud Storage or GitHub. Refer to [Helm Chart Repository Guide](https://helm.sh/docs/topics/chart_repository/) for more details. You can learn more about the structure of a chart repository by examining the [chart index file](https://raw.githubusercontent.com/IBM/helm101/master/repo/stable/index.yaml) in this lab.

In this part of the lab, we show you how to install the `guestbook` chart from the [Helm101 repo](https://remkohdev.github.io/helm101/).

1. Check the repositories configured on your system:

   ```console
   helm repo list
   ```

   The output should be similar to the following:

   ```console
   $ helm repo list
   Error: no repositories to show
   ```

   > Note: Chart repositories are not installed by default with Helm v3. It is expected that you add the repositories for the charts you want to use. The [Helm Hub](https://hub.helm.sh) provides a centralized search for publicly available distributed charts. Using the hub you can identify the chart with its hosted repository and then add it to your local respoistory list. The [Helm chart repository](https://github.com/helm/charts) like Helm v2 is in "maintenance mode" and will be deprecated by November 13, 2020. See the [project status](https://github.com/helm/charts#status-of-the-project) for more details.

1. Add `helm101` repo:

   ```console
   helm repo add remkohdev-helm101 https://remkohdev.github.io/helm101/repo/stable/
   ```

   Should generate an output as follows:

   ```console
   $ helm repo add remkohdev-helm101 https://remkohdev.github.io/helm101/repo/stable/
    "remkohdev-helm101" has been added to your repositories
   ```

   You can also search your repositories for charts by running the following command:

   ```console
   helm search repo remkohdev-helm101
   ```

   ```console
   $  helm search repo remkohdev-helm101
    NAME                    CHART VERSION   APP VERSION     DESCRIPTION                                     
      
    helm101/guestbook       0.2.0                           A Helm chart to deploy Guestbook three tier web.
    ..
   ```

1. Install the chart

   As mentioned we are going to install the `guestbook` chart from the [Helm101 repo](https://ibm.github.io/helm101/). As the repo is added to our local respoitory list we can reference the chart using the `repo name/chart name`, in other words `helm101/guestbook`. To see this in action, you will install the application to a new namespace called `repo-demo`.

   If the `repo-demo` namespace does not exist, create it using:

   ```console
   oc create namespace repo-demo
   ```

   Now install the chart using this command:

   ```console
   helm install guestbook-demo remkohdev-helm101/guestbook
   ```

   The output should be similar to the following:

   ```console
   $ helm install guestbook-demo remkohdev-helm101/guestbook
    NAME: guestbook-demo
    LAST DEPLOYED: Thu Mar 25 22:23:34 2021
    NAMESPACE: repo-demo
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    1. Get the application URL by running these commands:
      NOTE: It may take a few minutes for the LoadBalancer IP to be available.
            You can watch the status of by running 'kubectl get svc -w guestbook-demo --namespace repo-demo'
      export SERVICE_IP=$(kubectl get svc --namespace repo-demo guestbook-demo -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
      echo http://$SERVICE_IP:3000
   ```

   Check that release deployed as expected as follows:

   ```console
   $ helm list
    NAME            NAMESPACE       REVISION        UPDATED                                 STATUS         CHART            APP VERSION
    guestbook-demo  repo-demo       1               2021-03-25 22:23:34.86922709 +0000 UTC  deployed       guestbook-0.2.0
   ```

## Conclusion

This lab provided you with a brief introduction to the Helm repositories to show how charts can be installed. The ability to share your chart means ease of use to both you and your consumers.

## Extra

You can convert your own Github repository to a Helm Chart repository. Assuming your charts are in `charts/guestbook` and you want to store the packaged charts into a directory `repo/stable`, you can package the charts as follows,

```bash
helm package charts/guestbook -d repo/stable
```

Then index your repo directory,

```bash
helm repo index --url https://remkohdev.github.io/helm101/repo/stable repo/stable 
```

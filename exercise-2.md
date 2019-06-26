# Exercise 2 - Installing Istio on IBM Cloud Kubernetes Service

In this module, you download and install Helm and Istio.

## Install Helm

Helm helps you manage Kubernetes through Helm Charts, packages that define applications. Charts make it easier to install, upgrade, and share your configurations.

Mac users can use [Homebrew](https://brew.sh/):

```text
brew install kubernetes-helm
```

Windows users can use [Chocolatey](https://chocolatey.org/):

```text
choco install kubernetes-helm
```

For other operating systems and install methods, refer to the [Helm installation documentation](https://github.com/helm/helm#install).

## Install Istio

1. Either download Istio directly from [https://github.com/istio/istio/releases](https://github.com/istio/istio/releases) or get the latest version by using curl:

   ```text
   curl -L https://git.io/getLatestIstio | sh -
   ```

2. Change the directory to the Istio file location.

   ```text
   cd istio-<version-number>
   ```

3. Add the `istioctl` client to your PATH.

   ```text
   export PATH=$PWD/bin:$PATH
   ```

4. Create the Istio namespace

   ```text
   kubectl create namespace istio-system
   ```

5. Create the Istio custom resource definitions \(CRDs\)

   ```text
   helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -
   ```

   Verify the CRDs were created.

   ```text
   kubectl get crds | grep 'istio.io\|certmanager.k8s.io'
   ```

6. Install Istio using Helm.

   ```text
   helm template install/kubernetes/helm/istio --name istio --namespace istio-system --set pilot.traceSampling=100.0 --set global.proxy.tracer=datadog | kubectl apply -f -
   ```

   Helm allows us to easily change the default Istio configuration. Here we're updating the trace sampling value and setting our tracer to Datadog. We'll see more of this in exercise 4.

7. Ensure that the `istio-*` Kubernetes services are deployed before you continue.

   ```text
    kubectl get svc -n istio-system
   ```

   Sample output:

   ```text
    NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                                                                                                                                      AGE
    istio-citadel            ClusterIP      172.21.242.77    <none>           8060/TCP,15014/TCP                                                                                                                           34s
    istio-egressgateway      ClusterIP      172.21.20.200    <none>           80/TCP,443/TCP,15443/TCP                                                                                                                     35s
    istio-galley             ClusterIP      172.21.246.214   <none>           443/TCP,15014/TCP,9901/TCP                                                                                                                   36s
    istio-ingressgateway     LoadBalancer   172.21.151.128   169.60.168.234   80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:32268/TCP,15030:30743/TCP,15031:32200/TCP,15032:31341/TCP,15443:31059/TCP,15020:31039/TCP   35s
    istio-pilot              ClusterIP      172.21.243.70    <none>           15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                                       34s
    istio-policy             ClusterIP      172.21.144.137   <none>           9091/TCP,15004/TCP,15014/TCP                                                                                                                 34s
    istio-sidecar-injector   ClusterIP      172.21.230.192   <none>           443/TCP                                                                                                                                      33s
    istio-telemetry          ClusterIP      172.21.213.11    <none>           9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                                       34s
   ```

**Note: If your istio-ingressgateway service external IP is `<none>`, confirm that you are using a standard/paid cluster. Free cluster is not supported for this lab.**

1. Ensure the corresponding pods `istio-citadel-*`, `istio-ingressgateway-*`, `istio-pilot-*`, and `istio-policy-*` are all in **`Running`** state before you continue.

   ```text
    kubectl get pods -n istio-system
   ```

   Sample output:

   ```text
   NAME                                      READY   STATUS      RESTARTS   AGE
   istio-citadel-7695fc84d9-wsk74            1/1     Running     0          41s
   istio-cleanup-secrets-1.2.0-sv5ns         0/1     Completed   0          45s
   istio-galley-75466f5dc7-n9rg4             0/1     Running     0          42s
   istio-ingressgateway-69dfdb8d6-pn4bg      0/1     Running     0          42s
   istio-init-crd-10-272xq                   0/1     Completed   0          52s
   istio-init-crd-11-9zvsl                   0/1     Completed   0          52s
   istio-init-crd-12-rg4sj                   0/1     Completed   0          52s
   istio-pilot-5c5758cb-ptvdg                1/2     Running     0          41s
   istio-policy-8469ffd9df-gnp4p             2/2     Running     2          42s
   istio-security-post-install-1.2.0-hq42r   0/1     Completed   0          44s
   istio-sidecar-injector-6f4c67c6cd-z7jkj   1/1     Running     0          41s
   istio-telemetry-5bf99ff6df-8k8tm          2/2     Running     2          42s
   prometheus-776fdf7479-c786k               0/1     Running     0          41s
   ```

   Before you continue, make sure all the pods are deployed and are either in the **`Running`** or **`Completed`** state. If they're in `pending` state, wait a few minutes to let the deployment finish.

   Congratulations! You successfully installed Istio into your cluster.

## [Continue to Exercise 3 - Deploy Guestbook with Istio Proxy](exercise-3.md)


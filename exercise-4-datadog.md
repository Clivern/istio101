# Exercise 4 - Observe service telemetry: metrics and tracing

### Challenges with microservices

We all know that microservice architecture is the perfect fit for cloud native applications and it increases the delivery velocities greatly. Envision you have many microservices that are delivered by multiple teams, how do you observe the overall platform and each of the services to find out exactly what is going on with each of the services? When something goes wrong, how do you know which service or which communication among the few services are causing the problem?

### Istio telemetry

Istio's tracing and metrics features are designed to provide broad and granular insight into the health of all services. Istio's role as a service mesh makes it the ideal data source for observability information, particularly in a microservices environment. As requests pass through multiple services, identifying performance bottlenecks becomes increasingly difficult using traditional debugging techniques. Distributed tracing provides a holistic view of requests transiting through multiple services, allowing for immediate identification of latency issues. With Istio, distributed tracing comes by default. This will expose latency, retry, and failure information for each hop in a request.

The Istio’s design makes it easy to gather telemetry. In this workshop we’ll be using Datadog to gather metrics, distributed traces, and logs from Istio.

You can read more about how [Istio mixer enables telemetry reporting](https://istio.io/docs/concepts/policy-and-control/mixer.html).



### Setting up Datadog

Navigate to the Datadog directory.

```
cd ../../datadog
```

In exercise 3, you learned about sidecars and set up Istio’s automatic sidecar injection. To prevent automatic sidecar injection and to keep your kubernetes workloads organized and easier to manage, create a namespace for Datadog.

Create the `datadog` namespace.

```
kubectl create -f datadog-namespace.yaml
```

2. Create the role-based access controls (RBAC) that will allow the Datadog Agent to collect telemetry from the Kubernetes cluster.

```
Kubectl create -f datadog-rbac.yaml
```

3. Install Kube-state-metrics to expose more telemetry about the internal state of Kubernetes

```
Kubectl create -f datadog-kubestatemetrics.yaml
```

### The Datadog Daemonset

In the previous exercises, you created Kubernetes Deployments. Deployments specify the number of replica pods that Kubernetes should deploy. Kubernetes will automatically assign those pods to nodes with available resources. You’ll use a different type of Kubernetes resource, a Daemonset, to deploy the Datadog Agent. Daemonsets ensure that one pod is deployed on every Kubernetes node in the cluster.

Log into your Datadog account, then go to [https://app.datadoghq.com/account/settings#api](https://app.datadoghq.com/account/settings#api) to get your API key. Edit the `datadog-agent.yaml` file and replace the `<YOUR API KEY>` placeholder with your API key.

Create the Datadog Daemonset.

```
Kubectl create -f datadog-agent.yaml
```

2. Verify that the Daemonset has been created and is running one pod per node.

```
# Show the nodes in your cluster
Kubectl get nodes
# Show the pods running in the Datadog namespace along with which node they’re on
Kubectl get pods -n datadog -o wide
```

3. Verify that the Datadog Agent is collecting data. Copy the name of a Datadog pod from step 2 and use it to execute the `agent status` command.

```
Kubect exec -ti -n datadog your-datadog-agent-pod-name -- agent status
```

The output will contain a list of systems that Datadog is monitoring, including Istio:

```
istio (2.1.0)
-------------
Instance ID: istio:36ff5e31cbc31301 [OK]
Total Runs: 6
Metric Samples: Last Run: 328, Total: 1,500
Events: Last Run: 0, Total: 0
Service Checks: Last Run: 0, Total: 0
Average Execution Time : 452ms
```


=== EVERYTHING BELOW HERE IS LEGACY ===

### Configure Istio to receive telemetry data

1. Verify that the Grafana, Prometheus, Kiali and Jaeger add-ons were installed successfully. All add-ons are installed into the `istio-system` namespace.

   ```text
    kubectl get pods -n istio-system
    kubectl get services -n istio-system
   ```

2. Configure Istio to automatically gather telemetry data for services that run in the service mesh. Create a rule to collect telemetry data.

   ```text
    cd ../../plans/
    kubectl create -f guestbook-telemetry.yaml
   ```

3. Obtain the guestbook endpoint to access the guestbook. You can access the guestbook via the external IP for your service as guestbook is deployed as a load balancer service. Get the EXTERNAL-IP of the guestbook service via output below:

   ```text
    kubectl get service guestbook -n default
   ```

Go to this external ip address in the browser to try out your guestbook.

1. Generate a small load to the app.

   ```text
    for i in {1..20}; do sleep 0.5; curl http://<guestbook_IP>/; done
   ```


## Understand what happened

Although Istio proxies are able to automatically send spans, they need some hints to tie together the entire trace. Apps need to propagate the appropriate HTTP headers so that when the proxies send span information to Zipkin or Jaeger, the spans can be correlated correctly into a single trace.

In the example, when a user visits the Guestbook app, the HTTP request is sent from the guestbook service to Watson Tone Analyzer. In order for the individual spans of guestbook service and Watson Tone Analyzer to be tied together, we have modified the guestbook service to extract the required headers \(x-request-id, x-b3-traceid, x-b3-spanid, x-b3-parentspanid, x-b3-sampled, x-b3-flags, x-ot-span-context\) and forward them onto the analyzer service when calling the analyzer service from the guestbook service. The change is in the `v2/guestbook/main.go`. By using the `getForwardHeaders()` method, we are able to extract the required headers, and then we use the required headers further when calling the analyzer service via the `getPrimaryTone()` method.

## Questions

1. Does a user need to modify their app to get metrics for their apps? A: 1. Yes 2. No. \(2 is correct\)
2. Does a user need to modify their app to get distributed tracing for their app to work properly? A: 1. Yes 2. No. \(1 is correct\)
3. What distributed tracing system does Istio support by default? A: 1. Zipkin 2. Kibana 3. LogStash 4. Jaeger. \(1 and 4 are correct\)

#### [Continue to Exercise 5 - Expose the service mesh with the Istio Ingress Gateway](exercise-5.md)


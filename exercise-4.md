# Exercise 4 - Observe service telemetry: metrics and tracing

## Challenges with microservices

We all know that microservice architecture is the perfect fit for cloud native applications and it increases the delivery velocities greatly. Envision you have many microservices that are delivered by multiple teams, how do you observe the overall platform and each of the services to find out exactly what is going on with each of the services? When something goes wrong, how do you know which service or which communication among the few services are causing the problem?

## Istio telemetry

Istio's tracing and metrics features are designed to provide broad and granular insight into the health of all services. Istio's role as a service mesh makes it the ideal data source for observability information, particularly in a microservices environment. As requests pass through multiple services, identifying performance bottlenecks becomes increasingly difficult using traditional debugging techniques. Distributed tracing provides a holistic view of requests transiting through multiple services, allowing for immediate identification of latency issues. With Istio, distributed tracing comes by default. This will expose latency, retry, and failure information for each hop in a request.

The Istio’s design makes it easy to gather telemetry. In this workshop we’ll be using Datadog to gather metrics, distributed traces, and logs from Istio.

You can read more about how [Istio mixer enables telemetry reporting](https://istio.io/docs/concepts/policy-and-control/mixer.html).

## Setting up Datadog

Navigate to the Datadog directory.

```text
cd ../../datadog
```

In exercise 3, you learned about sidecars and set up Istio’s automatic sidecar injection. To prevent automatic sidecar injection and to keep your kubernetes workloads organized and easier to manage, create a namespace for Datadog.

1. Create the `datadog` namespace.

   ```text
   kubectl create namespace datadog
   ```

2. Create the role-based access controls \(RBAC\) that will allow the Datadog Agent to collect telemetry from the Kubernetes cluster.

   ```text
   kubectl create -f datadog-rbac.yaml
   ```

## The Datadog Daemonset

In the previous exercises, you created Kubernetes Deployments. Deployments specify the number of replica pods that Kubernetes should deploy. Kubernetes will automatically assign those pods to nodes with available resources. You’ll use a different type of Kubernetes resource, a Daemonset, to deploy the Datadog Agent. Daemonsets ensure that one pod is deployed on every Kubernetes node in the cluster.

1. Log into your Datadog account, then go to [https://app.datadoghq.com/account/settings\#api](https://app.datadoghq.com/account/settings#api) to get your API key. Edit the `datadog-agent.yaml` file and replace the `<YOUR API KEY>` placeholder with your API key.

   Create the Datadog Daemonset.

   ```text
   kubectl create -f datadog-agent.yaml
   ```

2. Verify that the Daemonset has been created and is running one pod per node.

   List the nodes in your cluster

   ```text
   kubectl get nodes
   ```

   List the pods running in the Datadog namespace along with which node they’re on.

   ```text
   kubectl get pods -n datadog -o wide
   ```

3. Verify that the Datadog Agent is collecting data. Copy the name of a Datadog pod from step 2 and use it to execute the `agent status` command.

   ```text
   Kubect exec -ti -n datadog your-datadog-agent-pod-name -- agent status
   ```

   The output will contain a list of systems that Datadog is monitoring, including Istio:

   ```text
   istio (2.1.0)
   -------------
   Instance ID: istio:36ff5e31cbc31301 [OK]
   Total Runs: 6
   Metric Samples: Last Run: 328, Total: 1,500
   Events: Last Run: 0, Total: 0
   Service Checks: Last Run: 0, Total: 0
   Average Execution Time : 452ms
   ```

## Testing it out

1. Obtain the guestbook endpoint to access the guestbook. You can access the guestbook via the external IP for your service as guestbook is deployed as a load balancer service. Get the EXTERNAL-IP of the guestbook service via output below:

   ```text
   kubectl get service guestbook -n default
   ```

   Go to this external ip address in the browser to try out your guestbook.

2. Generate a small load to the app.

   ```text
   for i in {1..20}; do sleep 0.5; curl http://<guestbook_IP>/; done
   ```

3. Open a browser and log into your [Datadog account](https://app.datadoghq.com). Explore the application and try to answer:
   * Which of your nodes is using the most CPU and what processes are running on that host?
   * What containers are using the most memory?
   * What's the P90 for requests to the guestbook?

### [Continue to Exercise 5 - Expose the service mesh with the Istio Ingress Gateway](exercise-5.md)


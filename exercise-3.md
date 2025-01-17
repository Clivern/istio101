# Exercise 3 - Deploy the Guestbook app with Istio Proxy



The Guestbook app is a sample app for users to leave comments. It consists of a web front end, Redis master for storage, and a replicated set of Redis slaves. We will also integrate the app with Watson Tone Analyzer which detects the sentiment in users' comments and replies with emoticons.

The guestbook files are in the `/workshop/guestbook` directory. Navigate into the app directory.

```text
    cd ../guestbook/v2
```

### Enable the automatic sidecar injection for the default namespace

In Kubernetes, a sidecar is a utility container in the pod, and its purpose is to support the main container. For Istio to work, Envoy proxies must be deployed as sidecars to each pod of the deployment. There are two ways of injecting the Istio sidecar into a pod: manually using the istioctl CLI tool or automatically using the Istio sidecar injector. In this exercise, we will use the automatic sidecar injection provided by Istio.

1. Annotate the default namespace to enable automatic sidecar injection:

   ```text
    kubectl label namespace default istio-injection=enabled
   ```

2. Validate the namespace is annotated for automatic sidecar injection:

   ```text
    kubectl get namespace -L istio-injection
   ```

   Sample output:

   ```text
    NAME             STATUS   AGE    ISTIO-INJECTION
    default          Active   271d   enabled
    istio-system     Active   5d2h
    ...
   ```

### Create a Redis database

The Redis database is a service that you can use to persist the data of your app. The Redis database comes with a master and slave modules.

1. Create the Redis controllers and services for both the master and the slave.

   ```text
    kubectl create -f redis-master-deployment.yaml
    kubectl create -f redis-master-service.yaml
    kubectl create -f redis-slave-deployment.yaml
    kubectl create -f redis-slave-service.yaml
   ```

2. Verify that the Redis controllers for the master and the slave are created.

   ```text
    kubectl get deployment
   ```

   Output:

   ```text
    NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    redis-master   1         1         1            1           5d
    redis-slave    2         2         2            2           5d
   ```

3. Verify that the Redis services for the master and the slave are created.

   ```text
    kubectl get svc
   ```

   Output:

   ```text
    NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
    redis-master   ClusterIP      172.21.85.39    <none>          6379/TCP       5d
    redis-slave    ClusterIP      172.21.205.35   <none>          6379/TCP       5d
   ```

4. Verify that the Redis pods for the master and the slave are up and running.

   ```text
    kubectl get pods
   ```

   Output:

   ```text
    NAME                            READY     STATUS    RESTARTS   AGE
    redis-master-4sswq              2/2       Running   0          5d
    redis-slave-kj8jp               2/2       Running   0          5d
    redis-slave-nslps               2/2       Running   0          5d
   ```

## Install the Guestbook app

1. Inject the Istio Envoy sidecar into the guestbook pods, and deploy the Guestbook app on to the Kubernetes cluster. Deploy both the v1 and v2 versions of the app:

   ```text
    kubectl apply -f ../v1/guestbook-deployment.yaml
    kubectl apply -f guestbook-deployment.yaml
   ```

These commands will inject the Istio Envoy sidecar into the guestbook pods, as well as deploy the Guestbook app on to the Kubernetes cluster. Here we have two versions of deployments, a new version \(`v2`\) in the current directory, and a previous version \(`v1`\) in a sibling directory. They will be used in future sections to showcase the Istio traffic routing capabilities.

1. Create the guestbook service.

   ```text
    kubectl create -f guestbook-service.yaml
   ```

2. Verify that the service was created.

   ```text
    kubectl get svc
   ```

   Output:

   ```text
    NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
    guestbook      LoadBalancer   172.21.36.181   169.61.37.140   80:32149/TCP   5d
   ```

3. Verify that the pods are up and running.

   ```text
    kubectl get pods
   ```

   Sample output:

   ```text
    NAME                            READY     STATUS    RESTARTS   AGE
    guestbook-v1-89cd4b7c7-frscs    2/2       Running   0          5d
    guestbook-v2-56d98b558c-mzbxk   2/2       Running   0          5d
   ```

   Note that each guestbook pod has 2 containers in it. One is the guestbook container, and the other is the Envoy proxy sidecar.

### Use Watson Tone Analyzer

Watson Tone Analyzer detects the tone from the words that users enter into the Guestbook app. The tone is converted to the corresponding emoticons.

1. Switch to your own account. Login again with `ibmcloud login`
   1. Choose your own account
   2. ```text
      Select an account:
      1. Sai Vennam's Account (d815248d6ad0cc354df42d43db45ce09) <-> 1909673
      2. IBM (62c8587074c362fc4525ef4ab5012951) <-> 1835659
      Enter a number> 1
      ```
2. Create Watson Tone Analyzer in your account.

   ```text
    ibmcloud resource service-instance-create my-tone-analyzer-service tone-analyzer lite us-south
   ```

3. Create the service key for the Tone Analyzer service. This command should output the credentials you just created. You will need the value for **apikey** & **url** later.

   ```text
    ibmcloud resource service-key-create tone-analyzer-key Manager --instance-name my-tone-analyzer-service
   ```

4. If you need to get the service-keys later, you can use the following command:

   ```text
    ibmcloud resource service-key tone-analyzer-key
   ```

5. Open the `analyzer-deployment.yaml` and find the env section near the end of the file. Replace `YOUR_API_KEY` with your own API key, and replace `YOUR_URL` with the url value you saved before. YOUR\_URL should look something like `https://gateway.watsonplatform.net/tone-analyzer/api`. Save the file.  
 

   ![](.gitbook/assets/screen-shot-2019-06-26-at-1.23.59-pm.png)

6. Deploy the analyzer pods and service, using the `analyzer-deployment.yaml` and `analyzer-service.yaml` files found in the `guestbook/v2` directory. The analyzer service talks to Watson Tone Analyzer to help analyze the tone of a message.

   ```text
    kubectl apply -f analyzer-deployment.yaml
    kubectl apply -f analyzer-service.yaml
   ```

7. Switch back to the IBM account. Login again with `ibmcloud login`
   1. Choose the IBM account
   2. ```text
      Select an account:
      1. Sai Vennam's Account (d815248d6ad0cc354df42d43db45ce09) <-> 1909673
      2. IBM (62c8587074c362fc4525ef4ab5012951) <-> 1835659
      Enter a number> 2
      ```

Great! Your guestbook app is up and running. In Exercise 4, you'll be able to see the app in action by directly accessing the service endpoint. You'll also be able to view Telemetry data for the app.

#### [Continue to Exercise 4 - Telemetry](exercise-4.md)


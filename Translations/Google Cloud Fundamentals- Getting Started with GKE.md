# Task 1: Sign in to the Google Cloud Platform (GCP) Console

For each lab, you get a new GCP project and set of resources for a fixed time at no cost.

1. Make sure you signed into Qwiklabs using an **incognito window**.

2. When ready, click ![start_lab](https://cdn.qwiklabs.com/XE8x7uvQokyubNwnYKKc%2BvBBNrMlo5iNZiDDzQQ3Ddo%3D).

2. Click **Open Google Console**.

3. Click **Use another account** and copy/paste credentials for this lab into the prompts.

4. Accept the terms and skip the recovery resource page.


# Task 2: Confirm that needed APIs are enabled
1. In GCP console, on the top right toolbar, click the Open Cloud Shell button.
![Cloud Shell icon](https://cdn.qwiklabs.com/vdY5e%2Fan9ZGXw5a%2FZMb1agpXhRGozsOadHURcR8thAQ%3D)


2. Click **Continue**. ![Continue icon](https://cdn.qwiklabs.com/lr3PBRjWIrJ%2BMQnE8kCkOnRQQVgJnWSg4UWk16f0s%2FA%3D)
3. Set the default project
``` console
gcloud config set project $DEVSHELL_PROJECT_ID
```

4. Enable **Kubernetes Engine API** and **Container Registry API** API's and Services
``` console
gcloud services enable container.googleapis.com

gcloud services enable containerregistry.googleapis.com
```

# Task 3: Start a Kubernetes Engine cluster

1. For convenience, place the zone that Qwiklabs assigned you to into an environment variable called MY_ZONE. At the Cloud Shell prompt, type this command and substitute with the zone assigned to you by qwiklabs:

```console
export MY_ZONE=<zone assignd by qwiklabs>
```

2. Start a Kubernetes cluster managed by Kubernetes Engine. Name the cluster webfrontend and configure it to run 2 nodes:

```console
gcloud container clusters create webfrontend --zone $MY_ZONE --num-nodes 2
```
3. After the cluster is created (may take several minutes), check your installed version of Kubernetes using the kubectl version command:
```console
kubectl version
```
View your running nodes in the in compute instances window
```console
gcloud compute instances list
```
# Task 4: Run and deploy a container
1. From your Cloud Shell, launch a single instance of the nginx container.
```console
kubectl create deploy nginx --image=nginx:1.17.10 --nodes=1
```
In Kubernetes, all containers run in pods. This use of the kubectl create command caused Kubernetes to create a deployment consisting of a single pod containing the nginx container.

2. View the pod running the nginx container:

```console
kubectl get pods
```

3. Expose the nginx container to the Internet via a service and an external load balancer with a public IP address attached to it:

``` console
kubectl expose deployment nginx --port 80 --type LoadBalancer
```

4. View the new service:

```console
kubectl get services
```

5. Use the displayed external IP address to test and contact the nginx container remotely via a webbrowser. It may take a few seconds before the External-IP field is populated for your service. This is normal. Just re-run the kubectl get services command every few seconds until the field is populated.

6. Scale up the number of pods running on your service:

```console
kubectl scale deployment nginx --replicas 3
```

7. Confirm that Kubernetes has updated the number of pods:

```console
kubectl get pods
```

8. Confirm that your external IP address has not changed:

```console
kubectl get services
```

9. Return to the web browser tab in which you viewed your cluster's external IP address. Refresh the page to confirm that the nginx web server is still responding.

# End your lab
When you have completed your lab, click End Lab. Qwiklabs removes the resources you’ve used and cleans the account for you.

You will be given an opportunity to rate the lab experience. Select the applicable number of stars, type a comment, and then click Submit.
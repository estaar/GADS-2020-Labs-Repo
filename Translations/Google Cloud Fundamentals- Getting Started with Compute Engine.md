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

For convenience, place the zone that Qwiklabs assigned you to into an environment variable called MY_ZONE. At the Cloud Shell prompt, type this partial command:
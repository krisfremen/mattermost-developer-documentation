---
title: "Kubernetes Troubleshooting"
date: 2018-11-07T15:24:42+01:00
weight: 10
---

This page is intended to help developers access and perform maintenance tasks in the Production Mattermost Kubernetes Cluster running on AWS using EKS.

## Set up local environment to access K8s

First, make sure you have installed `kubectl` version 1.10 or later.

Also you will need the AWS Keys for the Main Mattermost AWS account. You can get one using Onelogin following these [instructions](../../onelogin-aws).

When using the Onelogin-aws for Kubernetes configuration, please select the main Mattermost AWS account.

### To install kubectl

Follow the instructions on this [page](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

### Configure kubectl for Amazon EKS

Follow the instructions on this [page](https://docs.aws.amazon.com/eks/latest/userguide/configure-kubectl.html).

### Create a kubeconfig for Amazon EKS

Follow the instructions on this [page](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html).

NOTE: Please talk with Carlos to get the cluster name.

### Check if you can see the pods

To check if you can see the pods in the K8s cluster, you can do the following:

```Bash
$ kubectl get po -n community
NAME                                              READY   STATUS    RESTARTS   AGE
mattermost-community-0                            1/1     Running   0          5h
mattermost-community-1                            1/1     Running   0          23h
mattermost-community-jobserver-65985bfc47-88qq9   1/1     Running   0          5h
```

## Switch Back to EC2

This will be temporary because we are aiming to use Kubernetes for `community.mattermost.com`.

If you need to switch back to EC2 machines, carry out the following actions:

    - Log in to the main Mattermost AWS account.
    - Go to `Route53`.
    - Select `mattermost.com`.
    - Delete the entries for `community` and `pre-release`.
    - Add two entries for `pre-release` with a CNAME pointing to the proxy-server (you can find the DNS names by filtering for `pre-release` in the EC2 Dashboard).

Wait for the DNS renew, try to access the server, andm kae sure all EC2 servers are getting updates and are running.

## Troubleshooting

## Namespaces

We are using two namespaces to deploy Mattermost

    - `community` namespace holds the Mattermost deployment which uses the release branch or a stable release and the ingress is pointing to `https://community.mattermost.com` and `https://pre-release.mattermost.com`
    - `community-daily` namespace holds the Mattermost deployment which uses the `master` branch and the ingress is pointing to `https://community-daily.mattermost.com`

## Check if the PODS are running

To check if the pods are running in both namespaces, you can run the following command:

```Bash
$ kubectl get po -n community
NAME                                              READY   STATUS    RESTARTS   AGE
mattermost-community-0                            1/1     Running   0          5h
mattermost-community-1                            1/1     Running   0          23h
mattermost-community-jobserver-65985bfc47-88qq9   1/1     Running   0          5h

$ kubectl get po -n community-daily
NAME                                                    READY   STATUS    RESTARTS   AGE
mattermost-community-daily-0                            1/1     Running   0          3h
mattermost-community-daily-1                            1/1     Running   0          3h
mattermost-community-daily-jobserver-78f7cbf756-wls4f   1/1     Running   0          2h
```

If one or more pods show a status != `Running`, you can use the `describe` and `logs` commands below to investigate what is wrong:

Describe the pod:

```Bash
$ kubectl describe pods ${POD_NAME} -n ${NAMESPACE}
```

Get the Pod logs:

```Bash
$ kubectl logs pods ${POD_NAME} -n ${NAMESPACE}
```

# gitops-deployment-repo

This repository contains Kubernetes manifests for the sample application 
found in [gitops-app-repo](https://github.com/danielhelfand/gitops-app-repo). 

Follow the instructions in this README and in gitops-app-repo to complete 
the gitops multi-repo setup walkthrough.

## Prerequisites

1. A Kubernetes cluster
2. Install the `argocd cli`: https://argo-cd.readthedocs.io/en/stable/cli_installation/
2. Argo CD installed on your cluster:

```
kubectl create ns argocd
kubectl apply -f https://github.com/argoproj/argo-cd/blob/master/manifests/install.yaml -n argocd
kubectl config set-context --current --namespace=argocd
argocd login --core
```

3. A GitHub account

## Part 1: Configure this Repository with Argo CD

1. Fork this repository to your GitHub account
2. Run the following command to configure this repo with Argo CD (make sure to replace with your GitHub username): 

```
 argocd app create k8s-wild-west --repo https://github.com/<REPLACE WITH USERNAME>/gitops-deployment-repo --path config --dest-name in-cluster --dest-namespace wildwest
```

3. View how Argo CD tracks the difference between what's in this repo versus on the cluster. 
See what resources are used to deploy this application to Kubernetes.

```
argocd app get k8s-wild-west

Name:               k8s-wild-west
Project:            default
Server:             in-cluster
Namespace:          wildwest
URL:                http://localhost:57135/applications/k8s-wild-west
Repo:               https://github.com/danielhelfand/gitops-deployment-repo
Target:             
Path:               config
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        OutOfSync from  (f686798)
Health Status:      Missing

GROUP                      KIND         NAMESPACE  NAME      STATUS     HEALTH   HOOK  MESSAGE
                           Namespace               wildwest  OutOfSync  Missing        
                           Service      wildwest   wildwest  OutOfSync  Missing        
apps                       Deployment   wildwest   wildwest  OutOfSync  Missing        
rbac.authorization.k8s.io  RoleBinding  wildwest   view      OutOfSync  Missing
```

4. Now, sync the resources to your cluster via the argocd cli:

```
argocd app sync k8s-wild-west
```

5. View the resources in the wildwest namespace:

```
kubectl get all -n wildwest

NAME                            READY   STATUS    RESTARTS   AGE
pod/wildwest-65479bf4bd-8fz6v   1/1     Running   0          2m37s
pod/wildwest-65479bf4bd-chrqg   1/1     Running   0          2m37s
pod/wildwest-65479bf4bd-mz4bv   1/1     Running   0          2m37s
pod/wildwest-65479bf4bd-rhj8z   1/1     Running   0          2m37s
pod/wildwest-65479bf4bd-zm4tl   1/1     Running   0          2m37s

NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/wildwest   ClusterIP   10.101.201.103   <none>        8080/TCP   2m38s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wildwest   5/5     5            5           2m38s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/wildwest-65479bf4bd   5         5         5       2m38s
```

6. Once all the pods are running like above, view the logs of the Deployment to 
make sure the application is running:

```
k logs deploy/wildwest -n wildwest -f
```

When you see the log similar to what's below, the application is running:

```
2022-07-22 20:49:25.776  INFO 1 --- [           main] com.vmware.wildwest.WildWestApplication  : Started WildWestApplication in 190.983 seconds (JVM running for 222.429)
```

7. View the application running at http://localhost:8080 in your browser by running the following:

```
kubectl port-forward -n wildwest svc/wildwest 8080:8080
```

Congrats! You have deployed an application using Argo CD and gitops. Now head over 
to [gitops-app-repo](https://github.com/danielhelfand/gitops-app-repo) to complete 
the next part of this walkthrough.

## Part Two: Re-Sync Your Cluster Based on App Change

Before beginning this section, make sure you complete the walkthrough in gitops-app-repo.

Now that you have pushed a new version of the application, you can proceed with updating 
the deployment repo to make sure your changes get synced to your cluster.

1. In [deployment.yaml](https://github.com/danielhelfand/gitops-deployment-repo/blob/f686798a9f77c1a077c9909feda0f9d0d038cb2f/config/deployment.yaml#L20), update the Deployment 
image to use the new digest.

2. Save the image and push to the main branch of your forked repo.

3. After pushing, sync the changes to your cluster

```
argocd app sync k8s-wild-west
```

4. Now when you run `argocd app get k8s-wild-west`, you should see `Sync Status` matches 
the sha of the commit you pushed to this repo.

5. Once all the pods are running like before, view the logs of the Deployment to 
make sure the application is running:

```
k logs deploy/wildwest -n wildwest -f
```

When you see the log similar to what's below, the application is running:

```
2022-07-22 20:49:25.776  INFO 1 --- [           main] com.vmware.wildwest.WildWestApplication  : Started WildWestApplication in 190.983 seconds (JVM running for 222.429)
```

7. View the application running at http://localhost:8080 in your browser by running the following:

```
kubectl port-forward -n wildwest svc/wildwest 8080:8080
```

8. You can see your change by hitting a target in the same and seeing how the score is different this time.

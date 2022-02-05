# Static Pods 
  - Take me to [Video Tutorial](https://kodekloud.com/topic/static-pods/)
  
In this section, we will take a look at Static Pods

#### How do you provide a pod definition file to the kubelet without a kube-apiserver?
- You can configure the kubelet to read the pod definition files from a directory on the server designated to store information about pods.

## Configure Static Pod
- The designated directory can be any directory on the host and the location of that directory is passed in to the kubelet as an option while running the service.
  - The option is named as **`--pod-manifest-path`**.
  
  ![sp](../../images/sp.PNG)
  
## Another way to configure static pod 
- Instead of specifying the option directly in the **`kubelet.service`** file, you could provide a path to another config file using the config option, and define the directory path as staticPodPath in the file.

  ![sp1](../../images/sp1.PNG)

## View the static pods
- To view the static pods
  ```
  $ docker ps
  ```
  ![sp2](../../images/sp2.PNG)

#### The kubelet can create both kinds of pods - the static pods and the ones from the api server at the same time.

  ![sp3](../../images/sp3.PNG)

## Static Pods - Use Case

  ![sp4](../../images/sp4.PNG)
  
  ![sp5](../../images/sp5.PNG)
  
## Static Pods vs DaemonSets

   ![spvsds](../../images/spvsds.PNG)
  
## Create and Delete
Create a pod definition file in the manifests folder. To do this, run the command:

```
kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```

Find and delete static pod:

The pod is on node01; same for on control-plane

1. First, let's identify the node in which the pod called **static-greenbox** is created. To do this, run:

```
root@controlplane:~# kubectlget pods --all-namespaces -o wide  | grep static-greenbox
defaultstatic-greenbox-node01                 1/1Running   0          19s     10.244.1.2   node01       <none>           <none>
root@controlplane:~#

```

From the result of this command, we can see that the pod is running on node01.

1. Next, SSH to node01 and identify the path configured for static pods in this node.`Important`: The path need not be `/etc/kubernetes/manifests`. Make sure to check the path configured in the kubelet configuration file.

```
root@controlplane:~# ssh node01
root@node01:~# ps -ef |  grep /usr/bin/kubelet # or just grep kubelet
root       752   654  0 00:30 pts/0    00:00:00 grep --color=auto /usr/bin/kubelet
root     28567     1  0 00:22 ?        00:00:11 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2
root@node01:~# grep -i staticpod /var/lib/kubelet/config.yaml
staticPodPath: /etc/just-to-mess-with-you
root@node01:~#

```

Here the staticPodPath is `/etc/just-to-mess-with-you`

1. Navigate to this directory and delete the YAML file:

```
root@node01:/etc/just-to-mess-with-you# ls
greenbox.yaml
root@node01:/etc/just-to-mess-with-you# rm -rf greenbox.yaml
root@node01:/etc/just-to-mess-with-you#

```

1. Exit out of node01 using `CTRL + D` or type `exit`. You should return to the `controlplane` node. Check if the `static-greenbox` pod has been deleted:

```
root@controlplane:~# kubectl get pods --all-namespaces -o wide  | grep static-greenbox
root@controlplane:~#
```

#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/

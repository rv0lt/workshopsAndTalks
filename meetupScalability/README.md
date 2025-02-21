# Manually scaling

First, deploy the application with 
`kubectl create -f deployment.yaml`
And verify the pod is up and running with 
`kubectl get po`

- To scale manually we can either use the kubectl command tool, or edit our yaml file and apply the changes, let's see both:

#### Scale out

We can use the following command:

`kubectl scale deployment php-apache --replicas=4`

If we check the pods using 
`kubectl get po -owide`

We will notice that every pod has a different IP associated, the kubernetes service will load-balance the traffic to this different pods according to resource availability.

Another way is to use the interactive editor, which it is something handy to know if you are just starting in kubernetes.
`kubectl edit deployments.apps php-apache`

Aditionally, we can edit the inital yaml file and apply the changes using
`kubectl apply -f deployment.yaml`

>If you are keeping track of your deployments in a Git repository, editing with the command line can lead to confussion on what setting are actually being enforces, you can always replace the yaml file with `kubectl get deployments.apps php-apache -oyaml > deployment.yaml`

#### Scale up

Similarly, we can either use the command line or edit the yaml file. But in this case, the scaling is not as simple as adding more or less instances. Because the resources are differnet, kubernetes needs to create new pods with the new resources allocated. 

If we do`kubectl get po` we will see something like

| NAME  | READY   | STATUS   | RESTARTS | AGE | 
| ------------ | ------------ | ------------ | ------------ | ------------ |
| php-apache-55cf9b569d-mq6r8  | 1/1  |  Running |  0 | 64s|
|  php-apache-55cf9b569d-t79fh | 1/1  | Running |  0 | 62s|


Now, edit the resources section in the deployment either way, using `kubectl edit deployments.apps php-apache` or in the file and apply the changes.

If you fetch the pods inmeditaly after you may see something like

![](https://i.imgur.com/KCpNLFA.png)

What happened is that, Kubernetes requested new pods with the new resources to be created, after they were running, it requested the termination of the old pods, so, in every moment the application was available.


# Automatic scaling

#### Scale up

Let's take our initial deployment and scale it down to 1 replicas again

`kubectl scale deployment php-apache --replicas=1`

If we open the *hpa.yaml*, it defines a CPU target of 50%, which means that when a pod hits 50% of the resource limit, it will schedule new pods. With a maximun of 10 pods.

Let's apply it
`kubectl create -f hpa.yaml`
And, in a different terminal, simulate a higher load.

`kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"`

After waiting a bit, if we do

`kubectl get po`

We will see new pods have been created, we can check the status of the autoscaler with 

`kubectl get hpa`

> You need to have the metric server installed for this to work [https://github.com/kubernetes-sigs/metrics-serverhttp://](https://github.com/kubernetes-sigs/metrics-serverhttp://)

#### Scale out

A good tutorial to follow is [https://docs.aws.amazon.com/eks/latest/userguide/vertical-pod-autoscaler.htmlhttp://](https://docs.aws.amazon.com/eks/latest/userguide/vertical-pod-autoscaler.htmlhttp://)
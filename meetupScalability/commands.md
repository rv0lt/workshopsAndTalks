For quickly access of the commands

# Deploy the application
`kubectl create -f deployment.yaml`

### Verify the pod(s) is/are running
`kubectl get po`

### Save the current deployment configuration to a file
`kubectl get deployments.apps php-apache -o yaml > deployment.yaml`

# Manually scaling

### Scale manually with kubectl
`kubectl scale deployment php-apache --replicas=4`

### Edit deployment interactively
`kubectl edit deployments.apps php-apache`

### Edit and apply the YAML file
`kubectl apply -f deployment.yaml`

# Automatic Scaling

### Apply the Horizontal Pod Autoscaler (HPA)
`kubectl create -f hpa.yaml`

### Simulate load in a separate terminal
`kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"`

### Check the status of the autoscaler
`kubectl get hpa`

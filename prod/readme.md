kubectl apply -f 'filename'
kubectl apply -f emqx-depl.yaml
kubectl apply -f emqx-depl.yaml
kubectl apply -f emqx-depl.yaml
kubectl apply -f emqx-depl.yaml

expose tcp
https://skryvets.com/blog/2019/04/09/exposing-tcp-and-udp-services-via-ingress-on-minikube/
Find whether nginx-ingress-controller controller pod is running:

$ kubectl get deploy nginx-ingress-controller -n ingress-nginx

$ NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
  nginx-ingress-controller   1/1     1            1           8m39s
Copy
Get current controller configuration and save it to a file. In this example, I will create a file in the current directory called nginx-ingress-controller.yaml :

kubectl get deploy nginx-ingress-controller -n ingress-nginx -o yaml>> nginx-ingress-controller.yaml
Copy
Open it and add the following configuration:

spec.template.spec.hostNetwork = true
Copy
In my situation this was right above the containers key:

...
spec:
   hostNetwork: true # <--
   containers:
   - args:
     - /nginx-ingress-controller
     - --configmap=$(POD_NAMESPACE)/nginx-configuration
     - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
     - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
     - --publish-service=$(POD_NAMESPACE)/ingress-nginx
     - --annotations-prefix=nginx.ingress.kubernetes.io
     env:
...
Copy
Save the edited file and apply it by executing:

$ kubectl apply -f nginx-ingress-controller.yaml 
deployment.extensions/nginx-ingress-controller configured
Copy
Sidenote: sometimes it might be required to apply this config by using --force flag:

$ kubectl apply -f nginx-ingress-controller.yaml --force
deployment.extensions/nginx-ingress-controller configured
Copy
Special thanks to MerlinPong who pointed to this fix.

# kubectl get svc -n ingress-nginx
# kubectl get configmap tcp-services -n ingress-nginx -o yaml
# kubectl get deployment ingress-nginx-controller -n ingress-nginx -o yaml
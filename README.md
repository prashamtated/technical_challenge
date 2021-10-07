# TechnicalChallenge

### Task 2
Objective

The goal is to write the  necessary k8s YAML files that will deploy the RoR postfacto app successfully on a Kubernetes cluster.

The candidate is welcome to replace, modify or even drop requirements, as long as the arguments are clear on the README file section.

Apart from the basic requirements that will allow users to access the Postfacto app for their retrospective meetings, here are some wishes that when possible the candidate must address:

1. Postfacto must be accessible from the internet through HTTPS.
2. Postfacto persistent data must be stored in a way that in case of node or cluster failure, the same data will not be lost.
3. The Postfacto app pods cannot be deployed on the same nodes as the database and Redis.
4. To satisfy the QA team, we need to run two versions, 4.1.0 and 4.2.0, of the Postfacto app on the same cluster simultaneously. 
5. Postfacto must route requests based on X-POSTFACTO-VERSION HTTP header value, to the pods that contains the correspondent version of the app.


## Pre-requisites:- 

1. K8s cluster of atleast 2 worker nodes. 
2. Label both nodes with below command  
   kubectl label nodes <node1-name> hosttype=apphost
   kubectl label nodes <node2-name> hosttype=dbhost
3. Install ingress-controller to access Postfacto from internet by below command
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.2/deploy/static/provider/aws/deploy.yaml


## Solution - 

1. In AWS we use a Network load balancer (NLB) to expose the NGINX Ingress controller behind a Service of Type=LoadBalancer.
   then configure required SSL certificate for accessing application via https. By default its up on http.

2. Install Postfacto app version 4.1.0  by  below commad
   kubectl apply -f postfacto-4-1-0.yaml 
   install Postfacto app version 4.2.0  by  below commad
   kubectl apply -f postfacto-4-2-0.yaml 
   these both files contain defication of Deployment, statefullset, configmap , secret, pvc and services defication for both applications.
   after this command , you have 2 versions of application running in same cluster in default namespace. 
   
3. For data persistent in both application, PVC claim of 8GB created for redis db and postgress db in both application.
   example - 
   volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes:
          - "ReadWriteOnce"
        resources:
          requests:
            storage: "8Gi"
   
4. To avoid Postfacto app pods should not be deployed on the same nodes as the database and Redis, nodeSelector attibute added in deployment of Postfacto app of both version.
   example - 
   spec:
      nodeSelector:
        hosttype: apphost
   
5. install test-ingress.yaml (for testing perpose only)  and ingress.yaml for http header base routing. 
   commands : - 
   kubectl apply -f ingress.yaml
   kubectl apply -f test-ingress.yaml



## Post deployment steps- 

1. Create an admin user so you can access the admin console

   kubectl get pods
   # Take note of the postfacto pod name
   kubectl exec <pod name> create-admin-user <admin-email> <admin-password> 
   You should now have a Postfacto running in k8s, you can head to the /admin path to create a retro then to /retros/ to start using it.
# kubernetes

ingress-nginx controller
=========================
We declare which request should go to which service so write an ingress rules that declares how you would like users to be routed to a service. we call these mappings as ingress rules but writing these ingress rules is not enough there must be component that reads these rules and process them that component is called ingress controller.

# Deploy ingress-nginx controller(nlb)

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/aws/deploy.yaml
```

```
kubectl get pods --namespace=ingress-nginx
```
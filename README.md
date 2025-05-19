# CZERTAINLY-OpenShift
Values for CZERTAINLY when running on OpenShift

```
kubectl apply -f nginx-ingress-deployment.yaml \
    -f nginx-ingress-configmap.yaml \
    -f nginx-ingress-service.yaml \
    -f openshift-route.yaml
```
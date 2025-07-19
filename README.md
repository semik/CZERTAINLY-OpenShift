# CZERTAINLY-OpenShift

Values for CZERTAINLY when running on OpenShift

```
git clone https://github.com/semik/CZERTAINLY-OpenShift.git
cd CZERTAINLY-OpenShift

# create a secret for accessing private images in harbor.3key.company (typically not required)
oc create secret docker-registry harbor-secret \
  --docker-server=harbor.3key.company \
  --docker-username=<uid> \
  --docker-password=<password> \
  --docker-email=<registerd-email>

# create route first
oc apply -f openshift-route.yaml
# retreive info about hostname
oc get route czertainly -o jsonpath='{.spec.host}' ; echo ''

# create your private values and fill them with valid content
cp czertainly.values.private.example czertainly.values.private.yaml

# install CZERTAINLY
helm upgrade -n semik75-dev --install czertainly-tlm \
  oci://harbor.3key.company/czertainly-helm/czertainly \
  --values=./czertainly.values.openshift.base.yaml \
  --values=./czertainly.values.resources.yaml \
  --values=./czertainly.values.security.yaml \
  --values=./czertainly.values.private.yaml

# install NGINX to terminate mTLS
oc apply -f nginx-ingress-deployment.yaml \
  -f nginx-ingress-configmap.yaml \
  -f nginx-ingress-service.yaml \
  -f openshift-route.yaml

# now you can test your CZERTAINLY deployment, don't forget to add /administrator/ after hostname :)
```

## Note on Security Context

Bellow is no longer relevant as of CZERTAINLY v2.15.1., except of messagingService.

OpenShift uses UID isolation between namespaces. This isolation means that PODs in different namespaces can't run under the same UID. You can learn about assigned ranges:
```
$ oc describe project semik75-dev
Name:       semik75-dev
...
Annotations:
...
 openshift.io/sa.scc.supplemental-groups=1011740000/10000
 openshift.io/sa.scc.uid-range=1011740000/10000
```

OpenShift ignores the UID assigned by the `USER` command inside container images. CZERTAINLY Helm Charts respects those UIDs and defines them in `securityContext`, but this conflicts with OpenShift security limits, and containers fail to get scheduled with the error message:
`.containers[0].runAsUser: Invalid value: 70: must be in the ranges: [1011740000, 1011749999]`.

The problem can be resolved by removing `runAsUser` values from securityContext; this can be achieved by setting this key to a null value, like this:
```yaml
schedulerService:
 image:
   securityContext:
     runAsUser: null
 curl:
   image:
     securityContext:
     runAsUser: null
```
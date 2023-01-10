# openshift-sealed-secrets

A helm chart that depends on the upstream bitnami sealed secrets chart and installs with an existing public/private key.

Copy a previously created sealing secret cert and private key used in an existing cluster...

```sh
export controller_namespace=sealed-secrets
oc get $(oc get secret -l sealedsecrets.bitnami.com/sealed-secrets-key=active -n ${controller_namespace} -o name) -n ${controller_namespace} -o template='{{ index .data "tls.crt" | base64decode }}' > /tmp/sealing.crt
oc get $(oc get secret -l sealedsecrets.bitnami.com/sealed-secrets-key=active -n ${controller_namespace} -o name) -n ${controller_namespace} -o template='{{ index .data "tls.key" | base64decode }}' > /tmp/sealing.key
```

Install sealed secrets with existing sealing cert and key...

```sh
export controller_namespace=sealed-secrets
helm dependency build helm/sealed-secrets
helm upgrade --install sealed-secrets helm/sealed-secrets -n ${controller_namespace} --create-namespace --set-file sealing.crt=/tmp/sealing.crt --set-file sealing.key=/tmp/sealing.key
```

Sealed Secrets test commands...

```sh
export controller_namespace=sealed-secrets
export test_namespace=default
oc create secret generic mysecret -n ${test_namespace} --dry-run=client --from-literal=foo=bar -o yaml > /tmp/mysecret.yaml
kubeseal --controller-namespace=${controller_namespace} -o yaml </tmp/mysecret.yaml > /tmp/mysealedsecret.yaml
oc create -f /tmp/mysealedsecret.yaml
oc get secret mysecret -n ${test_namespace}
```

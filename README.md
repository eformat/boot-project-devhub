# boot-project-devhub

Red Hat Developer Hub - project bootstrap

```bash
helm repo add openshift-helm-charts https://charts.openshift.io/

helm show values openshift-helm-charts/redhat-developer-hub --version 1.0.0-1 > values.yaml

export BASE_DOMAIN=$(oc get dns cluster -o jsonpath='{.spec.baseDomain}')
export GITHUB_USER=<user>
export GITHUB_TOKEN=<token>

oc new-project redhat-developer-hub

cat <<EOF | oc -n redhat-developer-hub apply -f -
apiVersion: v1
data:
  password: "$(echo -n ${GITHUB_TOKEN} | base64)"
  username: "$(echo -n ${GITHUB_USER} | base64)"
kind: Secret
metadata:
  name: git-auth
type: kubernetes.io/basic-auth
EOF

# manually merge the custom-values.yaml into -> values.yaml file

helm upgrade \
  -i devhub \
  -f values.yaml \
  --namespace=redhat-developer-hub \
  --version 1.0.0-1 \
  --set clusterRouterBase=apps.${BASE_DOMAIN} \
  openshift-helm-charts/redhat-developer-hub
```

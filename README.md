# Argo CD Operator (Helm)

[Argo CD](https://argoproj.github.io) is a declarative, GitOps continuous delivery tool for Kubernetes.



## Operator Features
This Argo CD Operator is based on the official stable [Helm Chart](https://github.com/argoproj/argo-helm/tree/master/charts/argo-cd) 
and currently installs the non-HA version of Argo CD v1.2.4 in OpenShift 4.2.

* Basic Installation to a single Namespace
* Configure through Custom Resource


## Argo CD Operator (Helm) installation

As usual there is more than one way to install this Operator:

* OpenShift Marketplace
* OpenShift local


### OpenShift Marketplace
Before this Operator will (hopefully) be available as Community Operator in your OpenShift OperatorHub Marketplace you have to manually add a new Catalog Source to your OpenShift installation. 

```bash
# Create a new Catalog Source
oc apply -f deploy/olm-catalog/operator-source.yaml -n openshift-marketplace

# Wait for rollout and preview result
oc rollout status -w deployment/argocd-operators-helm -n openshift-marketplace
oc get operatorsource  argocd-operators-helm  -n openshift-marketplace
```

You will now find the Operator with 'Provider Type: Custom' in your OpenShift OperatorHub Marketplace and you can proceed via Web UI, or just use the following commands, to install the Operator in a 'argocd' Namespace (recommended):

```bash
# Create namespace and install operator
oc apply -f deploy/install-marketplace.yaml

# Wait for rollout
oc rollout status -w deployment/argocd-operator-helm -n argocd
```

Now you can proceed with chapter 'Install Argo CD'.


### OpenShift local 
Install this Operator from your local clone with:
```bash
# Create namespace and install operator
oc apply -f deploy/install-local.yaml

# Wait for rollout
oc rollout status -w deployment/argocd-operator-helm -n argocd
```
Now you can proceed with chapter 'Install Argo CD'.


## Install Argo CD
This Operator manages a single Namespace installation of Argo CD. Therefore you have to install the Operator and Argo CD in the same Namespace. For simplicity we recommend creating a Namespace 'argocd' (if not already done) so all the defaults from the example work out of the box. 

To install Argo CD create a new Custom Resource with your [customizations](https://github.com/disposab1e/argocd-operator-helm/blob/master/deploy/crds/argoproj.io_v1alpha1_argocd_cr.yaml) or use the provided example from the Web UI.

You can also use the following command to install in Argo CD in the 'argocd' namespace with defauls:

```bash
# Install Argo CD with default values
oc apply -f deploy/crds/argoproj.io_v1alpha1_argocd_cr.yaml -n argocd
```


## Install Argo CD CLI
Download the CLI directly from your Argo CD installation and make it available in your path:

```bash
curl https://<openshift route hostname>/download/argocd-linux-amd64 -o argocd
```


## Get your initial Argo CD 'admin' password
The initial 'admin' password is autogenerated to be the pod name of the Argo CD API server. This can be retrieved with the command:

```bash
oc get pods -n argocd -l app.kubernetes.io/name=argo-cd-server -o name | cut -d'/' -f 2 
```


## Login to Argo CD using the CLI

```bash
argocd login --insecure --username admin --password <initial admin password> <openshift route hostname>
```


If you like you can change your 'admin' password to e.g. 'admin' 

```bash
argocd account update-password --insecure --current-password <initial admin password> --new-password admin 
```


## Install guestbook App

```bash
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path helm-guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default

argocd app sync guestbook
```


## Uninstall Argo CD
Simply remove the Custom Resource through the Web UI or use following command:

```bash
oc delete ArgoCD argocd -n argocd
```

The uninstallation process will remove the Argo CD installation (CR) but NOT the CRD's. You have to remove them manually:

```bash
# Delete Argo CD CRDs
oc delete crd appprojects.argoproj.io
oc delete crd applications.argoproj.io
```


## Uninstall Argo CD Operator (Helm)
Simply uninstall the Operator through the Web UI and/or use following commands:

```bash
# Delete Cluster Service version
oc delete clusterserviceversions.operators.coreos.com argocd-operator-helm.v0.0.1 -n argocd

# Delete Subscription
oc delete subscriptions.operators.coreos.com argocd-operators-helm -n argocd

# Delete CRD
oc delete crd argocds.argoproj.io
```

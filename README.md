# Install GitOps Operator 

1.- Clone Repo for install GitOps  Operator

```
git clone https://github.com/Everything-is-Code/openshift-gitops.git
```

Login in the Cluster 

```
oc login --token=$TOKEN --server=$API_URL
```

After apply the resource folder for install the manifets in the local cluster

```
oc apply -k openshift-gitops/resources

clusterrolebinding.rbac.authorization.k8s.io/argocd-rbac-ca created
subscription.operators.coreos.com/openshift-gitops-operator created
```



# Installing ACM and ACS
While Wait for EKS we proceed to install ACM and  integrar with  Openshift GitOps

First of All Clone the ACM Repo

```
git clone https://github.com/Everything-is-Code/devsecops.git

cd devsecops
```

Then apply boostrap folder to create resources 

```
oc apply -k bootstrap/

appproject.argoproj.io/advance-cluster-management created
applicationset.argoproj.io/advance-cluster-management created
appproject.argoproj.io/advance-cluster-security created
applicationset.argoproj.io/advance-cluster-security created
```

# Install EKS

## AWS Config 
We need config the credentials to can create EKS instance 
create the credentials file:
```
vim $HOME/.aws/credentials

[default] 
aws_access_key_id = {AWSSECRETKEY} 
region = $REGION
aws_access_key_id = $KEY_ID
aws_secret_access_key = $KEY_SECRET
```

## Installing EKS

Proceed to execute the EKS creation

```
eksctl create cluster --name cluster-01 --region eu-west-1
```

when finished we have log similar to this:

```
2023-11-09 14:06:40 [✔]  all EKS cluster resources for "cluster-01" have been created
2023-11-09 14:06:40 [ℹ]  nodegroup "ng-7cb7773a" has 2 node(s)
2023-11-09 14:06:40 [ℹ]  node "ip-{SOMEIP}.eu-west-1.compute.internal" is ready
2023-11-09 14:06:40 [ℹ]  node "ip-{SOMEIP}.eu-west-1.compute.internal" is ready
2023-11-09 14:06:40 [ℹ]  waiting for at least 2 node(s) to become ready in "ng-7cb7773a"
2023-11-09 14:06:40 [ℹ]  nodegroup "ng-7cb7773a" has 2 node(s)
2023-11-09 14:06:40 [ℹ]  node "ip-{SOMEIP}.eu-west-1.compute.internal" is ready
2023-11-09 14:06:40 [ℹ]  node "ip-{SOMEIP}.eu-west-1.compute.internal" is ready
2023-11-09 14:06:42 [ℹ]  kubectl command should work with "/Users/nacho/.kube/config", try 'kubectl get nodes'
2023-11-09 14:06:42 [✔]  EKS cluster "cluster-01" in "eu-west-1" region is ready
```

## Importing EKS to ACM 

now we proced to import this cluster 

Go to Infracstuture -> Clusters -> Import Cluster

### step 1
![image](https://github.com/Everything-is-Code/devsecops/assets/10425803/523b7420-abba-4204-aade-ccae90efd5bd)
### step 2
![image](https://github.com/Everything-is-Code/devsecops/assets/10425803/4ba6b045-12f4-4e31-8896-5e6b2d20cbfb)
### step 3
![image](https://github.com/Everything-is-Code/devsecops/assets/10425803/1e1c954c-ed86-41ba-b921-0556a7c11337)

## Add EKS to Argo
For use EKS cluster in Argo with acm we need import it manualy for this we should login with kubectl

get the context:

```
kubectl config get-contexts

CURRENT   NAME                                                                      CLUSTER                                                AUTHINFO                                                          NAMESPACE
          default/api-cluster-j99wm-j99wm-sandboxXXX-opentlc-com:6443/kube:admin   api-cluster-j99wm-j99wm-sandboxXXX-opentlc-com:6443   kube:admin/api-cluster-j99wm-j99wm-sandbox1645-opentlc-com:6443   default
*         default/api-cluster-svrk8-svrk8-sandboxXXXX-opentlc-com:6443/kube:admin   api-cluster-svrk8-svrk8-sandboxXXX-opentlc-com:6443   kube:admin/api-cluster-svrk8-svrk8-sandbox1791-opentlc-com:6443   default
          default/api-gc07m0gr-westeurope-aroapp-io:6443/kube:admin                 api-gc07m0gr-westeurope-aroapp-io:6443                 kube:admin/api-gc07m0gr-westeurope-aroapp-io:6443                 default
          open-environment-svrk8-admin@cluster-01.eu-west-1.eksctl.io               cluster-01.eu-west-1.eksctl.io                         open-environment-svrk8-admin@cluster-01.eu-west-1.eksctl.io       open-cluster-management-agent
```


And add cluster to argo:

```
argocd cluster add open-environment-svrk8-admin@cluster-01.eu-west-1.eksctl.io cluster-01 --insecure
```

After this we cna continue using ACM and Argo Placements without any problem 


# Installing AKS
for AKS we will to add credentials to ACM and create CLuster from the UI 

## Add Azure Credentials to ACM
![image](https://github.com/Everything-is-Code/devsecops/assets/10425803/70f3e4bc-e500-4d28-94c7-70987b41909d)

## Create Cluster
Go to Infracstuture -> Clusters -> Create Cluster

### step 1
![image](https://github.com/Everything-is-Code/devsecops/assets/10425803/c7ff71a1-1e9d-4e0b-82b7-3958f1639947)

### step 2
![image](https://github.com/Everything-is-Code/devsecops/assets/10425803/5fb54b6f-2628-48c6-8f62-a6827c8a17e6)

### step 3
![image](https://github.com/Everything-is-Code/devsecops/assets/10425803/2eff955a-2084-44b3-b1ed-8e7dd7e5a036)

### step 4 Verifiy cluster is creating
![image](https://github.com/Everything-is-Code/devsecops/assets/10425803/e83275dc-f822-49d2-abdf-d2f1473eedbc)



# Import Clusters to ACS

we proceed to import the cluster to ACS
### step 1
![image](https://github.com/Everything-is-Code/devsecops/assets/10425803/e4887118-e649-45d5-8f67-ab7b60503f6c)

### step 2
![image](https://github.com/Everything-is-Code/devsecops/assets/10425803/85ba088d-cbdf-4932-be5f-ced50e774eeb)
### step 3
Download the file unzip and execute

```
cd sensor 
unzip sensor-cluster-01.zip 


./sensor.sh 
namespace/stackrox created
Warning: jq not found on your system; unable to parse docker credentials.
Enter username for docker registry at registry.redhat.io: username
Enter password for username @ registry.redhat.io: 

secret/collector-stackrox created
Creating sensor RBAC roles...
serviceaccount/sensor created
clusterrole.rbac.authorization.k8s.io/stackrox:view-cluster created
clusterrolebinding.rbac.authorization.k8s.io/stackrox:monitor-cluster created
role.rbac.authorization.k8s.io/edit created
rolebinding.rbac.authorization.k8s.io/manage-namespace created
clusterrole.rbac.authorization.k8s.io/stackrox:edit-workloads created
clusterrolebinding.rbac.authorization.k8s.io/stackrox:enforce-policies created
clusterrole.rbac.authorization.k8s.io/stackrox:network-policies created
clusterrolebinding.rbac.authorization.k8s.io/stackrox:network-policies-binding created
clusterrole.rbac.authorization.k8s.io/stackrox:update-namespaces created
clusterrolebinding.rbac.authorization.k8s.io/stackrox:update-namespaces-binding created
clusterrole.rbac.authorization.k8s.io/stackrox:create-events created
clusterrolebinding.rbac.authorization.k8s.io/stackrox:create-events-binding created
clusterrole.rbac.authorization.k8s.io/stackrox:review-tokens created
clusterrolebinding.rbac.authorization.k8s.io/stackrox:review-tokens-binding created
Creating sensor network policies...
networkpolicy.networking.k8s.io/sensor created
Pod security policies are not supported on this cluster. Skipping...
Creating upgrader service account
serviceaccount/sensor-upgrader created
clusterrolebinding.rbac.authorization.k8s.io/stackrox:upgrade-sensors created
Creating admission controller secrets...
secret/admission-control-tls created
Creating admission controller RBAC roles...
serviceaccount/admission-control created
role.rbac.authorization.k8s.io/watch-config created
rolebinding.rbac.authorization.k8s.io/admission-control-watch-config created
Creating admission controller network policies...
networkpolicy.networking.k8s.io/admission-control-no-ingress created
Pod security policies are not supported on this cluster. Skipping...
Creating admission controller deployment...
deployment.apps/admission-control created
service/admission-control created
validatingwebhookconfiguration.admissionregistration.k8s.io/stackrox created
Creating secrets for sensor...
secret/sensor-tls created
Creating collector secrets...
secret/collector-tls created
Creating collector RBAC roles...
serviceaccount/collector created
Creating collector network policies...
networkpolicy.networking.k8s.io/collector-no-ingress created
Pod security policies are not supported on this cluster. Skipping...
Creating collector daemon set...
daemonset.apps/collector created
Creating sensor deployment...
deployment.apps/sensor created
service/sensor created
service/sensor-webhook created
secret/helm-effective-cluster-name created
```

### step 4 Verify import
![image](https://github.com/Everything-is-Code/devsecops/assets/10425803/f23e46e8-994d-436b-b48f-d206e8be7554)

![image](https://github.com/Everything-is-Code/devsecops/assets/10425803/b1130b4a-457f-4a05-b32d-79a144ec9c65)
















# Install ACS with the yamls:

## First we create the namespaceinstall ACS: 

oc apply -f bootstrap/base/00_acs/00_create_namespaces 
### namespace/rhacs-operator created
### namespace/stackrox created
### secret/acs-password created

## Second we install the ACS

oc apply -f bootstrap/base/00_acs/01_install_acs       
### subscription.operators.coreos.com/rhacs-operator created
### role.rbac.authorization.k8s.io/acs-cluster-init-role created
### rolebinding.rbac.authorization.k8s.io/acs-cluster-init-rb created
### serviceaccount/acs-gitops created
### central.platform.stackrox.io/stackrox-central-services created

## Third we impor the local cluster:

oc apply -f bootstrap/base/00_acs/02_import_cluster 
### job.batch/acs-init-bundle created
### securedcluster.platform.stackrox.io/stackrox-secured-cluster-services created

## And we are DONE !


# Install ACS With Kustomize and ArgoCD or ArgoCD+ACM:

## We install using argocd and kustomize:

oc apply -k ./bootstrap/base/01_acs_with_argocd
### application.argoproj.io/redhat-acs created

## And we are DONE !

# Install ACS by hand:
### We create the ACS Namespace and secret
```
---
apiVersion: v1
kind: Namespace
metadata:
 labels:
    argocd.argoproj.io/sync-wave: "0"
 name: stackrox
spec: {}
---
kind: Secret
apiVersion: v1
metadata:
  name: acs-password
  namespace: stackrox
  annotations:
    argocd.argoproj.io/sync-wave: "2"
data:
  password: cmVkaGF0MDE=
type: Opaque
```
### From the Operator Hub we search for: 

Advanced Cluster Security for Kubernetes

![Operator Hub](./resources/images/image.png)

### We click install, review the configuration (default is okey) and we click Install again.
### Then we wait for it to finish
![Installed Operator](./resources/images/image2.png)
### We click on view Operator and we go to Central => Create Central and we paste the following yaml:

```
---
apiVersion: platform.stackrox.io/v1alpha1
kind: Central
metadata:
  name: stackrox-central-services
  namespace: stackrox
  annotations:
    argocd.argoproj.io/sync-wave: "3"
    argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true
spec:
  central:
    adminPasswordSecret:
        name: acs-password
    exposure:
      loadBalancer:
        enabled: false
        port: 443
      nodePort:
        enabled: false
      route:
        enabled: true
    db:
      isEnabled: Default
      persistence:
        persistentVolumeClaim:
          claimName: central-db
    persistence:
      persistentVolumeClaim:
        claimName: stackrox-db
  egress:
    connectivityPolicy: Online
  scanner:
    analyzer:
      scaling:
        autoScaling: Enabled
        maxReplicas: 5
        minReplicas: 2
        replicas: 3
    scannerComponent: Enabled
```
### and we click Create
### We go to the stackrox-central-services we click in Resources and we locate the route or in the Namespace Stackrox => Routes:
![ACS Resources](./resources/images/image3.png)
### We log into the ACS Central web console using the credentials admin / redhat01
![ACS Console](./resources/images/image4.png)
### Now we go to integrations => Cluster Init Bundle - Generate Bundle => we type init-bundle and click generate:
![Init Bundle](./resources/images/image5.png)
### We now have to download either the Helm or Kubernetes file for importing the cluster:
![Init Bundle Download](./resources/images/image6.png)
### We apply the yaml on the cluster:
oc apply -f init-bundle-cluster-init-secrets.yaml -n stackrox
### secret/collector-tls created
### secret/sensor-tls created
### secret/admission-control-tls created
### And for the last step we go to the ACS Operator and we click on Create Secured Cluster and we paste the following yaml and click Create:
```
apiVersion: platform.stackrox.io/v1alpha1
kind: SecuredCluster
metadata:
  name: stackrox-secured-cluster-services
  namespace: stackrox
  annotations:
    argocd.argoproj.io/sync-wave: "5"
spec:
  admissionControl:
    listenOnCreates: true
    listenOnEvents: true
    listenOnUpdates: true
  clusterName: local-cluster
  perNode:
    collector:
      collection: KernelModule
      imageFlavor: Regular
    taintToleration: TolerateTaints
```
### We wait for it to import
![Imported Cluster](./resources/images/image7.png)

## and we are DONE!

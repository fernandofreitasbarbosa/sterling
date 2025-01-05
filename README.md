# Sterling
I write here codes that I am using to deploy software mainly in Kubernetes or Red Hat OpenShit

Install the prerequirements software
- Helm
- MS VSCode
- OC or Kubectl command line tools
- Podman to pull and push the images

# Deploying the Sterling Secure Proxy Configuration Manager 

# Download the ssp cm helm chart

helm repo update
helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm
helm pull ibm-helm/ibm-ssp-cm

# Download the image

tar -zxf <ibm-ssp-cm-helm chart-name>  
export SSP_CM_IMAGE=$(grep -w "repository:" ibm-ssp-cm/values.yaml |cut -d '"' -f 2):$(grep -w "tag:" ibm-ssp-cm/values.yaml | cut -d '"' -f 2)

export ENTITLED_REGISTRY=cp.icr.io
export ENTITLED_REGISTRY_USER=cp
export ENTITLED_REGISTRY_KEY=<entitlement_key>

# To donwload the image you can use this command
podman login "$ENTITLED_REGISTRY" -u "$ENTITLED_REGISTRY_USER" -p "$ENTITLED_REGISTRY_KEY"

# or you can create a secret in the your Kubernetes or OpenShift Cluster with this command to pull the images directly from the IBM Container Registry
kubectl/oc create secret docker-registry <any_name_for_the_secret> --docker-username=$ENTITLED_REGISTRY_USER --docker-password=$ENTITLED_REGISTRY_KEY --docker-email=<your_docker_email_address> --docker-server=$ENTITLED_REGISTRY -n <your namespace/project name>

# After that update the helm chart image pull secret configurations using `image.imageSecrets` parameter with the above secret name

podman pull $SSP_CM_IMAGE

# login to OpenShift Cluster
oc login --token=sha256~hiGlIDe1mvbeyqfsfsfsfsfstbygRJ_dcQLuUv80 --server=https://c100-e.us-south.containers.cloud.ibm.com:31901

# create a new project to deploy SSP CM
oc new-project sspcm

# creating the security context constrainsts
cd ibm-ssp-cm/ibm_cloud_pak/pak_extensions/pre-install/clusterAdministration
./createSecurityClusterPrereqs.sh
cd ibm-ssp-cm/ibm_cloud_pak/pak_extensions/pre-install/namespaceAdministration
./createSecurityNamespacePrereqs.sh sspcm

# Creating Secrets
cd ibm-ssp-cm/ibm_cloud_pak/pak_extensions/pre-install/secret
oc create -f ibm-ssp-cm-secret.yaml

# Lets pusch the image to the OpenShift Registry
oc patch configs.imageregistry.operator.openshift.io/cluster --patch '{"spec":{"defaultRoute":true}}' --type=merge
HOST=$(oc get route default-route -n openshift-image-registry --template='{{ .spec.host }}')
podman login -u $(oc whoami -t) -p $(oc whoami -t) --tls-verify=false $HOST
podman tag cp.icr.io/cp/ibm-ssp-cm/ssp-cm-docker-image:6.2.0.0.01 $HOST/sspcm/cm:6.2.0.0.01
podman push $HOST/sspcm/cm:6.2.0.0.01

# now we need need to edit the values.yaml file and fill the follow fieds.
cp values.yaml override-ssp-cm.yaml
vi override-ssp-cm.yaml

helm install ssp62-cm -f override-ssp-cm.yaml  --debug .

# open the web browser type the url
https://ssp62-cm-ibm-ssp-cm-sspcm.ffb-wks-sterling-dal12-c3-8c1e167e9d701a9f52a0a22606bde374-0000.us-south.containers.appdomain.cloud

# Deploying the Sterling Secure Proxy Engine 

# Download the ssp engine helm chart

helm repo update
helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm       helm pull ibm-helm/ibm-ssp-engine

# Download the image

tar -zxf <ibm-ssp-engine-helm chart-name> 

export SSP_ENGINE_IMAGE=$(grep -w "repository:" ibm-ssp-engine/values.yaml |cut -d '"' -f 2):$(grep -w "tag:" ibm-ssp-engine/values.yaml | cut -d '"' -f 2)

export ENTITLED_REGISTRY=cp.icr.io
export ENTITLED_REGISTRY_USER=cp
export ENTITLED_REGISTRY_KEY=<entitlement_key>

#To donwload the image you can use this command
podman login "$ENTITLED_REGISTRY" -u "$ENTITLED_REGISTRY_USER" -p "$ENTITLED_REGISTRY_KEY"

#or you can create a secret in the your Kubernetes or OpenShift Cluster with this command to pull the images directly from the IBM Container Registry
kubectl/oc create secret docker-registry <any_name_for_the_secret> --docker-username=$ENTITLED_REGISTRY_USER --docker-password=$ENTITLED_REGISTRY_KEY --docker-email=<your_docker_email_address> --docker-server=$ENTITLED_REGISTRY -n <your namespace/project name>

#After that update the helm chart image pull secret configurations using `image.imageSecrets` parameter with the above secret name

docker pull $SSP_ENGINE_IMAGE

# Deploying the Sterling External Authentication Server 

# Download the ssp cm helm chart

helm repo update
helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm       helm pull ibm-helm/ibm-seas 

# Download the image

tar -zxf <ibm-seas-helm chart-name> 
export SEAS_IMAGE=$(grep -w "repository:" ibm-seas/values.yaml |cut -d '"' -f 2):$(grep -w "tag:" ibm-seas/values.yaml | cut -d '"' -f 2)

export ENTITLED_REGISTRY=cp.icr.io
export ENTITLED_REGISTRY_USER=cp
export ENTITLED_REGISTRY_KEY=<entitlement_key>

#To donwload the image you can use this command
podman login "$ENTITLED_REGISTRY" -u "$ENTITLED_REGISTRY_USER" -p "$ENTITLED_REGISTRY_KEY"

#or you can create a secret in the your Kubernetes or OpenShift Cluster with this command to pull the images directly from the IBM Container Registry
kubectl/oc create secret docker-registry <any_name_for_the_secret> --docker-username=$ENTITLED_REGISTRY_USER --docker-password=$ENTITLED_REGISTRY_KEY --docker-email=<your_docker_email_address> --docker-server=$ENTITLED_REGISTRY -n <your namespace/project name>

#After that update the helm chart image pull secret configurations using `image.imageSecrets` parameter with the above secret name

docker pull $SEAS_IMAGE

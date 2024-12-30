# Sterling
I write here codes that I am using to deploy software mainly in Kubernetes or Red Hat OpenShit

Install the prerequirements software
- Helm
- MS VSCode
- OC or Kubectl command line tools

# Deploying the Sterling Secure Proxy Configuration Manager 

# Download the ssp cm helm chart

helm repo update
helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm
helm pull ibm-helm/ibm-ssp-cm

# Download the image

tar -zxf <ibm-ssp-ps-helm chart-name>  
export SSP_CM_IMAGE=$(grep -w "repository:" ibm-ssp-cm/values.yaml |cut -d '"' -f 2):$(grep -w "tag:" ibm-ssp-cm/values.yaml | cut -d '"' -f 2)

export ENTITLED_REGISTRY=cp.icr.io
export ENTITLED_REGISTRY_USER=cp
export ENTITLED_REGISTRY_KEY=<entitlement_key>

docker login "$ENTITLED_REGISTRY" -u "$ENTITLED_REGISTRY_USER" -p "$ENTITLED_REGISTRY_KEY"









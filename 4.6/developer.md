# Overview of the demo
1. build sample app with oc command
2. check logging and monitoring
2. build sample backend with odo and spring boot with swagger support
3. build sample 3scale api interface

# Preparation
```
export BACKEND_NAMESPACE=demo-backend
export THREESCALE_NAMESPACE=3scale-apimanager
```

# OC Simple Application Deploy
```
oc new-app -L //Check existing supported languages templates

oc new-project "$BACKEND_NAMESPACE"

oc new-app https://github.com/sclorg/cakephp-ex

oc expose svc cakephp-ex

oc get route -n "$BACKEND_NAMESPACE" cakephp-ex -o jsonpath='{.spec.host}'

export SIMPLE_APP_HOSTNAME=http://"$(oc get route -n "$BACKEND_NAMESPACE" cakephp-ex -o jsonpath='{.spec.host}')"

curl -sfk $SIMPLE_APP_HOSTNAME
```

https://docs.openshift.com/container-platform/4.6/builds/build-strategies.html

https://docs.openshift.com/container-platform/4.5/openshift_images/using_images/images-other-jenkins.html

# ODO Backend application deploy
```
oc project "$BACKEND_NAMESPACE"

## Install ODO
sudo curl -L https://mirror.openshift.com/pub/openshift-v4/clients/odo/latest/odo-linux-amd64 -o /usr/local/bin/odo
sudo chmod +x /usr/local/bin/odo

## Using ODO
odo catalog list components //To check which framework supported
git clone https://github.com/openshift/nodejs-ex
cd nodejs-ex
odo create nodejs --app=nodejs_app / odo create openshift/nodejs:8
odo push
odo url create --port 8080
odo push
odo url list
curl <url>
<code changes>
odo push

## Play with loadtest for cakephp
## Checking Monitoring of cakephp

git clone https://github.com/developertyrone/spring-petclinic-rest
cd spring-petclinic-rest
odo create java-springboot petclinic --app=backend 
odo config help
odo config set --env PLATFORM=OCP --env APP=JAVA
odo url create api --port 9966
odo push
oc get route

## Checking Logging

---------------------------EXTRA-----------------------------------------
## odo create java-springboot spring-petclinic-rest --app=api_backend //For configuration udpate
## odo config set Ports 9966/TCP, odo push //For configuration update

## Adding storage
odo storage create mystore --path=/data --size=1G
odo push
oc rsh <app-pod> lsblk
odo storage list
odo storage delete <storage_name>
odo storage list
odo push
---------------------------EXTRA-----------------------------------------
```

# 3scale deployment
```
rpm -ivh https://github.com/3scale-labs/3scale_toolbox_packaging/releases/download/v0.18.0/3scale-toolbox-0.18.0-1.el7.x86_64.rpm

// access the 3scale portal and retrieve the service management api token

3scale remote -k add 3scale-demo "https://<token>@3scale-admin.apps.cluster-453c.453c.sandbox995.opentlc.com/"

oc create secret generic 3scale-toolbox -n "$BACKEND_NAMESPACE" --from-file="$HOME/.3scalerc.yaml"

export BACKEND_HOSTNAME="$(oc get route -n "$BACKEND_NAMESPACE" api-petclinic -o jsonpath='{.spec.host}')"

git clone https://github.com/developertyrone/3scale-toolbox-jenkins-samples

cd 3scale-toolbox-jenkins-samples

// browse https://3scale-admin.apps.cluster-453c.453c.sandbox995.opentlc.com/buyers/accounts/3 

export ONPREM_DEVELOPER_ACCOUNT_ID=3

oc process -f 3scale-toolbox-jenkins-samples/onprem-custom-backend/setup.yaml \
           -p DEVELOPER_ACCOUNT_ID="$ONPREM_DEVELOPER_ACCOUNT_ID" \
           -p PRIVATE_BASE_URL="http://$BACKEND_HOSTNAME" \
           -p PRIVATE_BASE_OPENAPI="/v2/api-docs" \
           -p PRIVATE_BASE_INTEGRATION_TEST="/api/pets" \
           -p TARGET_INSTANCE=3scale-demo \
           -p DISABLE_TLS_VALIDATION=yes \
           -p NAMESPACE="$BACKEND_NAMESPACE" |oc apply -f -

oc -n "$BACKEND_NAMESPACE" start-build onprem-custom-backend-3scale-demo

```

# Deploy with VSCODE

- https://api.cluster-7eb9.7eb9.sandbox31.opentlc.com:6443
- admin
- asfasfasf

- app-from-vscode
- https://github.com/openshift/nodejs-ex

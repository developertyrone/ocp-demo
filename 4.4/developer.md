# Overview of the demo
1. build sample app with oc command
2. build sample backend with odo and spring boot with swagger support
3. build sample 3scale api interface

# Preparation
-   export BACKEND_NAMESPACE=demo-backend
-   export 3SCALE_NAMESPACE=3scale-apimanager

# OC Simple Application Deploy
```
oc new-app -L //Check existing supported languages templates

oc new-project "$BACKEND_NAMESPACE"

oc new-app https://github.com/sclorg/cakephp-ex

oc expose svc cakephp-ex

oc get route -n "$BACKEND_NAMESPACE" cakephp-ex -o jsonpath='{.spec.host}'

export SIMPLE_APP_HOSTNAME=http://"$(oc get route -n "$BACKEND_NAMESPACE" cakephp-ex -o jsonpath='{.spec.host}')"

curl -sfk $SIMPLE_APP_HOSTNAME

echo $SIMPLE_APP_HOSTNAME
```

https://docs.openshift.com/container-platform/4.6/builds/build-strategies.html

https://docs.openshift.com/container-platform/4.5/openshift_images/using_images/images-other-jenkins.html

# ODO Backend application deploy
```
odo catalog list components //To check whether
git clone https://github.com/openshift/nodejs-ex
odo create nodejs --app=nodejs_app / odo create openshift/nodejs:8
odo push
odo url create --port 8080
odo push
odo url list
curl <url>
<code changes>
odo push


git clone https://github.com/developertyrone/spring-petclinic-rest
cd spring-petclinic-rest
odo create java-springboot spring-petclinic-rest --app=api_backend --port 9966/tcp
odo config help
odo url create api --port 9966 --path /petclinic, odo push
oc get route

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
```

# Jenkins deployment
```
oc new-app -n "$BACKEND_NAMESPACE" --template=jenkins-ephemeral --name=jenkins -p MEMORY_LIMIT=2Gi

oc set env -n "$BACKEND_NAMESPACE" dc/jenkins JENKINS_OPTS=--sessionTimeout=86400

//oc set env -n "$BACKEND_NAMESPACE" dc/jenkins INSTALL_PLUGINS=http_request:1.8.24
 
```

# 3scale deployment
```
// access the 3scale portal and retrieve the service management api token

3scale remote -k add 3scale-demo "https://874793196e1ea27bea47eef08c66b98b2b83ba1c82c2d4369cdcf976af1aa35d@3scale-admin.apps.dev.ocp.local/"

oc create secret generic 3scale-toolbox -n "$BACKEND_NAMESPACE" --from-file="$HOME/.3scalerc.yaml"

export BACKEND_HOSTNAME="$(oc get route -n "$BACKEND_NAMESPACE" api-spring-petclinic-rest -o jsonpath='{.spec.host}')"


git clone https://github.com/developertyrone/3scale-toolbox-jenkins-samples

cd 3scale-toolbox-jenkins-samples

oc process -f 3scale-toolbox-jenkins-samples/onprem-custom-backend/setup.yaml \
           -p DEVELOPER_ACCOUNT_ID="$ONPREM_DEVELOPER_ACCOUNT_ID" \
           -p PRIVATE_BASE_URL="http://$BACKEND_HOSTNAME" \
           -p PRIVATE_BASE_OPENAPI="/petclinic/v2/api-docs" \
           -p PRIVATE_BASE_INTEGRATION_TEST="/petclinic/api/pets" \
           -p TARGET_INSTANCE=3scale-demo \
           -p DISABLE_TLS_VALIDATION=yes \
           -p NAMESPACE="$BACKEND_NAMESPACE" |oc apply -f -

oc start-build onprem-custom-backend-3scale-demo

```

# Deploy with VSCODE

- https://api.cluster-7eb9.7eb9.sandbox31.opentlc.com:6443
- admin
- TLgONyeVpUUA8oN5

- app-from-vscode
- https://github.com/openshift/nodejs-ex

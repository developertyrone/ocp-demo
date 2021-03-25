odo catalog list components

# OC App Deploy
oc new-project project4

oc new-app https://github.com/sclorg/cakephp-ex

oc expose svc cakephp-ex

https://docs.openshift.com/container-platform/4.6/builds/build-strategies.html

https://docs.openshift.com/container-platform/4.5/openshift_images/using_images/images-other-jenkins.html

# ODO App deploy
odo login -u developer -p developer
odo project create myproject
mkdir project4 && project4
git clone https://github.com/openshift/nodejs-ex
odo create nodejs --app=nodejs_app / odo create openshift/nodejs:8
odo push
odo url create --port 8080
odo push
odo url list
curl <url>
<code changes>
odo push

git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
odo create java-springboot spring-petclinic --app=java_app
odo push
odo url create --port 8080

# Adding storage
odo storage create mystore --path=/data --size=1G
odo push
oc rsh <app-pod> lsblk
odo storage list
odo storage delete <storage_name>
odo storage list
odo push


Demo
(VS code)
https://api.cluster-7eb9.7eb9.sandbox31.opentlc.com:6443
admin
TLgONyeVpUUA8oN5

app-from-vscode
https://github.com/openshift/nodejs-ex

New url

(Azure DevOps)

# Output modified build config in YAML
oc set env bc/sample-build STORAGE_DIR=/data -o yaml

# Update all containers in all replication controllers in the project to have
oc set env rc --all ENV=prod

# Import environment from a secret
oc set env --from=secret/mysecret dc/myapp

# Import environment from a config map with a prefix
oc set env --from=configmap/myconfigmap --prefix=MYSQL_ dc/myapp



oc delete all --selector app=cakephp

https://piotrminkowski.com/2021/02/05/java-development-on-openshift-with-odo/
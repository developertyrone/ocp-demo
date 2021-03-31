 # Locust
 ```
 mkdir -p loadtest
 oc new-project loadtest
 git clone https://github.com/developertyrone/locust-openshift
 oc process -p NAMESPACE="loadtest" -f master-deployment.yaml | oc create -f -
 oc process -p NAMESPACE="loadtest" -f slave-deployment.yaml | oc create -f -
 oc get route -n "loadtest" app -o jsonpath='{.spec.host}'

############# hostonly.py #########################

from locust import HttpLocust, TaskSet, task
class UserTasks(TaskSet):
    @task
    def index(self):
        self.client.get("/")

class WebsiteUser(HttpLocust):
    task_set = UserTasks


####################################################

 ./seed.sh loadtest samples/hostonly.py $SIMPLE_APP_HOSTNAME 
```
 # Vegeta
 ```
oc run vegeta --rm --attach --restart=Never --image="peterevans/vegeta"  -- sh -c "echo 'GET $SIMPLE_APP_HOSTNAME' | vegeta attack -rate=10 -duration=5s | tee results.bin | vegeta report"

OR 

oc run vegeta  --restart=Never --image="peterevans/vegeta" --env=TARGETURL=$SIMPLE_APP_HOSTNAME  -- sleep 3600
oc rsh vegeta sh -c "echo 'GET $SIMPLE_APP_HOSTNAME' | vegeta attack -rate=10 -duration=5s | tee results.bin | vegeta report ;cat results.bin | vegeta plot > plot.html"
oc rsync  vegeta:/plot.html  ./
oc delete pod --force vegeta
```
 # K6


# Overview of the demo
# Build cluster

# Test basic operation
oc get nodes

oc top nodes

oc get machines -A

oc get pods 

oc get project

oc new-project project1

htpasswd -c -B -b users.htpasswd user1 MyPassword!
htpasswd -B -b users.htpasswd user2 MyPassword!
htpasswd -B -b users.htpasswd admin TLgONyeVpUUA8oN5
oc create secret generic htpass-secret --from-file=htpasswd=users.htpasswd -n openshift-config

[add users ]
oc apply -f - <<EOF
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: my_htpasswd_provider
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret
EOF

[add roles ]
oc adm policy add-cluster-role-to-user cluster-admin admin 2>/dev/null
oc adm policy add-cluster-role-to-user view user1
oc adm policy add-role-to-user edit user2 -n project1
[project isolation]

[project isolation test]

# Build with Logging Stack
```
oc apply -f - << EOF
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-operators-redhat
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-monitoring: "true"
---
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-logging
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-monitoring: "true"
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-operators-redhat
  namespace: openshift-operators-redhat
spec: {}
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: elasticsearch-operator
  namespace: openshift-operators
spec:
  channel: "4.4"
  installPlanApproval: Automatic
  name: elasticsearch-operator
  source: "$(oc get packagemanifest elasticsearch-operator -n openshift-marketplace -o jsonpath='{.status.catalogSource}')"
  sourceNamespace: "$(oc get packagemanifest elasticsearch-operator -n openshift-marketplace -o jsonpath='{.status.catalogSourceNamespace}')"
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-logging
  namespace: openshift-logging
spec:
  targetNamespaces:
  - openshift-logging
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: cluster-logging
  namespace: openshift-logging
spec:
  channel: "4.4"
  installPlanApproval: Automatic
  name: cluster-logging
  source: "$(oc get packagemanifest cluster-logging -n openshift-marketplace -o jsonpath='{.status.catalogSource}')"
  sourceNamespace: "$(oc get packagemanifest cluster-logging -n openshift-marketplace -o jsonpath='{.status.catalogSourceNamespace}')"
EOF
```
```
cat <<EOF | oc apply -f -
apiVersion: "logging.openshift.io/v1"
kind: "ClusterLogging"
metadata:
  name: "instance"
  namespace: "openshift-logging"
spec:
  managementState: "Managed"
  logStore:
    type: "elasticsearch"  
    retentionPolicy: 
      application:
        maxAge: 1d
      infra:
        maxAge: 7d
      audit:
        maxAge: 7d
    elasticsearch:
      nodeCount: 2 
      storage:
        storageClassName: "gp2" 
        size: 100G
      resources: 
        requests:
          memory: "8Gi"
      proxy: 
        resources:
          limits:
            memory: 256Mi
          requests:
             memory: 256Mi
      redundancyPolicy: "SingleRedundancy"
  visualization:
    type: "kibana"  
    kibana:
      replicas: 1
  curation:
    type: "curator"
    curator:
      schedule: "30 3 * * *" 
  collection:
    logs:
      type: "fluentd"  
      fluentd: {}
EOF
```

3. Build with monitoring

4. 3scale and SSO


[Logging]
oc new-app https://github.com/sclorg/cakephp-ex
DEMO
(EFK)
search cakephp on .all


[Monitoring]
Deploy Monitoring Stacks
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    techPreviewUserWorkload:
      enabled: true
---
apiVersion: v1
kind: Namespace
metadata:
  name: project1

---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: prometheus-example-app
  name: prometheus-example-app
  namespace: project1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus-example-app
  template:
    metadata:
      labels:
        app: prometheus-example-app
    spec:
      containers:
      - image: quay.io/brancz/prometheus-example-app:v0.2.0
        imagePullPolicy: IfNotPresent
        name: prometheus-example-app
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: prometheus-example-app
  name: prometheus-example-app
  namespace: project1
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
    name: web
  selector:
    app: prometheus-example-app
  type: ClusterIP

oc expose service  prometheus-example-app 
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
	k8s-app: prometheus-example-monitor
  name: prometheus-example-monitor
  namespace: project1
spec:
  endpoints:
  - interval: 30s
    port: web
    scheme: http
  selector:
    matchLabels:
      app: prometheus-example-app	
---
oc -n project1 get servicemonitor
---
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: example-alert-http-requests-total
  namespace: project1
spec:
  groups:
  - name: example
    rules:
      - alert: VersionAlert
        expr: http_requests_total{job="prometheus-example-app"} > 20

DEMO
(Prometheus)
(Alert manager)


# openshift-gitops-demo
For showcasing the features of autoscale of an application with backend as mariadb


# Follow the steps

git clone https://github.com/binssam/openshift-gitops-demo.git

oc apply -f safe-demo.yaml
oc get pods -w
oc logs -l app=demo-web --tail=20

oc get route demo-web-route -o custom-columns=URL:.spec.host
oc delete pod -l app=mariadb-db

# Add Resource Limits and the HPA
Will use the demo-web-app Deployment to include a deliberately low CPU request (so it scales easily during the demo) and create the HPA object.

oc apply -f autoscale.yaml

oc get hpa demo-web-hpa -w

URL=$(oc get route demo-web-route -o custom-columns=URL:.spec.host | tail -n 1)

while true; do curl -s "http://$URL" > /dev/null; echo -n "."; done

# To clean up the namespace

oc delete all,pvc,secret,configmap -l app=demo-web
oc delete all,pvc,secret,configmap -l app=mariadb-db

# Get your Argo CD Login
The operator automatically provisions an Argo CD instance. Retrieve your admin password and the web URL via the CLI:

# Get the password
oc extract secret/openshift-gitops-cluster -n openshift-gitops --to=-

# Get the Argo CD URL
oc get route openshift-gitops-server -n openshift-gitops -o custom-columns=URL:.spec.host

# Connecting Git to OpenShift
We need to tell Argo CD to watch your GitHub repository and sync it to your OpenShift cluster.

Make sure to update the destination.namespace matches the OpenShift project you are working in.


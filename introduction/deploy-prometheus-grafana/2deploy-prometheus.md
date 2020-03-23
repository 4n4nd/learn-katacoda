

First let's create a new namespace to deploy our monitoring tools.

`oc new-project pad-monitoring`{{execute}}

Now we need to edit the configmap for our prometheus deployment, so that Prometheus knows to scrape our demo application for metrics.
To do that, we need to add the following section:

we will store the configmap template in `prometheus-configmap.yaml`{{open}}

<pre class="file" data-filename="prometheus-configmap.yaml" data-target="replace">
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-demo
data:
  prometheus.yml: |
    global:
      external_labels:
        monitor: prometheus
    scrape_configs:
      - job_name: 'prometheus'

        static_configs:
          - targets: ['localhost:9090']
            labels:
              group: 'prometheus'
</pre>

The second half defines the new targets that Prometheus should scrape data from. In this example, we have defined one new target below.

<pre class="file" data-filename="prometheus-configmap.yaml">
          - targets: ['metrics-demo-app-metrics-demo.[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com']
            labels:
              group: 'pad'
</pre>

Once we have added it to the configMap, we can go ahead and create this configmap in our namespace
`oc create -f prometheus-configmap.yaml -n pad-monitoring`{{execute}}

and deploy it using the following command.
`oc process -f deploy-prometheus.yaml | oc apply -n pad-monitoring -f -`{{execute}}

To see the url for your Prometheus instance, run the following command:
`echo -e "http://$(oc get route prometheus-demo-route -o jsonpath='{.spec.host}' -n pad-monitoring)"`{{execute}}

or you can use the dashboard to check on the Prometheus deployment.

The credentials to access the openshift console are `developer/developer`

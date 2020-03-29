
### Create Namespace

First let's create a new namespace to deploy our monitoring tools. This will be the project where all our monitoring applications
will be deployed. Let's call it `pad-monitoring`

`oc new-project pad-monitoring`{{execute}}


### Configure Prometheus for our application

In Openshift, we use ConfigMaps to manage configurations for our applications. ([More info](https://docs.openshift.com/container-platform/3.11/dev_guide/configmaps.html#overview))

* We will use the following configmap to set up our Prometheus instance.

<pre class="file" data-filename="~/prometheus-configmap.yaml" data-target="replace">
apiVersion: v1          # Click on 'Copy to Editor' --->
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

* Click on `Copy to Editor` for the above yaml block, to copy it to the editor.
This will replace all the text in the editor with the above yaml text block

* Now we need to edit the configmap for our prometheus deployment, so that Prometheus knows to scrape our demo application for metrics.
To do that, we need to add the following section:

<pre class="file" data-filename="~/prometheus-configmap.yaml">
          - targets: ['metrics-demo-app-metrics-demo.[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com'] # Click on 'Copy to Editor'->
            labels:
              group: 'pad'
</pre>

In the above yaml block, we have defined a new targets list for our prometheus to collect metrics from.

This targets list has the hostname for our demo application, i.e. `metrics-demo-app-metrics-demo.[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com`

By default Prometheus collects metrics from the `/metrics` http endpoint. ([More info](https://prometheus.io/docs/prometheus/latest/configuration/configuration/) on Prometheus Configuration)

Above, you can see that we have added a label `group: 'pad'` <br>
So all the metrics from our demo application can be
queried with a singe PromQL query, i.e. `{group="pad"}`

The configmap file should be stored in `~/prometheus-configmap.yaml`{{open}}

### Deploying Prometheus

Once we have updated the configuration with our new target, we can go ahead and create this configmap in our namespace: <br>
`oc create -f prometheus-configmap.yaml -n pad-monitoring`{{execute}}

and deploy it using the following command: <br>
`oc process -f deploy-prometheus.yaml | oc apply -n pad-monitoring -f -`{{execute}}

To see the url for your Prometheus instance, run the following command:
`echo -e "http://$(oc get route prometheus-demo-route -o jsonpath='{.spec.host}' -n pad-monitoring)"`{{execute}}

you can use the [Openshift dashboard](https://console-openshift-console-[[HOST_SUBDOMAIN]]-443-[[KATACODA_HOST]].environments.katacoda.com/k8s/ns/pad-monitoring/deploymentconfigs/prometheus-demo) to check on the Prometheus deployment.

The credentials to access the openshift console are `developer/developer`


### Check application metrics
* Once Prometheus is deployed, you can click [here](http://prometheus-demo-route-pad-monitoring.[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com/graph?g0.range_input=1h&g0.expr={group%3D"pad"}) to see all the metrics for the demo application

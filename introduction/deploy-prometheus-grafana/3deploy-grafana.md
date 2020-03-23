Next, we will deploy Grafana and connect it to our Prometheus instance, so we are able to visualize metrics better.

We can just directly deploy the vanilla container image from Docker Hub using the following command:
`oc new-app grafana/grafana:6.6.1 -n pad-monitoring`{{execute}}

Then we need to expose the service so that it is accessible outside of the cluster.

`oc expose svc/grafana -n pad-monitoring`{{execute}}

To see the url for your Grafana instance, run the following command:
`echo -e "https://$(oc get route grafana -o jsonpath='{.spec.host}' -n pad-monitoring)"`{{execute}}

The default username and password for grafana are: `admin/admin`

Instructions to add Prometheus as a data source:

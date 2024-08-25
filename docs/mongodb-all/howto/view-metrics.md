---
orphan: true
---


# How to view metrics

You can view Charmed MongoDB-{{charm}} metrics in two ways:

* by querying the metric endpoint directly
* in Grafana


## Querying the metrics endpoint

Charmed MongoDB comes with [MongoDB Exporter](https://github.com/percona/mongodb_exporter) and provides replication and cluster metrics. The metrics can be queried by accessing the `http://<unit-ip>:9216/metrics` endpoint.

To quickly view them, do the following:

1. Identify the unit to query by entering `juju status`. Choose a unit to gather metrics from and copy its `Public Address`.

2. Use the `curl` tool to query the metrics endpoint:

   ```none
   curl http://<unit-ip>:9216/metrics
   ```

   This should show a long set of metrics.


## Accessing metrics with Grafana

Charmed MongoDB provides integration with the [Canonical Observability Stack (COS)](https://charmhub.io/topics/canonical-observability-stack), which allows you to view the metrics in a graphical interface (GUI). The Observability stack integrates with VM charm deployments via the `grafana-agent` sidecar charm. This sidecar charm integrates with all the relevant relations in a COS deployment.

To view the GUI:

1. Deploy the `cos-lite` bundle in a Kubernetes environment by following the [deployment tutorial](https://charmhub.io/topics/canonical-observability-stack/tutorials/install-microk8s). Wait for the `cos-lite` bundle to become `active` and then `idle` (check using `juju status`).

2. Switch back to your model that hosts Charmed MongoDB:

    {{switch_to_model}}

3. Deploy the `grafana-agent` sidecar charm, and relate it to Charmed MongoDB:

    ```none
    juju deploy grafana-agent
    juju integrate grafana-agent mongodb
    ```

    Watch `juju status` and wait for the model to become idle, `grafana-agent` should eventually go into the error state because it requires relations from the COS bundle.

4. Consume offers from the COS bundle:

    ```none
    juju consume <k8s-controller-name>:admin/cos.prometheus-scrape
    juju consume <k8s-controller-name>:admin/cos.alertmanager-karma-dashboard
    juju consume <k8s-controller-name>:admin/cos.grafana-dashboards
    juju consume <k8s-controller-name>:admin/cos.loki-logging
    juju consume <k8s-controller-name>:admin/cos.prometheus-receive-remote-write
    ```

{{integrate_relate}}

{{wait_idle}}

7. Navigate to the Grafana dashboard. To find the dashboard URL, switch to the Kubernetes model hosting the `cos-lite` bundle and show all applications:

    ```none
    juju switch <Kubernetes-controller>
    juju status
    ```

    Under the report of `juju status`, the IP address of the Grafana GUI should be listed as the `Public Address` for the application `grafana`. Copy this address and navigate to it in your browser.

8. In the login form, enter the username `admin`.

9. To retrieve the password, run:

    ```none
    juju run grafana/0 get-admin-password --wait
    ```

    Use this password to access the Grafana dashboard.

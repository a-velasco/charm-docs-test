---
myst:
  substitutions:
    charm: "K8s"
    switch_to_model: |
      ```{code} none
      juju switch <model with Charmed MongoDB K8s>
      ```

    integrate_relate: |
      5. Relate Charmed MongoDB to the applications within the COS bundle:

          ```
          juju relate mongodb-k8s prometheus-scrape
          juju relate mongodb-k8s loki-logging
          juju relate mongodb-k8s grafana-dashboards
          ```

    wait_idle: "6. Wait for the model to become `idle` in `juju~status`."

---

:::{include} ../../mongodb-all/howto/view-metrics.md
:::

---
myst:
  substitutions:
    charm: "VM"
    switch_to_model: |
      ```none
      juju switch <VM-controller>
      ```

    integrate_relate: |
      5. Integrate the COS bundle with the `grafana-agent`:

          ```none
          juju integrate grafana-agent prometheus-receive-remote-write
          juju integrate grafana-agent loki-logging
          juju integrate grafana-agent grafana-dashboards
          ```

    wait_idle: "6. Wait for `grafana-agent` to show `active` and then `idle` status in `juju~status`."

---

:::{include} ../../mongodb-all/howto/view-metrics.md
:::

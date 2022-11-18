# hypershift-rhobs-prototype

This repository is intended to test the triaged shipping of HyperShift Hosted Control Planes' metrics to RHOBS.

- Create an OSD cluster in staging.
- Create Hosted Control Planes, preferably two for testing.
- Make sure you have the latest version of Observability Operator running (v17.0 at the time of writing this). If not, update the OO subscription channel to **development** and it's catalogsource manifest image to point to the latest SHA in [quay](https://quay.io/repository/rhobs/observability-operator-catalog?tab=tags&tag=latest). The OO CSV will then automatically updated in a few minutes.
- The hypershift-tenant credenials currently reside in [Vault](https://vault.devshift.net/ui/vault/secrets/osd-sre/show/rhobs-hypershift-platform-staging). Create a secret in the `openshift-observability-operator` namespace.

```bash
oc create secret generic rhobs-observatorium-credentials -n openshift-observability-operator \
--from-literal=client-id=observatorium-hypershift-platform-staging \
--from-literal=client-secret=<CLIENT_SECRET> \
--from-literal=receiver-url=https://observatorium-mst.api.stage.openshift.com/api/metrics/v1/hypershift-platform/api/v1/receive
```

- Setup authentication with RHOBS.
  - **Option 1:** Deploy [token-refresher](./token-refresher/), this acts as a middle-man to establish a connect betweeb our Management Cluster and Observatorium. Then deploy the [MonitoringStackCR](./MonitoringStack/MonitoringStackCR.yaml) in the `openshift-observability-operator` namespace.
  - **Option 2:** Leverage Oauth and deploy the [MonitoringStackOauth](./MonitoringStack/MonitoringStackOauth.yaml) CR in the `openshift-observability-operator` namespace.
- The ServiceMonitor and PodMonitor objects created by HyperShift are of the API version `monitoring.coreos.com/v1`; for the OO to be able to recognize these you will have to duplicate those objects to be of the version `monitoring.rhobs/v1` instead.
- Switch project to your HCP namespace and run:

```bash
oc get servicemonitors.monitoring.coreos -o yaml | grep '^[- ] ' | sed -e 's/^- /---\n/g' -e 's/^  //g' -e 's@monitoring.coreos.com/v1@monitoring.rhobs/v1@g' | oc create -f -

oc get podmonitors.monitoring.coreos -o yaml | grep '^[- ] ' | sed -e 's/^- /---\n/g' -e 's/^  //g' -e 's@monitoring.coreos.com/v1@monitoring.rhobs/v1@g' | oc create -f -
```

- For Observability Operator to be able to scrape metrics from HCP namespace, it should have access to all pods in the HCP namespace. The _openshift-monitoring_ network policy in HCP namespaces allows ingress traffic to all of its pods from all pods belonging to the namespace which has the `network.openshift.io/policy-group: monitoring label`. Label the namespace:

```bash
oc label namespace openshift-observability-operator network.openshift.io/policy-group=monitoring
```

- If you did everything right, you will now be able to see your HCPs' metrics via <https://promlens.stage.devshift.net>.

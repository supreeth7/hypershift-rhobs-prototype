# hypershift-rhobs-prototype
This repository is intended to test the triaged shipping of HyperShift Hosted Control Planes' metrics to RHOBS.

- Make sure you have the latest version of Observability Operator running. If not, update the OO catalogsource manifest's image to point to the latest SHA in [quay](https://quay.io/repository/rhobs/observability-operator-catalog?tab=tags&tag=latest).
- The hypershift-tenant credenials currently reside in Vault. Create a secret in the `openshift-observability-operator` namespace. 
```
oc create secret generic rhobs-observatorium-credentials -n openshift-observability-operator \
--from-literal=client-id=observatorium-hypershift-platform-staging \
--from-literal=client-secret=<CLIENT_SECRET> \
--from-literal=receiver-url=https://observatorium-mst.api.stage.openshift.com/api/metrics/v1/hypershift-platform/api/v1/receive
```
- Deploy token-refresher, this acts as a middle-man to establish a connect betweeb our Management Cluster and Observatorium.
- Deploy the MonitoringStack CR in the `openshift-observability-operator` namespace.
- The ServiceMonitor and PodMonitor objects created by HyperShift are of the API version `monitoring.coreos.com/v1`; for the OO to be able to recognize these you will have to duplicate those objects to be of the version `monitoring.rhobs/v1` instead.
- If you did everything right, you will now be able to see your HCPs' metrics via https://promlens.stage.devshift.net.

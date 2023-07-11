# TAP Kube State Metrics

This is a proof of concept for monitoring TAP in prometheus using the custom resource support in kube state metrics.

# Sample deployment
1. Prepare values file based on [this file](./values-example.yaml) and name it values.yaml
   - you will need to update the ingress settings to match your environment
   - you will need to update the admin password as per your needs
2. Add the helm repo locally
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   ```
3. Install the chart
   ```bash
   helm upgrade --install tap-monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace -f values.yaml
   ```

# Example PromQL Queries you can run
| Use Case                                               | PromQL Query Example                                                                                                 |
| ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| NUMBER OF WORKLOADS USING EACH BUILDPACK               | count by (id) (topk(1000, count by (id, workload) (kpack_build_involved_buildpacks)))                                |
| NUMBER OF SUCCESSFUL BUILD PER WORKLOAD                | count by (workload) (kpack_build_status{status="True", type="Succeeded"})                                            |
| NUMBER OF FAILED BUILDS PER WORKLOAD                   | count by (workload) (kpack_build_status{status="Failed", type="Succeeded"})                                          |
| NUMBER OF FAILED TESTING PIPELINES PER WORKLOAD        | count by (workload) (tekton_pipeline_run_status{type="Succeeded",status="False"})                                    |
| NUMBER OF SUCCESSFUL TESTING PIPELINES PER WORKLOAD    | count by (workload) (tekton_pipeline_run_status{type="Succeeded",status="True"})                                     |
| NUMBER OF SUCCESSFULL CONFIG WRITER TASKS PER WORKLOAD | count by (workload) (tekton_task_run_status{component="config-writer",type="Succeeded",status="True"})               |
| NUMBER OF FAILED CONFIG WRITER TASKS PER WORKLOAD      | count by (workload) (tekton_task_run_status{component="config-writer",type="Succeeded",status="False"})              |
| SUCCESSFUL GIT SOURCE FOR A WORKLOAD                   | flux_git_repository_status{type="Ready",owner_type="Workload",component="source",status="True"}                      |
| FAILED GIT SOURCE FOR A WORKLOAD                       | flux_git_repository_status{type="Ready",owner_type="Workload",component="source",status="False"}                     |
| SUCCESSFUL GIT SOURCE FOR A DELIVERABLE                | flux_git_repository_status{type="Ready",owner_type="Deliverable",component="source",status="True"}                   |
| FAILED GIT SOURCE FOR A DELIVERABLE                    | flux_git_repository_status{type="Ready",owner_type="Deliverable",component="source",status="False"}                  |
| WORKLOADS WITHOUT A MATCHING SUPPLY CHAIN              | cartographer_workload_status{type="SupplyChainReady",status="False"}                                                 |
| NUMBER OF WORKLOADS PER TYPE                           | count by (type) (cartographer_workload_info)                                                                         |
| NUMBER OF WORKLOADS USING GIT AS SOURCE                | count(cartographer_workload_info{source_image=""})                                                                   |
| NUMBER OF WORKLOADS USING IMAGE AS SOURCE              | count(cartographer_workload_info{source_git_url=""})                                                                 |
| NUMBER OF WORKLOADS WITHOUT LIVE UPDATE ENABLED        | count(cartographer_workload_info{live_update_enabled="false"} or cartographer_workload_info{live_update_enabled=""}) |
| NUMBER OF WORKLOADS WITH LIVE UPDATE ENABLED           | count(cartographer_workload_info{live_update_enabled="true"})                                                        |
| NUMBER OF WORKLOADS WITHOUT DEBUG ENABLED              | count(cartographer_workload_info{debug_enabled="false"} or cartographer_workload_info{debug_enabled=""})             |
| NUMBER OF WORKLOADS WITH DEBUG ENABLED                 | count(cartographer_workload_info{debug_enabled="true"})                                                              |
| NUMBER OF DELIVERABLES USING GIT AS SOURCE             | count(cartographer_deliverable_info{source_image=""})                                                                |
| NUMBER OF DELIVERABLES USING IMAGE AS SOURCE           | count(cartographer_deliverable_info{source_git_url=""})                                                              |
| WORKLOADS IN FAILED STATE                              | cartographer_workload_status{type="Ready",status="False"}                                                            |
| DELIVERABLES IN FAILED STATE                           | cartographer_deliverable_status{type="Ready",status="False"}                                                         |
| TAP PACKAGE REPOSITORIES IN FAILED STATE               | carvel_packagerepository_info{namespace="tap-install",type="ReconcileSucceeded",status="False"}                      |
| TAP PACKAGE REPOSITORIES IN GOOD STATE                 | carvel_packagerepository_info{namespace="tap-install",type="ReconcileSucceeded",status="True"}                       |
| TAP PACKAGE INSTALLS IN FAILED STATE                   | carvel_packageinstall_info{namespace="tap-install",type="ReconcileSucceeded",status="False"}                         |
| TAP PACKAGE INSTALLS IN GOOD STATE                     | carvel_packageinstall_info{namespace="tap-install",type="ReconcileSucceeded",status="True"}                          |
| TAP PACKAGES WITH CUSTOM OVERLAYS                      | carvel_packageinstall_info{ytt_overlay_secret!=""}                                                                   |

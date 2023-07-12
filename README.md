# TAP Kube State Metrics

This is a proof of concept for monitoring TAP in prometheus using the custom resource support in kube state metrics.

The resource types and corresponding prometheus metrics being monitored are:
* **workloads** - cartographer_workload_info , cartographer_workload_status
* **deliverables** - cartographer_deliverable_info , cartographer_deliverable_status
* **service bindings** - service_binding_info, service_binding_status
* **cluster instance classes** - stk_cluster_instance_class_composition_selector, stk_cluster_instance_class_status
* **class claims** - stk_class_claim_info , stk_class_claim_status
* **resource claims** - stk_resource_claim_info, stk_resource_claim_status
* **knative services** - knative_service_info, knative_service_status
* **knative revisions** - knative_revision_info, knative_revision_status
* **kapp controller package repositories** - carvel_packagerepository_info
* **kapp controller package installations** - carvel_packageinstall_info
* **kapp controller apps** - carvel_app_info, carvel_app_namespaces
* **api descriptors** - api_descriptor_info, api_descriptor_status
* **tekton pipeline runs** - tekton_pipeline_run_info, tekton_pipeline_run_status
* **tekton task runs** - tekton_task_run_info, tekton_task_run_status
* **accelerators** - accelerator_info, accelerator_imports_info, accelerator_status
* **fragments** - accelerator_fragment_info, accelerator_fragment_status
* **flux git repositories** - flux_git_repository_info, flux_git_repository_status
* **image scans** - scst_image_scan_info, scst_image_scan_status
* **source scans** - scst_source_scan_info, scst_source_scan_status
* **kpack images** - kpack_image_info, kpack_image_status
* **kpack builds** - kpack_build_info, kpack_build_involved_buildpacks, kpack_build_status


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
| Use Case                                               | PromQL Query Example                                                                                                                                                                                                                                    |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| NUMBER OF WORKLOADS USING EACH BUILDPACK               | count by (id) (topk(1000, count by (id, workload) (kpack_build_involved_buildpacks)))                                                                                                                                                                   |
| NUMBER OF SUCCESSFUL BUILD PER WORKLOAD                | count by (workload) (kpack_build_status{status="True", type="Succeeded"})                                                                                                                                                                               |
| NUMBER OF FAILED BUILDS PER WORKLOAD                   | count by (workload) (kpack_build_status{status="Failed", type="Succeeded"})                                                                                                                                                                             |
| NUMBER OF FAILED TESTING PIPELINES PER WORKLOAD        | count by (workload) (tekton_pipeline_run_status{type="Succeeded",status="False"})                                                                                                                                                                       |
| NUMBER OF SUCCESSFUL TESTING PIPELINES PER WORKLOAD    | count by (workload) (tekton_pipeline_run_status{type="Succeeded",status="True"})                                                                                                                                                                        |
| NUMBER OF SUCCESSFULL CONFIG WRITER TASKS PER WORKLOAD | count by (workload) (tekton_task_run_status{component="config-writer",type="Succeeded",status="True"})                                                                                                                                                  |
| NUMBER OF FAILED CONFIG WRITER TASKS PER WORKLOAD      | count by (workload) (tekton_task_run_status{component="config-writer",type="Succeeded",status="False"})                                                                                                                                                 |
| SUCCESSFUL GIT SOURCE FOR A WORKLOAD                   | flux_git_repository_status{type="Ready",owner_type="Workload",component="source",status="True"}                                                                                                                                                         |
| FAILED GIT SOURCE FOR A WORKLOAD                       | flux_git_repository_status{type="Ready",owner_type="Workload",component="source",status="False"}                                                                                                                                                        |
| SUCCESSFUL GIT SOURCE FOR A DELIVERABLE                | flux_git_repository_status{type="Ready",owner_type="Deliverable",component="source",status="True"}                                                                                                                                                      |
| FAILED GIT SOURCE FOR A DELIVERABLE                    | flux_git_repository_status{type="Ready",owner_type="Deliverable",component="source",status="False"}                                                                                                                                                     |
| WORKLOADS WITHOUT A MATCHING SUPPLY CHAIN              | cartographer_workload_status{type="SupplyChainReady",status="False"}                                                                                                                                                                                    |
| NUMBER OF WORKLOADS PER TYPE                           | count by (type) (cartographer_workload_info)                                                                                                                                                                                                            |
| NUMBER OF WORKLOADS USING GIT AS SOURCE                | count(cartographer_workload_info{source_image=""})                                                                                                                                                                                                      |
| NUMBER OF WORKLOADS USING IMAGE AS SOURCE              | count(cartographer_workload_info{source_git_url=""})                                                                                                                                                                                                    |
| NUMBER OF WORKLOADS WITHOUT LIVE UPDATE ENABLED        | count(cartographer_workload_info{live_update_enabled="false"} or cartographer_workload_info{live_update_enabled=""})                                                                                                                                    |
| NUMBER OF WORKLOADS WITH LIVE UPDATE ENABLED           | count(cartographer_workload_info{live_update_enabled="true"})                                                                                                                                                                                           |
| NUMBER OF WORKLOADS WITHOUT DEBUG ENABLED              | count(cartographer_workload_info{debug_enabled="false"} or cartographer_workload_info{debug_enabled=""})                                                                                                                                                |
| NUMBER OF WORKLOADS WITH DEBUG ENABLED                 | count(cartographer_workload_info{debug_enabled="true"})                                                                                                                                                                                                 |
| NUMBER OF DELIVERABLES USING GIT AS SOURCE             | count(cartographer_deliverable_info{source_image=""})                                                                                                                                                                                                   |
| NUMBER OF DELIVERABLES USING IMAGE AS SOURCE           | count(cartographer_deliverable_info{source_git_url=""})                                                                                                                                                                                                 |
| WORKLOADS IN FAILED STATE                              | cartographer_workload_status{type="Ready",status="False"}                                                                                                                                                                                               |
| DELIVERABLES IN FAILED STATE                           | cartographer_deliverable_status{type="Ready",status="False"}                                                                                                                                                                                            |
| TAP PACKAGE REPOSITORIES IN FAILED STATE               | carvel_packagerepository_info{namespace="tap-install",type="ReconcileSucceeded",status="False"}                                                                                                                                                         |
| TAP PACKAGE REPOSITORIES IN GOOD STATE                 | carvel_packagerepository_info{namespace="tap-install",type="ReconcileSucceeded",status="True"}                                                                                                                                                          |
| TAP PACKAGE INSTALLS IN FAILED STATE                   | carvel_packageinstall_info{namespace="tap-install",type="ReconcileSucceeded",status="False"}                                                                                                                                                            |
| TAP PACKAGE INSTALLS IN GOOD STATE                     | carvel_packageinstall_info{namespace="tap-install",type="ReconcileSucceeded",status="True"}                                                                                                                                                             |
| TAP PACKAGES WITH CUSTOM OVERLAYS                      | carvel_packageinstall_info{ytt_overlay_secret!=""}                                                                                                                                                                                                      |
| TAP ENABLED NAMESPACES                                 | carvel_app_namespaces{namespace="tap-namespace-provisioning",name="provisioner"}                                                                                                                                                                        |
| NUMBER OF WORKLOADS PER SUPPLY CHAIN                   | count by (supply_chain) (cartographer_workload_info)                                                                                                                                                                                                    |
| NUMBER OF CLASS CLAIMS PER CLASS                       | count by (class) (stk_class_claim_info)                                                                                                                                                                                                                 |
| NUMBER OF CLASS CLAIM IN READY STATE                   | count(stk_class_claim_status{type="Ready",status="True"})                                                                                                                                                                                               |
| NUMBER OF CLASS CLAIMS IN FAILED STATE                 | count(stk_class_claim_status{type="Ready",status="False"})                                                                                                                                                                                              |
| NUMBER OF RESOURCE CLAIMS NOT MANAGED BY A CLASS CLAIM | count(stk_resource_claim_info{class_claim=""})                                                                                                                                                                                                          |
| NUMBER OF RECONCILED API DESCRIPTORS                   | count(api_descriptor_status{type="Ready",status="True"})                                                                                                                                                                                                |
| NUMBER OF API DESCRIPTORS IN NOT READY STATE           | count(api_descriptor_status{type="Ready",status="False"})                                                                                                                                                                                               |
| NUMBER OF ACCELERATORS                                 | count(accelerator_info)                                                                                                                                                                                                                                 |
| NUMBER OF FRAGMENTS                                    | count(accelerator_fragment_info)                                                                                                                                                                                                                        |
| NUMBER OF READY FRAGMENTS                              | count(accelerator_fragment_info{ready="true"})                                                                                                                                                                                                          |
| NUMBER OF READY ACCELERATORS                           | count(accelerator_info{ready="true"})                                                                                                                                                                                                                   |
| NOT READY ACCELERATORS                                 | accelerator_info{ready!="true"}                                                                                                                                                                                                                         |
| NOT READY FRAGMENTS                                    | accelerator_fragment_info{ready!="true"}                                                                                                                                                                                                                |
| KNATIVE SERVICES IN FAILED STATE                       | knative_service_status{status="False",type="Ready"}                                                                                                                                                                                                     |
| KNATIVE SERVICES WHERE LATEST REVISION IS NOT READY    | knative_service_info unless ignoring(__dst__) (label_replace(knative_service_info, "__dst__", "$1", "latest_created_revision", "(.\*)") == label_replace(knative_service_info, "__dst__", "$1", "latest_ready_revision", "(.\*)"))                      |
| UNHEALTHY KNATIVE REVISIONS                            | knative_revision_status{status="False",type="Ready"}                                                                                                                                                                                                    |
| WORKLOADS WHERE LATEST REVISION IS NOT HEALTHY         | count by (workload) (knative_service_info unless ignoring(__dst__) (label_replace(knative_service_info, "__dst__", "$1", "latest_created_revision", "(.\*)") == label_replace(knative_service_info, "__dst__", "$1", "latest_ready_revision", "(.\*)")) |
| SERVICE BINDING IN NOT READY STATE                     | service_binding_status{type="Ready",status!="True"}                                                                                                                                                                                                     |
| NUMBER OF SERVICE BINDING PER TYPE OF SERVICE CLAIM    | count by (service_kind) (service_binding_info)                                                                                                                                                                                                          |
| CLUSTER INSTANCE CLASSES IN NOT READY STATE            | stk_cluster_instance_class_status{type="Ready",status!="True"}                                                                                                                                                                                          |

---
title: 'Update Labels and Annotations'
weight: 30
---

As you have seen in the previous chapter Cluster generator work on Cluster labels. In this chapter you will update labels and annotations for the hub-cluster. These will be used by ApplicationSet to generate Applications.

![eks-blueprint-blue](/static/images/argocd-update-metadata.png)

In the Argo CD user interface, go to the hub cluster. The hub-cluster currently has some existing Labels and Annotations defined. These are added by GitOps Bridge.

![Hub Cluster Metadata](/static/images/hubcluster-initial-metadata.png)

> Labels can be used to find collections of objects that satisfy generator conditions. Annotations provide additional information.


### 1. Codecommit Remote State
The hub cluster references codecommit module outputs. 

```json
cat <<'EOF' >> ~/environment/hub/remote_state.tf 

data "terraform_remote_state" "git" {
  backend = "local"

  config = {
    path = "${path.module}/../codecommit/terraform.tfstate"
  }
}

EOF
```

### 2. Reference Codecommit outputs values

```json
cat <<'EOF' >> ~/environment/hub/main.tf 
locals{

  gitops_addons_url      = data.terraform_remote_state.git.outputs.gitops_addons_url
  gitops_addons_basepath = data.terraform_remote_state.git.outputs.gitops_addons_basepath
  gitops_addons_path     = data.terraform_remote_state.git.outputs.gitops_addons_path
  gitops_addons_revision = data.terraform_remote_state.git.outputs.gitops_addons_revision

  gitops_platform_url      = data.terraform_remote_state.git.outputs.gitops_platform_url
  gitops_platform_basepath = data.terraform_remote_state.git.outputs.gitops_platform_basepath
  gitops_platform_path     = data.terraform_remote_state.git.outputs.gitops_platform_path
  gitops_platform_revision = data.terraform_remote_state.git.outputs.gitops_platform_revision

  gitops_workload_url      = data.terraform_remote_state.git.outputs.gitops_workload_url
  gitops_workload_basepath = data.terraform_remote_state.git.outputs.gitops_workload_basepath
  gitops_workload_path     = data.terraform_remote_state.git.outputs.gitops_workload_path
  gitops_workload_revision = data.terraform_remote_state.git.outputs.gitops_workload_revision
  
}
EOF
```




### 2. Define addons variables

Define  enable-* addons boolean variables. These provide a simple way to control whether addons are installed or removed. Define addons variable as a list of key/value pairs of addon(enable-*) values. Define addons_metadata variable as a list of key/value pairs of mainly codecommit values.


Some values are commented and will be used later in the workshop.

:::code{showCopyAction=true showLineNumbers=false language=json highlightLines='48,58'}
cat <<'EOF' >> ~/environment/hub/main.tf

locals{
  aws_addons = {
    enable_aws_argocd                            = try(var.addons.enable_aws_argocd, false)    
    enable_cert_manager                          = try(var.addons.enable_cert_manager, false)
    enable_aws_efs_csi_driver                    = try(var.addons.enable_aws_efs_csi_driver, false)
    enable_aws_fsx_csi_driver                    = try(var.addons.enable_aws_fsx_csi_driver, false)
    enable_aws_cloudwatch_metrics                = try(var.addons.enable_aws_cloudwatch_metrics, false)
    enable_aws_privateca_issuer                  = try(var.addons.enable_aws_privateca_issuer, false)
    enable_cluster_autoscaler                    = try(var.addons.enable_cluster_autoscaler, false)
    enable_external_dns                          = try(var.addons.enable_external_dns, false)
    enable_external_secrets                      = try(var.addons.enable_external_secrets, false)
    enable_aws_load_balancer_controller          = try(var.addons.enable_aws_load_balancer_controller, false)
    enable_fargate_fluentbit                     = try(var.addons.enable_fargate_fluentbit, false)
    enable_aws_for_fluentbit                     = try(var.addons.enable_aws_for_fluentbit, false)
    enable_aws_node_termination_handler          = try(var.addons.enable_aws_node_termination_handler, false)
    enable_karpenter                             = try(var.addons.enable_karpenter, false)
    enable_velero                                = try(var.addons.enable_velero, false)
    enable_aws_gateway_api_controller            = try(var.addons.enable_aws_gateway_api_controller, false)
    enable_aws_ebs_csi_resources                 = try(var.addons.enable_aws_ebs_csi_resources, false)
    enable_aws_secrets_store_csi_driver_provider = try(var.addons.enable_aws_secrets_store_csi_driver_provider, false)
    enable_ack_apigatewayv2                      = try(var.addons.enable_ack_apigatewayv2, false)
    enable_ack_dynamodb                          = try(var.addons.enable_ack_dynamodb, false)
    enable_ack_s3                                = try(var.addons.enable_ack_s3, false)
    enable_ack_rds                               = try(var.addons.enable_ack_rds, false)
    enable_ack_prometheusservice                 = try(var.addons.enable_ack_prometheusservice, false)
    enable_ack_emrcontainers                     = try(var.addons.enable_ack_emrcontainers, false)
    enable_ack_sfn                               = try(var.addons.enable_ack_sfn, false)
    enable_ack_eventbridge                       = try(var.addons.enable_ack_eventbridge, false)
  }
  oss_addons = {
    enable_argocd                          = try(var.addons.enable_argocd, false)
    enable_argo_rollouts                   = try(var.addons.enable_argo_rollouts, false)
    enable_argo_events                     = try(var.addons.enable_argo_events, false)
    enable_argo_workflows                  = try(var.addons.enable_argo_workflows, false)
    enable_cluster_proportional_autoscaler = try(var.addons.enable_cluster_proportional_autoscaler, false)
    enable_gatekeeper                      = try(var.addons.enable_gatekeeper, false)
    enable_gpu_operator                    = try(var.addons.enable_gpu_operator, false)
    enable_ingress_nginx                   = try(var.addons.enable_ingress_nginx, false)
    enable_kyverno                         = try(var.addons.enable_kyverno, false)
    enable_kube_prometheus_stack           = try(var.addons.enable_kube_prometheus_stack, false)
    enable_metrics_server                  = try(var.addons.enable_metrics_server, false)
    enable_prometheus_adapter              = try(var.addons.enable_prometheus_adapter, false)
    enable_secrets_store_csi_driver        = try(var.addons.enable_secrets_store_csi_driver, false)
    enable_vpa                             = try(var.addons.enable_vpa, false)
  }
  addons = merge(
    local.aws_addons,
    local.oss_addons,
    { kubernetes_version = local.cluster_version },
    { aws_cluster_name = module.eks.cluster_name },
    { workloads = true }
    #enablewebstore,{ workload_webstore = true }      
  )


  addons_metadata = merge(
    #enableaddonmetadata module.eks_blueprints_addons.gitops_metadata,
    {
      aws_cluster_name = module.eks.cluster_name
      aws_region       = local.region
      aws_account_id   = data.aws_caller_identity.current.account_id
      aws_vpc_id       = local.vpc_id
    },
    {
      #enableirsarole argocd_iam_role_arn = aws_iam_role.argocd_hub.arn
      argocd_namespace    = local.argocd_namespace
    },
    {
       addons_repo_url      = local.gitops_addons_url
       addons_repo_basepath = local.gitops_addons_basepath
       addons_repo_path     = local.gitops_addons_path
       addons_repo_revision = local.gitops_addons_revision
    },
    {
       platform_repo_url      = local.gitops_platform_url
       platform_repo_basepath = local.gitops_platform_basepath
       platform_repo_path     = local.gitops_platform_path
       platform_repo_revision = local.gitops_platform_revision
    },
    {
       workload_repo_url      = local.gitops_workload_url
       workload_repo_basepath = local.gitops_workload_basepath
       workload_repo_path     = local.gitops_workload_path
       workload_repo_revision = local.gitops_workload_revision
    }

  )
}

EOF
:::

### 4. Update Labels and Annotations

We need to update the labels and annotations on the hub-cluster Cluster object. To do this, we will use the GitOps Bridge. The GitOps Bridge is configured to update labels and annotations on the specified cluster object.

```bash
sed -i "s/#enablemetadata//g" ~/environment/hub/main.tf
```

The code provided above uncomments metdata and addons variables as highlighted below in `main.tf`. The values defined in the addons variable are assigned to Labels, while the metadata values are assigned to Annotations on the cluster object.

:::code{language=yml showCopyAction=false showLineNumbers=false highlightLines='7-8'}
module "gitops_bridge_bootstrap" {
  source  = "gitops-bridge-dev/gitops-bridge/helm"
  version = "0.0.1"
  cluster = {
    cluster_name = module.eks.cluster_name
    environment  = local.environment
     metadata     = local.addons_metadata
     addons       = local.addons
  }
:::


### 5. Terraform apply

```bash
cd ~/environment/hub
terraform apply --auto-approve
```
### 6. Validate update to labels and addons


Goto to the **Settings > Clusters > hub-cluster**  in the Argo CD dashboard. Examine the Hub-Cluster Cluster object. This will confirm that GitOps Bridge has successfully updated the Labels and Annotations.

![Hub Cluster Updated Metadata](/static/images/hubcluster-update-metadata.png)


ArgoCD pulls lables and annotations for the cluster object from a kubernetes secret. We used gitops bridge to update labels and annotations for the secret. 

You can check  the Labels and annotations on the cluster secret: 

```bash
kubectl --context hub get secrets -n argocd hub-cluster -o yaml
```

:::expand{header="Example of output"}
```
apiVersion: v1
data:
  config: ewogICJ0bHNDbGllbnRDb25maWciOiB7AiaW5zZWN1cmUiOiBmYWxzZQogIH0KfQo=
  name: aHViLWNsdXN0ZXI=
  server: aHR0cHM6Ly9rdWJlcm5VzLmRlZmF1bHQuc3Zj
kind: Secret
metadata:
  annotations:
    addons_repo_basepath: assets/platform/addons/
    addons_repo_path: applicationset/
    addons_repo_revision: HEAD
    addons_repo_url: https://github.com/aws-samples/eks-blueprints-for-terraform-workshop.git
    argocd_namespace: argocd
    aws_account_id: "382076407153"
    aws_cluster_name: hub-cluster
    aws_load_balancer_controller_iam_role_arn: arn:aws:iam::12345678910:role/alb-controller-20240604085058813100000015
    aws_load_balancer_controller_namespace: kube-system
    aws_load_balancer_controller_service_account: aws-load-balancer-controller-sa
    aws_region: us-east-2
    aws_vpc_id: vpc-09924bd9e1637d9a1
    cluster_name: hub-cluster
    environment: hub
    platform_repo_basepath: assets/platform/
    platform_repo_path: bootstrap
    platform_repo_revision: HEAD
    platform_repo_url: https://github.com/aws-samples/eks-blueprints-for-terraform-workshop.git
    workload_repo_basepath: assets/developer/
    workload_repo_path: gitops/apps
    workload_repo_revision: HEAD
    workload_repo_url: https://github.com/aws-samples/eks-blueprints-for-terraform-workshop.git
  creationTimestamp: "2024-06-04T08:52:40Z"
  labels:
    argocd.argoproj.io/secret-type: cluster
    aws_cluster_name: hub-cluster
    cluster_name: hub-cluster
    enable_ack_apigatewayv2: "false"
    enable_ack_dynamodb: "false"
    enable_ack_emrcontainers: "false"
    enable_ack_eventbridge: "false"
    enable_ack_prometheusservice: "false"
    enable_ack_rds: "false"
    enable_ack_s3: "false"
    enable_ack_sfn: "false"
    enable_argo_events: "false"
    enable_argo_rollouts: "false"
    enable_argo_workflows: "false"
    enable_argocd: "true"
    enable_aws_cloudwatch_metrics: "false"
    enable_aws_ebs_csi_resources: "false"
    enable_aws_efs_csi_driver: "false"
    enable_aws_for_fluentbit: "false"
    enable_aws_fsx_csi_driver: "false"
    enable_aws_gateway_api_controller: "false"
    enable_aws_load_balancer_controller: "true"
    enable_aws_node_termination_handler: "false"
    enable_aws_privateca_issuer: "false"
    enable_aws_secrets_store_csi_driver_provider: "false"
    enable_cert_manager: "false"
    enable_cluster_autoscaler: "false"
    enable_cluster_proportional_autoscaler: "false"
    enable_external_dns: "false"
    enable_external_secrets: "false"
    enable_fargate_fluentbit: "false"
    enable_gatekeeper: "false"
    enable_gpu_operator: "false"
    enable_ingress_nginx: "false"
    enable_karpenter: "false"
    enable_kube_prometheus_stack: "false"
    enable_kyverno: "false"
    enable_metrics_server: "false"
    enable_prometheus_adapter: "false"
    enable_secrets_store_csi_driver: "false"
    enable_velero: "false"
    enable_vpa: "false"
    environment: hub
    kubernetes_version: "1.28"
    workload_webstore: "false"
    workloads: "false"
  name: hub-cluster
  namespace: argocd
  resourceVersion: "309742"
  uid: 1156e385-97af-4732-83ae-55aafeb9ec62
type: Opaque
```
:::
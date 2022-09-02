# Kubeflow 清单

## 目录

<!-- toc -->

- [概述](#overview)
- [Kubeflow 组件版本 versions](#kubeflow-components-versions)
- [安装](#installation)
    * [先决条件](#prerequisites)
    * [使用单个命令安装](#install-with-a-single-command)
    * [安装单个组件](#install-individual-components)
    * [连接 Kubeflow 集群](#connect-to-your-kubeflow-cluster)
    * [更改默认用户密码](#change-default-user-password)
- [常见问题](#frequently-asked-questions)

<!-- tocstop -->

## 概述

这个 repo 归 [Manifests Working Group](https://github.com/kubeflow/community/blob/master/wg-manifests/charter.md)所有。
如果您是包的创作或编辑贡献者，请参阅[最佳实践](./docs/KustomizeBestPractices.md).

Kubeflow Manifests 存储库组织在三 (3) 个主目录下，其中包括用于安装的清单：

| 目录 | 目的 |
| - | - |
| `apps` | Kubeflow 的官方组件，由各自的 Kubeflow WG 维护 |
| `common` | 由 Manifests WG 维护的公共服务 |
| `contrib` | 第 3 方贡献的应用程序，由外部维护，不属于 Kubeflow WG |

`distributions` 目录包含 Kubeflow 特定的、配置好的发行版清单，并将在 1.4 版本期间逐步淘汰， [因为未来的发行版将在各自的外部存储库中维护它们的清单](https://github.com/kubeflow/community/blob/master/proposals/kubeflow-distributions.md)。

`docs`、`hack` 及 `tests` 也将逐步淘汰。

从 Kubeflow 1.3 开始，所有组件都应该只能使用 `kustomize` 来部署。任何用于在清单之上部署的自动化工具都应由分发所有者在外部维护。

## Kubeflow 组件版本

这个 repo 会定期同步来自各自上游 repo 的所有官方 Kubeflow 组件。以下矩阵显示了我们为每个组件包含的 git 版本：

| 组件 | 本地清单路径 | 上游版本 |
| - | - | - |
| 训练控制器 | apps/training-operator/upstream | [v1.5.0-rc.0](https://github.com/kubeflow/training-operator/tree/v1.5.0-rc.0/manifests) |
| Notebook 控制器 | apps/jupyter/notebook-controller/upstream | [v1.6.0-rc.0](https://github.com/kubeflow/kubeflow/tree/v1.6.0-rc.0/components/notebook-controller/config) |
| Tensorboard 控制器 | apps/tensorboard/tensorboard-controller/upstream | [v1.6.0-rc.0](https://github.com/kubeflow/kubeflow/tree/v1.6.0-rc.0/components/tensorboard-controller/config) |
| 看板中心 | apps/centraldashboard/upstream | [v1.6.0-rc.0](https://github.com/kubeflow/kubeflow/tree/v1.6.0-rc.0/components/centraldashboard/manifests) |
| Profiles + KFAM | apps/profiles/upstream | [v1.6.0-rc.0](https://github.com/kubeflow/kubeflow/tree/v1.6.0-rc.0/components/profile-controller/config) |
| PodDefaults Webhook | apps/admission-webhook/upstream | [v1.6.0-rc.0](https://github.com/kubeflow/kubeflow/tree/v1.6.0-rc.0/components/admission-webhook/manifests) |
| Jupyter Web 应用 | apps/jupyter/jupyter-web-app/upstream | [v1.6.0-rc.0](https://github.com/kubeflow/kubeflow/tree/v1.6.0-rc.0/components/crud-web-apps/jupyter/manifests) |
| Tensorboards Web 应用 | apps/tensorboard/tensorboards-web-app/upstream | [v1.6.0-rc.0](https://github.com/kubeflow/kubeflow/tree/v1.6.0-rc.0/components/crud-web-apps/tensorboards/manifests) |
| Volumes Web 应用 | apps/volumes-web-app/upstream | [v1.6.0-rc.0](https://github.com/kubeflow/kubeflow/tree/v1.6.0-rc.0/components/crud-web-apps/volumes/manifests) |
| Katib | apps/katib/upstream | [v0.14.0-rc.0](https://github.com/kubeflow/katib/tree/v0.14.0-rc.0/manifests/v1beta1) |
| KServe | contrib/kserve/kserve | [release-0.8](https://github.com/kserve/kserve/tree/8079f375cbcedc4d45a1b4aade2e2308ea6f9ae8/install/v0.8.0) |
| KServe Models Web 应用 | contrib/kserve/models-web-app | [v0.8.0](https://github.com/kserve/models-web-app/tree/v0.8.0/config) |
| Kubeflow Pipelines | apps/pipeline/upstream | [2.0.0-alpha.3](https://github.com/kubeflow/pipelines/tree/2.0.0-alpha.3/manifests/kustomize) |
| Kubeflow Tekton Pipelines | apps/kfp-tekton/upstream | [v1.2.1](https://github.com/kubeflow/kfp-tekton/tree/v1.2.1/manifests/kustomize) |

以下也是一个矩阵，其中包含来自 Kubeflow 不同项目的
常用组件的版本：

| Component | 本地清单路径 | 上游版本 |
| - | - | - |
| Istio | common/istio-1-14 | [1.14.1](https://github.com/istio/istio/releases/tag/1.14.1) |
| Knative | common/knative | [1.2.5](https://github.com/knative/serving/releases/tag/knative-v1.2.5) |

## 安装

从 Kubeflow 1.3 开始，Manifests WG 提供了两种使用 kustomize 安装 Kubeflow 官方组件和常用服务选项。目的是帮助最终用户轻松安装，并帮助发行版所有者构建他们配置好的的发行版用于端点测试：

1. 单命令安装 `apps` 和 `common` 下所有组件
2. 多命令安装 `apps` 和 `common` 下单个组件

选项 1 旨在简化最终用户的部署。 \
选项 2 的目标是支持定制和挑选单个组件的能力。

`example` 录包含一个示例 kustomization，以便能够运行单个命令。

:warning: 在这两个选项中，我们使用默认电子邮件 ( `user@example.com`) 和密码 ( `12341234`)。对于任何生产 Kubeflow 部署，您应该按照[相关部分](#change-default-user-password)更改默认密码。

### 先决条件

- `Kubernetes` (升级到 `1.22`) 带有默认的 [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- `kustomize` (版本 `3.2.0`) ([下载地址](https://github.com/kubernetes-sigs/kustomize/releases/tag/v3.2.0))
    - :warning: Kubeflow 1.5.0 不兼容于 kustomize 4.x 最新版本。这是由于资源排序和打印发生了变化。请查看 [kubernetes-sigs/kustomize#3794](https://github.com/kubernetes-sigs/kustomize/issues/3794) 及 [kubeflow/manifests#1797](https://github.com/kubeflow/manifests/issues/1797)。我们知道这并不理想，并且正在与上游 kustomize 团队合作，尽快添加对最新版本 kustomize 的支持。
- `kubectl`

---
**NOTE**

`kubectl apply` 命令再初次尝试时可能会失败。这是 Kubernetes 和 `kubectl` 工作方式固有的（例如，必须在 CRD 准备好之后创建 CR）。解决方案是简单地重新运行命令，直到它成功。对于单行命令，我们包含了一个 bash 单行来重试该命令。

---

### 使用单个命令安装

您可以使用以下命令安装所有 Kubeflow 官方组件（位于 `apps` 下）和所有常用服务（位于 `common` 下）：

```sh
while ! kustomize build example | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
```

一切都安装成功后，您可以通过[登录集群来访问](#connect-to-your-kubeflow-cluster) Kubeflow Central Dashboard 。

恭喜！您现在可以开始使用 Kubeflow 试验和运行您的端到端 ML 工作流。

### 安装单个组件

在本节中，我们将分别安装每个 Kubeflow 官方组件（在 `apps` 下）和每个公共服务（在 `common` 下），仅使用 `kubectl` 和 `kustomize`。

如果以下所有命令都执行，结果与上面单条命令安装部分的结果相同。本节的目的是：

- 提供每个组件的描述并了解它是如何安装的。
- 使用户或分发所有者能够只挑选他们需要的组件。

#### 证书管理器

许多 Kubeflow 组件使用 cert-manager 为
准入 webhook 提供证书。

安装 cert-manager：

```sh
kustomize build common/cert-manager/cert-manager/base | kubectl apply -f -
kustomize build common/cert-manager/kubeflow-issuer/base | kubectl apply -f -
```

#### Istio

许多 Kubeflow 组件使用 Istio 来保护其流量、强制执行
网络授权和实施路由策略。

安装 Istio:

```sh
kustomize build common/istio-1-14/istio-crds/base | kubectl apply -f -
kustomize build common/istio-1-14/istio-namespace/base | kubectl apply -f -
kustomize build common/istio-1-14/istio-install/base | kubectl apply -f -
```

#### Dex

Dex 是个具有多个后端身份验证的 OpenID Connect Identity (OIDC) 服务。在这个默认安装中，它包括一个带有 email 的静态用户 `user@example.com`。默认情况下，用户的密码是 `12341234`。对于任何生产 Kubeflow 部署，您应该按照[相关部分](#change-default-user-password)更改默认密码。

安装 Dex:

```sh
kustomize build common/dex/overlays/istio | kubectl apply -f -
```

#### OIDC 身份验证服务

OIDC AuthService 扩展了您的 Istio Ingress-Gateway 功能，能够充当 OIDC 客户端：

```sh
kustomize build common/oidc-authservice/base | kubectl apply -f -
```

#### Knative

Knative 由 Kubeflow 官方组件 KFServing 使用。

安装 Knative Serving：

```sh
kustomize build common/knative/knative-serving/overlays/gateways | kubectl apply -f -
kustomize build common/istio-1-14/cluster-local-gateway/base | kubectl apply -f -
```

或者，您可以安装可用于推理请求日志记录的 Knative Eventing：

```sh
kustomize build common/knative/knative-eventing/base | kubectl apply -f -
```

#### Kubeflow 命名空间

创建 Kubeflow 组件所在的命名空间。这个空间
叫做 `kubeflow`。

安装 kubeflow 命名空间：

```sh
kustomize build common/kubeflow-namespace/base | kubectl apply -f -
```

#### Kubeflow 角色

创建 Kubeflow ClusterRoles, `kubeflow-view`、`kubeflow-edit` 及
`kubeflow-admin`。Kubeflow 组件聚合
这些 ClusterRoles 的权限。

安装 kubeflow 角色：

```sh
kustomize build common/kubeflow-roles/base | kubectl apply -f -
```

#### Kubeflow Istio 资源

创建 Kubeflow 所需的 Istio 资源。 现在定制化的
Istio Gateway 命名 `kubeflow-gateway`，在命名空间 `kubeflow`。
果您想使用自己的 Istio 进行安装，那么您也需要
此定制。

安装 istio 资源:

```sh
kustomize build common/istio-1-14/kubeflow-istio-resources/base | kubectl apply -f -
```

#### Kubeflow Pipelines

安装 [多用户 Kubeflow Pipelines](https://www.kubeflow.org/docs/components/pipelines/multi-user/) 官方 Kubeflow 组件：

```sh
kustomize build apps/pipeline/upstream/env/cert-manager/platform-agnostic-multi-user | kubectl apply -f -
```

如果您的容器运行时不是 docker，请改用 pns 执行器：

```sh
kustomize build apps/pipeline/upstream/env/platform-agnostic-multi-user-pns | kubectl apply -f -
```

有关它们的优缺点，请参阅[argo 工作流执行器文档](https://argoproj.github.io/argo-workflows/workflow-executors/#process-namespace-sharing-pns)。

**多用户 Kubeflow Pipelines 依赖**

* Istio + Kubeflow Istio 资源
* Kubeflow Roles
* OIDC 身份验证服务（或云提供商特定的身份验证服务）
* Profiles + KFAM

**替代方案：Kubeflow Pipelines Standalone**

您可以安装 [独立 Kubeflow Pipelines](https://www.kubeflow.org/docs/components/pipelines/installation/standalone-deployment/)，它

* 不支持多用户分离
* 不依赖此处提到的其他服务

您可以在 [Kubeflow Pipelines 的安装选项中](https://www.kubeflow.org/docs/components/pipelines/installation/overview/)
了解更多关于它们的差异。

除了 Kubeflow Pipelines Standalone 文档中的安装说明外，您还需要应用两个虚拟服务来在 kubeflow-gateway 中公开 [Kubeflow Pipelines UI](https://github.com/kubeflow/pipelines/blob/1.7.0/manifests/kustomize/base/installs/multi-user/virtual-service.yaml) 和 [Metadata API](https://github.com/kubeflow/pipelines/blob/1.7.0/manifests/kustomize/base/metadata/options/istio/virtual-service.yaml)。

#### KServe / KFServing

KFServing 已更名为 KServe。

安装 KServe 组件：

```sh
kustomize build contrib/kserve/kserve | kubectl apply -f -
```

安装 Models web app：

```sh
kustomize build contrib/kserve/models-web-app/overlays/kubeflow | kubectl apply -f -
```

- ../contrib/kserve/models-web-app/overlays/kubeflow

#### Katib

安装 Kubeflow 官方组件 Katib：

```sh
kustomize build apps/katib/upstream/installs/katib-with-kubeflow | kubectl apply -f -
```

#### 看板中心

安装官方 Kubeflow 组件 Central Dashboard：

```sh
kustomize build apps/centraldashboard/upstream/overlays/kserve | kubectl apply -f -
```

#### Admission Webhook

为 PodDefaults 安装 Admission Webhook：

```sh
kustomize build apps/admission-webhook/upstream/overlays/cert-manager | kubectl apply -f -
```

#### Notebooks

安装官方 Kubeflow 组件 Notebook Controller：

```sh
kustomize build apps/jupyter/notebook-controller/upstream/overlays/kubeflow | kubectl apply -f -
```

安装官方 Kubeflow 组件 Jupyter Web App：

```sh
kustomize build apps/jupyter/jupyter-web-app/upstream/overlays/istio | kubectl apply -f -
```

#### Profiles + KFAM

安装官方 Kubeflow 组件 Profile Controller 及 Kubeflow Access-Management (KFAM)：

```sh
kustomize build apps/profiles/upstream/overlays/kubeflow | kubectl apply -f -
```

#### Volumes Web App

安装官方 Kubeflow 组件 Volumes Web 应用：

```sh
kustomize build apps/volumes-web-app/upstream/overlays/istio | kubectl apply -f -
```

#### Tensorboard

安装官方 Kubeflow 组件 Tensorboards Web 应用：

```sh
kustomize build apps/tensorboard/tensorboards-web-app/upstream/overlays/istio | kubectl apply -f -
```

安装官方 Kubeflow 组件 Tensorboard 控制器：

```sh
kustomize build apps/tensorboard/tensorboard-controller/upstream/overlays/kubeflow | kubectl apply -f -
```

#### Training Operator

安装官方 Kubeflow 组件 Training Operator：

```sh
kustomize build apps/training-operator/upstream/overlays/kubeflow | kubectl apply -f -
```

#### User 命名空间

最后，为默认用户创建一个新的命名空间（名为 `kubeflow-user-example-com`）。

```sh
kustomize build common/user-namespace/base | kubectl apply -f -
```

### 连接到 Kubeflow 集群

安装后，所有 Pod 都需要一些时间才能准备就绪。在尝试连接之前，请确保所有 Pod 都已准备就绪，否则您可能会遇到意外错误。要检查所有与 Kubeflow 相关的 Pod 是否已准备就绪，请使用以下命令：

```sh
kubectl get pods -n cert-manager
kubectl get pods -n istio-system
kubectl get pods -n auth
kubectl get pods -n knative-eventing
kubectl get pods -n knative-serving
kubectl get pods -n kubeflow
kubectl get pods -n kubeflow-user-example-com
```

#### 端口转发

访问 Kubeflow 的默认方式是通过端口转发。这使您能够快速开始，而不会对您的环境提出任何要求。运行以下命令将 Istio 的 Ingress-Gateway 端口转发到本地端口 `8080`：

```sh
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```

运行命令后，您可以通过执行以下操作访问 Kubeflow Central Dashboard：

1. 打开浏览器并访问 `http://localhost:8080`。您应该会看到 Dex 登录屏幕。
2. 使用默认用户的凭据登录。默认电子邮件地址是 `user@example.com`，默认密码是 `12341234`。

#### NodePort / LoadBalancer / Ingress

为了使用 NodePort / LoadBalancer / Ingress 连接到 Kubeflow，您需要设置 HTTPS。原因是我们的许多 Web 应用程序（例如，Tensorboard Web 应用程序、Jupyter Web 应用程序、Katib UI）都使用 [Secure Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#restrict_access_to_cookies)，因此通过非 localhost 域使用 HTTP 访问 Kubeflow 不起作用。

使用适当的 HTTPS 公开您的 Kubeflow 集群是一个严重依赖于您的环境的过程。因此，请查看针对特定环境的可用 [Kubeflow 发行版](https://www.kubeflow.org/docs/started/installing-kubeflow/#install-a-packaged-kubeflow-distribution)，然后选择适合您需求的发行版。

---
**注意**

如果您确定需要通过 HTTP 公开 Kubeflow，您可以通过在每个相关的 Web 应用程序中设置环境变量 `APP_SECURE_COOKIES` 为 `false` 来禁用 `Secure Cookies` 功能。这么做是不建议的，因为它会带来安全风险。

---

### 更改默认用户密码

出于安全原因，我们不想在对安全敏感的环境中安装时使用默认 Kubeflow 用户的默认密码。相反，您应该在部署之前定义自己的密码。为默认用户定义密码：

1. 使用 email 为默认用户 `user@example.com` 选择一个密码，并使用以下方法 `bcrypt` 对其进行哈希处理：

    ```sh
    python3 -c 'from passlib.hash import bcrypt; import getpass; print(bcrypt.using(rounds=12, ident="2y").hash(getpass.getpass()))'
    ```

2. 编辑 `common/dex/base/config-map.yaml` 并填充相关字段：

    ```yaml
    ...
      staticPasswords:
      - email: user@example.com
        hash: <enter the generated hash here>
    ```

## 常见问题

- **Q:** 哪些版本的 Istio、Knative、Cert-Manager、Argo、... 与 Kubeflow 1.4 兼容？ \
  **A:** 请参阅每个单独组件的文档以了解依赖项兼容性范围。对于 `common` 中的 Istio、Knative、Dex、Cert-Manager 和 OIDC-AuthService 版本是我们已经验证过的版本。

- **Q:** 我可以使用最新的 Kustomize 版本 (`v4.x`) 吗？ \
  **A:** Kubeflow 1.4.0 与最新版本的 kustomize 4.x 不兼容。这是由于资源排序和打印发生了变化。请查看 [kubernetes-sigs/kustomize#3794](https://github.com/kubernetes-sigs/kustomize/issues/3794) 及 [kubeflow/manifests#1797](https://github.com/kubeflow/manifests/issues/1797)。我们知道这并不理想，并且正在与上游 kustomize 团队合作，尽快添加对最新版本 kustomize 的支持。

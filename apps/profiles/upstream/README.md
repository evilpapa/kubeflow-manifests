### Manifests

本文件夹包含安装 `profile-controller` 的所有工件。结构如下：

```
.
├── crd
├── default
├── manager
├── rbac
├── samples
├── base
├── overlays
│   ├── kubeflow
│   └── standalone
```

The breakdown is the following:
- `crd`, `default`, `manager`, `rbac`, `samples`: Kubebuilder-generated structure. We keep this in order to be compatible with kubebuilder workflows. This is not meant for the consumer of the manifests.
- `base`, `overlays`: Kustomizations meant for consumption by the user:
    - `overlays/kubeflow`: Installs `profile-controller` as part of Kubeflow. The resulting manifests should be the same as the result of the [deprecated `base_v3` from kubeflow/manifests](https://github.com/kubeflow/manifests/tree/306d02979124bc29e48152272ddd60a59be9306c/profiles/base_v3). At a glance, it makes the following changes:
        - Use namespace `kubeflow`.
        - 移除空间资源。
        - 添加 KFAM 容器。
        - 添加 KFAM Service 和 VirtualService。
    - `overlays/standalone`: 安装 `profile-controller` 到专有空间。Useful for testing or for users that prefer to install just the controller.


### 设置

#### 命名空间标签注入

Profile Controller 为每个 Profile 空间应用多个标签。这些标签通过修改 `namespace-labels` ConfigMap 进行配置。 Refer to the current value for usage instruction.

# 简介
Skaffold是一个命令行工具，可促进Kubernetes应用程序的持续开发。您可以在本地迭代应用程序源代码，然后部署到本地或远程Kubernetes集群。Skaffold处理构建，推送和部署应用程序的工作流程。它还提供了构建块并描述了CI / CD管道的自定义项。

![binaryTree](./img/architecture.png "binaryTree")

1. 收集并监视您的源代码以进行更改
2. 如果用户将文件标记为可同步，则将文件直接同步到Pod
3. 从源代码构建 artifacts
4. 使用容器结构测试来测试构建的 artifacts
5. 标记 artifacts 
6. 推送 artifacts
7. 部署 artifacts
8. 监视部署的 artifacts
9. 在退出时清理部署的 artifacts（Ctrl + C）

Skaffold 支持以下工具：
* Build
    - 本地 Dockerfile
    - 集群内 kaniko 构建
    - 谷歌云上构建
    - 本地 Bazel
    - 本地 Jib Maven/Gradle
* Test
    - 容器结构测试
* Tag
    - tag by git commit 
    - tag by current date&time
    - tag by environment variables based template
* Push
    - 不推送，保留image在本地
    - 推送至远程仓库
* Deploy
    - Kubectl
    - Helm
    - Kustomize
    
除上述步骤外，skaffold还将自动为您管理以下实用程序：
- 使用以下命令将容器端口转发到本地计算机 kubectl port-forward
- 汇总来自已部署Pod的所有日志

#### 示例：
项目地址:
``` sh
git clone https://gitlab.oifitech.com/fanchao/skaffold-demo.git
```
项目结构:
``` sh
 fanchao@fanchaodeMacBook-Pro  ~/go/src/skaffold-demo   master  tree .
.
├── README.md
├── img
│   └── architecture.png
├── kustomize
│   ├── base
│   │   ├── leeroy-app
│   │   │   ├── deployment.yaml
│   │   │   └── kustomization.yaml
│   │   └── leeroy-web
│   │       ├── deployment.yaml
│   │       └── kustomization.yaml
│   └── overlays
│       └── dev
│           ├── kustomization.yaml
│           ├── leeroy-app
│           │   ├── kustomization.yaml
│           │   └── skaffold.yaml
│           └── leeroy-web
│               ├── kustomization.yaml
│               └── skaffold.yaml
├── leeroy-app
│   ├── Dockerfile
│   └── app.go
└── leeroy-web
    ├── Dockerfile
    └── web.go

11 directories, 15 files
```

构建 leeroy-app 的 skaffold 配置，tag 镜像策略这里用的是 git commit, 部署工具用的是 kustomize。
```yaml
apiVersion: skaffold/v2beta1
kind: Config
build:
  artifacts:
    - image: registry.cn-hangzhou.aliyuncs.com/tekton-pipelines/leeroy-app
      context: ../../../../leeroy-app
  tagPolicy:
    gitCommit:
      variant: AbbrevCommitSha   # 使用缩写git commit sha
deploy:
  kustomize:
    paths: ["../../../../kustomize/overlays/dev/leeroy-app"]
```
gitCommit是Skaffold的默认标记策略：如果您未tagPolicy在此build部分中指定字段，Skaffold将使用Git信息标记工件。

skaffold将查看包含工件context目录和标记的Git工作区，并根据以下规则进行标记：

* 如果工作区在Git标签上，则该标签用于标记镜像
* 如果工作空间在Git提交上，则使用短提交
* 如果工作空间中有未提交的更改，则将后缀 -dirty 附加到镜像标签
```
切换目录至 kustomize/overlays/dev/leeroy-app

运行命令：
```
skaffold dev       # 开发模式
skaffold build     # 构建镜像
skaffold deploy    # 部署至k8s
skaffold run       # 构建镜像然后部署至k8s
skaffold delete    # 删除部署
```

skaffold 详细配置说明地址： https://skaffold.dev/docs/references/yaml/
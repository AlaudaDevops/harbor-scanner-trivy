# scanner-trivy alauda 分支开发指南

## 背景

此前，scanner-trivy 在 [harbor](https://github.com/alauda/harbor) 中构建并打包在镜像中，修复漏洞时需要单独 clone 该项目再用 go get 替换，每次构建镜像都需要耗费一定时间编译出 binary。

为了解决上述问题，所以我们基于 [trivy-adapter](https://github.com/goharbor/harbor-scanner-trivy.git) fork 出了当前仓库，并通过 `alauda-vx.xx.xx` 分支来维护。

使用 [renovate](https://gitlab-ce.alauda.cn/devops/tech-research/renovate/-/blob/main/docs/quick-start/0002-quick-start.md) 自动修复对应版本上的漏洞。

## 仓库结构

在原有代码的基础上，添加了以下内容：

1. `.tekton`: 维护 pac 流水线，包含编译、测试、漏洞扫描等步骤，最终会将制品上传到 nexus
2. `.github/release-alauda.yaml`: 使用 goreleaser 自动创建 release
3. `.goreleaser-alauda.yml`: 发布 alauda 版本的 release 的配置文件 

## 维护方案

当需要使用新版本的 scanner-trivy 时，按照以下步骤执行：

1. 从对应 tag 拉出 alauda 分支，例如 `v0.62.1` tag 对应 `alauda-v0.62.1` 分支
2. 将新分支加入到 renovate 的配置文件中，用于自动扫描并修复漏洞
3. renovate 提 PR 后，会自动跑流水线，若所有测试通过，则 PR 将会被自动合并
4. 合并到 `alauda-v0.62.1` 分支后，goreleaser 会自动创建出 `alauda-v0.62.1` release
5. 其他插件中配置的 renovate 会根据配置自动从 release 中获取制品

## 自动化流水线

- `.github/workflow/build.yaml`: 官方测试，包含单元测试、集成测试等，基于 Github Action 运行
- `.tekton/all-in-one.yaml`: pac 流水线，包含编译、测试、漏洞扫描等步骤，基于 Tekton 运行（后续考虑废弃该流水线，仅依靠 Github Action 运行官方测试？）

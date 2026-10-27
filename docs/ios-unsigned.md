# 自动同步上游并构建未签名 IPA

本 Fork 的工作流：`.github/workflows/sync-ios-unsigned.yml`。

## 自动运行

- 每 6 小时检查 `Shirolin/s1er` 的 `main`，计划时间为北京时间
  02:23、08:23、14:23、20:23。GitHub 定时任务可能延迟。
- 使用正常合并同步上游应用代码，不强制覆盖 Fork 的提交。
- `.github/workflows/` 保留 Fork 版本，避免覆盖本构建流程以及触发工作流写入权限问题。
  上游工作流的改动需要手动评估、更新。
- 同一提交已有未过期的 IPA 下载产物时跳过 macOS 构建；构建失败或产物过期时，
  下次检查会重新尝试。上游没有更新时不反复生成相同 IPA。
- 非工作流文件出现合并冲突时停止并报错，不推送冲突内容，也不强制重置分支。

## 手动运行与下载

1. 打开仓库 **Actions**；如果 Fork 的 Actions 尚未启用，先启用。
2. 选择 **Sync upstream and build unsigned IPA** → **Run workflow**，分支选 `main`。
3. 默认勾选 `force_build`，可为当前版本重新打包；取消勾选则复用已有产物。
4. 构建成功后打开该次运行，在 **Artifacts** 下载 `s1er-unsigned-<提交 SHA>`。
5. 解压 ZIP，取得 `.ipa`；同目录提供 SHA-256 校验文件及版本、源码提交信息。

产物保留期配置为 90 天，实际受仓库设置限制。过期后可以手动重建。
GitHub 可能暂停长期无活动的公开仓库定时工作流；届时在 Actions 页面重新启用。

## 构建与安装

macOS runner 使用 Flutter stable 3.44.4（与上游 `.metadata` 中的 revision 一致）；
上游升级 SDK 要求时需同步调整此版本。
安装依赖后直接执行
`flutter build ios --release --no-codesign`，再将设备版 `Runner.app` 打包为 IPA。
按使用者要求，不执行格式检查、静态分析、单元测试或 M3 审计。
同一流程启动新运行时会取消正在进行的旧运行，避免旧任务占用构建时间。
构建不需要配置 Apple ID、签名证书、描述文件或自定义 GitHub Token。
使用仓库自动提供的 `GITHUB_TOKEN` 同步代码和检查已有产物。

IPA 未签名，不能直接安装到普通 iPhone。下载后需要用自己的签名方式签名安装。
本流程仅提供 Actions 构建产物，不修改上游的正式发版脚本及应用内升级清单。

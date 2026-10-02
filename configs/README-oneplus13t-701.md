# 一加 13T：ColorOS 16.0.5.701(CN01)

专用工作流：`Build OnePlus13T 16.0.5.701`。无需填写机型、版本或功能参数。

## 固定的源码

| 项目 | 提交 / 版本 |
| --- | --- |
| 一加 common | `16fe7f198aae3506e9087737b1398c12a3fcf86f`，实际 6.6.89 |
| 一加 msm-kernel | `9cb654690db3dff13b17185674803b84d2e07ace` |
| 一加 modules/devicetree | `0365cea9152355a6a5857c60480aa2f0f0eac542` |
| SukiSU 管理器发布基准 | v4.2.0 / `85eb4a95b8a61d756ecf53b9c5785e48e1b15039` |
| SukiSU builtin 驱动 | `b20dee702035af09cb2ecb5f35443bbc1747f3e6` |
| SUSFS | v2.2.0 / `a5d22370122e0eb2b2f7c5e3cadaa30ea1e3ca2f`，cctv18/susfs4oki |
| KPM 镜像修补工具 | SukiSU-Ultra/SukiSU_patch `547ae94bcaec53d030398f857950c64662043a5d` |
| AnyKernel3 | Numbersf/AnyKernel3 `47f23f7ece3ef212a392ec9ea5466e5f0b55d3c7` |

系统 Android 16 与内核 KMI `android15-6.6` 是不同信息。源码版本固定为 6.6.89，不修改 SUBLEVEL 冒充版本。

驱动整数版本明确固定为 `KSU_VERSION=40900`，不通过 Git 提交数推断。v4.2.0 发布提交的本地 `git rev-list --count` 实测为 3714，旧公式会得到 40899；提交数仅用于日志诊断。builtin 使用 v4.2.0 系列加上恢复 `kernel_umount_feature_set` 的修复，不声称它与管理器 main 发布提交完全相同。管理器的 `40900-2` 标识不作为内核整数驱动版本。

## 修复内容

原构建使用移动的源码分支（已变成 6.6.118）与最新 SUSFS 补丁；最新补丁调用 builtin 中不存在的 `ksu_handle_post_execveat_sucompat`，导致链接失败。

本工作流锁定 SUSFS 2.2.0 的旧接口，并针对 701 源码适配补丁：移除重复的 smaps_rollup 旧上下文 hunk、修复 namespace 文件末尾上下文、将 task_mmu 的 VMA fixup 逻辑改为与该机型原源码一致的调用结构。最终补丁使用实际源码重新生成；24 个目标文件均已通过 `patch --dry-run --fuzz=0`。

仅启用 KSU、KPM、SUSFS；保留原构建器的 OGKI/GKI 转换。首次验证关闭构建缓存，不加入 BBR、BBG、额外 ZRAM 等可选修改。构建失败会保留日志供定位。

## 构建与下载

工作流加入默认分支后，在 Actions 选择 `Build OnePlus13T 16.0.5.701`，点击 Run workflow。

成功后下载 `AK3_OnePlus13T_701_Suki40900_KPM_SUSFS` artifact。解压 GitHub artifact 的外层 ZIP，里面的 `AK3_OnePlus13T_16.0.5.701_6.6.89_Suki40900_KPM_SUSFS.zip` 才是刷入包；旁边有 SHA256 校验文件。

当前只完成配置和补丁检查，尚未跑完整内核构建，也未在手机上验证启动。完整构建成功之前不能把这些静态检查视作 AK3 已可用。

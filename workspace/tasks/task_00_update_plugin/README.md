# task_00_update_plugin

<!-- METADATA:STATUS=Open,ASSIGNEE= -->

## 背景

需要升级 Intern Agent Helper 插件到共享 repo 中 releases 目录下的最高版本。

## 目标

按主管要求执行以下步骤：

1. cd 到共享 repo（通常是 `<WORK_AGENTS_ROOT>/axis_intern_agents/`），切 master，`git pull --rebase origin master`
2. `ls releases/` 找到版本号最高的 `intern-agent-helper-*.vsix`
3. 执行 `code-insiders --install-extension <vsix 绝对路径>`
4. 删除 `<WORK_AGENTS_ROOT>/.github/hooks/.version`，让 hooks 重新解压覆盖
5. 完成后告知安装版本号，并提示用户 Reload Window

## 约束

- 不擅自改代码
- 不擅自改配置
- 只做上述插件升级步骤

## 验收标准

- 共享 repo 已同步 master
- 最高版本 Intern Agent Helper VSIX 已通过 `code-insiders --install-extension` 安装
- hooks `.version` 文件已删除
- 已向用户报告安装版本号并提示 Reload Window

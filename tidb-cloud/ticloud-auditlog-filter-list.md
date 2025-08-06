---
title: ticloud serverless audit-log filter-rule list
summary: `ticloud serverless audit-log filter-rule list` 的参考文档。
---

# ticloud serverless audit-log filter-rule list

列出 TiDB Cloud Serverless 集群的审计日志过滤规则。

```shell
ticloud serverless audit-log filter-rule list [flags]
```

或者使用以下别名命令：

```shell
ticloud serverless audit-log filter list [flags]
```

## 示例

以交互模式列出所有审计日志过滤规则：

```shell
ticloud serverless audit-log filter list
```

以非交互模式列出所有审计日志过滤规则：

```shell
ticloud serverless audit-log filter list -c <cluster-id>
```

以非交互模式并以 JSON 格式列出所有审计日志过滤规则：

```shell
ticloud serverless audit-log filter list -c <cluster-id> -o json
```

## 参数说明

在非交互模式下，你需要手动输入所需的参数。在交互模式下，你只需按照 CLI 提示填写即可。

| Flag                    | Description                                                                                       | Required | Note                                                 |
|-------------------------|---------------------------------------------------------------------------------------------------|----------|------------------------------------------------------|
| -c, --cluster-id string | 你想要列出其审计日志过滤规则的集群 ID。                                        | No       | 仅在非交互模式下生效。                  |
| -o, --output string     | 指定输出格式（默认为 `human`）。有效值为 `human` 或 `json`。如需获取完整结果，请使用 `json` 格式。 | No       | 在非交互和交互模式下均可使用。 |
| -h, --help              | 显示该命令的帮助信息。                                                          | No       | 在非交互和交互模式下均可使用。 |

## 继承参数

| Flag                 | Description                                                                                          | Required | Note                                                                                                             |
|----------------------|------------------------------------------------------------------------------------------------------|----------|------------------------------------------------------------------------------------------------------------------|
| --no-color           | 禁用输出中的颜色。                                                                            | No       | 仅在非交互模式下生效。在交互模式下，禁用颜色可能会影响部分 UI 组件的显示。 |
| -P, --profile string | 指定该命令使用的活动 [用户配置文件](/tidb-cloud/cli-reference.md#user-profile)。 | No       | 在非交互和交互模式下均可使用。                                                             |
| -D, --debug          | 启用调试模式。                                                                                  | No       | 在非交互和交互模式下均可使用。                                                             |

## 反馈

如果你对 TiDB Cloud CLI 有任何问题或建议，欢迎创建 [issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)。同时也欢迎任何贡献。
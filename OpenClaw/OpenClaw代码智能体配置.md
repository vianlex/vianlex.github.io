

## TOOLS.md 文件

```bash
# TOOLS.md - OpenClaw 工具注册清单
_更新时间：2026-03-15 | 适配版本：OpenClaw 2026.3.13_

## 一、代码类工具（Code Tools）
| 工具名称 | 调用路径 | 依赖插件 | 权限要求 | 入参规范 | 输出格式 | 执行超时 | 版本约束 | 环境变量 | 错误码 | 备注 |
|----------|----------|----------|----------|----------|----------|----------|----------|----------|--------|------|
| java-code-generate | skills/java-code-generate.py | java-tools@2.0.0 | system:file.write,model:codeqwen.call | `{"input":"string(必填)","output":"string(默认:./gen-code)","framework":"string(默认:springboot-3.2)"}` | text/java,application/json | 60 | >=2026.3.0 | JAVA_HOME=/usr/lib/jvm/java-17 | 1001:模型调用失败<br>1002:输出路径无权限 | 遵循阿里巴巴Java开发规范 |
| java-code-style-check | skills/java-code-style-check.sh | java-tools@2.0.0 | system:file.read,system:command.exec | `{"input":"string(必填)","rule":"string(默认:alibaba)"}` | text/markdown | 30 | >=2026.3.0 | CHECKSTYLE_HOME=/opt/checkstyle | 2001:代码文件不存在<br>2002:规则文件缺失 | 基于 CheckStyle 10.12.0 |
| java-dependency-analyze | skills/java-dependency-analyze.py | maven-tools@2.0.0 | system:file.read,system:command.exec | `{"input":"string(必填)","format":"string(默认:json)"}` | application/json,text/markdown | 90 | >=2026.3.0 | MAVEN_HOME=/opt/maven | 3001:pom.xml解析失败<br>3002:依赖冲突检测失败 | 支持 Maven/Gradle 项目 |
| python-code-analyze | skills/python-code-analyze.py | python-tools@2.0.0 | system:file.read,system:command.exec | `{"input":"string(必填)","check_type":"string(默认:performance)"}` | text/markdown | 45 | >=2026.3.0 | PYTHONPATH=/usr/local/lib/python3.11 | 4001:语法错误<br>4002:性能分析超时 | 支持 pandas/numpy 代码分析 |

## 二、文档类工具（Document Tools）
| 工具名称 | 调用路径 | 依赖插件 | 权限要求 | 入参规范 | 输出格式 | 执行超时 | 版本约束 | 环境变量 | 错误码 | 备注 |
|----------|----------|----------|----------|----------|----------|----------|----------|----------|--------|------|
| javadoc-generate | skills/javadoc-generate.py | java-tools@2.0.0 | system:file.read,system:file.write | `{"input":"string(必填)","output":"string(默认:./javadoc)","format":"string(默认:html)"}` | text/html,text/markdown | 60 | >=2026.3.0 | JAVA_HOME=/usr/lib/jvm/java-17 | 5001:代码解析失败<br>5002:文档生成失败 | 适配 SpringDoc 3.0+ |
| swagger-generate | skills/swagger-generate.py | java-tools@2.0.0 | system:file.read,system:file.write | `{"input":"string(必填)","version":"string(默认:3.0)"}` | application/json,application/yaml | 30 | >=2026.3.0 | - | 6001:Controller识别失败<br>6002:注解解析错误 | 支持 OpenAPI 3.0 规范 |
| feishu-doc-create | skills/feishu-doc-create.py | feishu@3.0.0 | feishu:docx.document.create,feishu:drive.folder.read | `{"title":"string(必填)","content":"string(必填)","folderId":"string(可选)"}` | application/json（飞书文档链接） | 20 | >=2026.3.0 | FEISHU_APP_ID=xxx,FEISHU_APP_SECRET=xxx | 7001:飞书API调用失败<br>7002:文件夹无权限 | 支持 Markdown 转富文本 |
| feishu-doc-edit | skills/feishu-doc-edit.py | feishu@3.0.0 | feishu:docx.document.update | `{"docId":"string(必填)","content":"string(必填)","type":"string(默认:append)"}` | application/json | 20 | >=2026.3.0 | FEISHU_APP_ID=xxx,FEISHU_APP_SECRET=xxx | 8001:文档不存在<br>8002:编辑权限不足 | 支持追加/替换内容 |

## 三、数据类工具（Data Tools）
| 工具名称 | 调用路径 | 依赖插件 | 权限要求 | 入参规范 | 输出格式 | 执行超时 | 版本约束 | 环境变量 | 错误码 | 备注 |
|----------|----------|----------|----------|----------|----------|----------|----------|----------|--------|------|
| feishu-bitable-read | skills/feishu-bitable-read.py | feishu@3.0.0 | feishu:bitable.read | `{"appToken":"string(必填)","tableId":"string(必填)","filter":"string(可选)"}` | application/json,application/vnd.ms-excel | 30 | >=2026.3.0 | FEISHU_APP_ID=xxx,FEISHU_APP_SECRET=xxx | 9001:表格不存在<br>9002:筛选条件错误 | 支持分页/筛选读取 |
| feishu-bitable-write | skills/feishu-bitable-write.py | feishu@3.0.0 | feishu:bitable.write | `{"appToken":"string(必填)","tableId":"string(必填)","data":"array(必填)"}` | application/json | 30 | >=2026.3.0 | FEISHU_APP_ID=xxx,FEISHU_APP_SECRET=xxx | 10001:数据格式错误<br>10002:写入权限不足 | 支持批量插入/更新 |
| data-calculate | skills/data-calculate.py | python-tools@2.0.0 | system:file.read | `{"input":"string(必填)","calc_type":"string(默认:mom)"}` | application/json | 45 | >=2026.3.0 | PANDAS_HOME=/usr/local/lib/python3.11/site-packages | 11001:数据解析失败<br>11002:计算类型不支持 | mom=环比,yoy=同比 |
| data-visualize | skills/data-visualize.py | python-tools@2.0.0 | system:file.write | `{"input":"string(必填)","chart_type":"string(默认:line)","output":"string(默认:./chart.png)"}` | image/png,application/json（图表配置） | 60 | >=2026.3.0 | MATPLOTLIBRC=/etc/matplotlib | 12001:数据为空<br>12002:图表生成失败 | 支持折线图/柱状图/饼图 |

## 四、测试类工具（Test Tools）
| 工具名称 | 调用路径 | 依赖插件 | 权限要求 | 入参规范 | 输出格式 | 执行超时 | 版本约束 | 环境变量 | 错误码 | 备注 |
|----------|----------|----------|----------|----------|----------|----------|----------|----------|--------|------|
| junit-generate | skills/junit-generate.py | java-tools@2.0.0 | system:file.read,system:file.write | `{"input":"string(必填)","framework":"string(默认:junit5)","mock":"boolean(默认:true)"}` | text/java | 60 | >=2026.3.0 | JAVA_HOME=/usr/lib/jvm/java-17 | 13001:类解析失败<br>13002:Mock数据生成失败 | 支持 Mockito 4.0+ |
| junit-exec | skills/junit-exec.sh | maven-tools@2.0.0 | system:command.exec,system:file.write | `{"input":"string(必填)","output":"string(默认:./test-report)"}` | text/html,text/markdown | 120 | >=2026.3.0 | MAVEN_HOME=/opt/maven | 14001:测试执行失败<br>14002:覆盖率分析失败 | 集成 Jacoco 0.8.10 |
| api-test-generate | skills/api-test-generate.py | code-exec@2.0.0 | system:file.write | `{"input":"string(必填)","format":"string(默认:postman)"}` | application/json | 30 | >=2026.3.0 | - | 15001:接口文档解析失败<br>15002:用例生成失败 | 支持 RESTful/gRPC API |
| api-test-exec | skills/api-test-exec.py | code-exec@2.0.0 | system:network.access,system:file.write | `{"input":"string(必填)","baseUrl":"string(必填)","timeout":"number(默认:30)"}` | text/markdown | 90 | >=2026.3.0 | - | 16001:接口访问失败<br>16002:响应校验失败 | 支持 HTTP/HTTPS |

## 五、调试类工具（Debug Tools）
| 工具名称 | 调用路径 | 依赖插件 | 权限要求 | 入参规范 | 输出格式 | 执行超时 | 版本约束 | 环境变量 | 错误码 | 备注 |
|----------|----------|----------|----------|----------|----------|----------|----------|----------|--------|------|
| java-exception-analyze | skills/java-exception-analyze.py | java-tools@2.0.0 | system:file.read | `{"input":"string(必填)","exception_type":"string(可选)"}` | text/markdown | 45 | >=2026.3.0 | JAVA_HOME=/usr/lib/jvm/java-17 | 17001:堆栈解析失败<br>17002:根因分析失败 | 支持 NPE/OOM/死锁等 |
| jvm-log-parse | skills/jvm-log-parse.py | java-tools@2.0.0 | system:file.read | `{"input":"string(必填)","keyword":"string(默认:GC)"}` | application/json,text/markdown | 60 | >=2026.3.0 | JAVA_HOME=/usr/lib/jvm/java-17 | 18001:日志格式错误<br>18002:关键字无匹配 | 支持 GC/堆转储/线程日志 |
| multi-lang-debug | skills/multi-lang-debug.py | code-exec@2.0.0 | system:file.read | `{"input":"string(必填)","lang":"string(默认:java)"}` | text/markdown | 60 | >=2026.3.0 | - | 19001:语言不支持<br>19002:异常分析失败 | 支持 Java/Python/JS/Go |
| code-fix-suggest | skills/code-fix-suggest.py | java-tools@2.0.0 | system:file.read,system:file.write | `{"input":"string(必填)","error":"string(必填)"}` | text/java,text/markdown | 60 | >=2026.3.0 | - | 20001:修复建议生成失败<br>20002:代码写入失败 | 基于 CodeQwen-7B 模型 |
```
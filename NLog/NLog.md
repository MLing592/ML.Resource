## NLog 配置说明

### 版本基础
NLog 配置模板基于 **NLog V6** 版本开发，使用前需将配置文件名统一改为 `NLog.Config`。

### 设计目标
1. 在指定时间（年月日）内，目标记录器记录对应事件的指定级别日志消息，示例路径：`Logs/Active/MainApp/Debug.log`
2. 日志记录格式：`时间(年月日时分秒毫秒)|线程名|线程ID|消息 异常信息`,示例`2025-01-01 23:04:13.0645|:1|这是一条日志`
3. 归档功能：
* `Logs/Active/`：存储最新未归档日志
* `Logs/Archive/`：存储已归档日志
* 超出预设规则后，自动删除旧归档日志

### 版本差异说明
| 版本 | 核心功能差异 | 关键特性                                                                              |
| -- | ------ | --------------------------------------------------------------------------------- |
| V1 | 基础日志写入 | 1. 支持日志写入本地文件（File）和控制台（Console） <br>2. 新增`loggerName="ToJson"`日志器，将日志内容格式化为 JSON 格式输出 |
| V2 | 新增自动归档 | 1. 移除V1中的`ToJson` 日志器<br>2. 新增 NLog 内部错误日志输出 <br>3. 新增自动归档规则：日志文件超过指定大小 / 达到指定时间后，自动转移至归档目录  |
| V3 | 新增归档压缩 | 1. 继承V2自动归档能力,归档文件自动压缩为 `.zip` 格式<br>2. 多份满足归档条件的文件按序号滚动生成（避免覆盖）                 |

### 依赖说明
1. `NLog: 6.0.3`
2. `NLog.Targets.ConcurrentFile: 6.0.3`

### 补充说明
1. 归档规则：文件大小阈值 / 时间周期可通过配置自定义，超出阈值的旧归档文件会按预设规则（保留时长 / 文件数量）自动清理；
2. 兼容性：所有版本均兼容 NLog V6 核心特性，依赖 `NLog.Targets.ConcurrentFile` 扩展实现多线程安全写入；
3. 命名规范：归档压缩文件命名格式为 `{时间}_{序号}_{日志级别}_{日志器名称}.zip`，便于检索和版本追溯；
4. 依赖说明：NLog6不再支持`EnableArchiveFileCompression`压缩，使用了NLog 5 对应的`NLog.Targets.ConcurrentFile`扩展支持压缩功能;

### 示例代码
以下是 NLog 日志输出的 C# 示例代码，包含普通文本日志和对象结构化日志：

```C#
static Logger \_log = NLog.LogManager.GetLogger("MainApp");
_log.Info("Hello World");
_log.Info("person info: {@Info}", new Person());
```

### 官方文档引用
1. 文件日志说明：[https://github.com/NLog/NLog/wiki/File-target](https://github.com/NLog/NLog/wiki/File-target)
2. NLog V6 更新说明：[https://nlog-project.org/2025/04/29/nlog-6-0-major-changes.html](https://nlog-project.org/2025/04/29/nlog-6-0-major-changes.html)

### 更新说明
1. V2、V3当loggername为`xxxx/xxxx`带有目录解析符时，归档路径`{####}_${level}_${logger}.log`解析时{xxxx}解析失效，因此将`${logger}`作为目录不再作为文件
1. V1、V2、V3给规则`rule`添加`ruleName`，方便代码控制
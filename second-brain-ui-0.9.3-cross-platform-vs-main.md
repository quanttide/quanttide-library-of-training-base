# second-brain-ui-0.9.3-cross-platform 分支相对主分支的增量

本文记录 `BlackCat205/quanttide-profile-of-agent-engineering` 的 `second-brain-ui-0.9.3-cross-platform` 分支相比主分支额外做了什么。对比方法是把仓库克隆到临时目录后直接用 Git 比较，不是基于网页摘要的转述。

---

## 对比口径

克隆仓库到 `/tmp/qt-profile-compare`，取 `main` 与 `origin/second-brain-ui-0.9.3-cross-platform` 两个引用，执行 `git diff --stat`、`git diff --numstat` 与 `git log`。复现命令如下：

```sh
git clone https://github.com/BlackCat205/quanttide-profile-of-agent-engineering.git
cd quanttide-profile-of-agent-engineering
git fetch origin second-brain-ui-0.9.3-cross-platform
git log --oneline main..FETCH_HEAD
git diff --stat main...FETCH_HEAD
```

对比结果：主分支指向 `2b5666c`（2026-09-04），分支头指向 `287fe1b`（2026-09-21），两者的合并基点就是主分支本身。分支领先主分支 39 个提交，落后 0 个提交，因此这是一个可以直接快进合并的纯增量分支，与主分支没有分叉。

差异统计为 256 个文件变更、14999 行新增、0 行删除。其中 253 个文件是新增，3 个文件是修改：`CHANGELOG.md`、`README.md`、`quanttide-asset/README.md`。零删除意味着分支没有改动主分支的既有内容，只在其上追加。

## 增量的构成

按目录统计新增行数，可以看清这批增量花在哪里。

| 部分 | 位置 | 规模 |
|:--|:--|--:|
| 测试证据 | `quanttide-asset/loops/second-brain-init/trials/` | 196 个文件，8228 行 |
| 自动化测试 | `quanttide-asset/loops/second-brain-init/tests/` | 17 个文件，2041 行 |
| 可复用技能与执行器 | `quanttide-asset/skills/second-brain-init/` | 2125 行 |
| 中文网页向导 | `quanttide-asset/loops/second-brain-init/ui/` | 1379 行 |
| 使用与交付文档 | `quanttide-asset/loops/second-brain-init/` 下除 `ui`、`tests`、`trials` 外 | 682 行 |
| 启动、登记与说明文件 | 仓库根目录、`quanttide-asset/` | 544 行 |

六行相加恰为 14999 行。可执行脚本合计 34 个文件、5283 行，其中 Python 26 个文件 4891 行、JavaScript 4 个文件 309 行、中文命名的 shell 与批处理启动脚本 4 个文件 83 行；其余主要是 JSON 证据 6216 行与 Markdown 文档 2092 行。测试证据占全部新增行数的 55%，是这批增量里体量最大的部分。

## 这些增量在做什么

分支围绕一个目标展开：把原本只存在于对话记录里的「第二大脑创建流程」变成一个非技术同事能双击启动、按页面提示完成的本机工具。

第一部分是可双击启动的中文本机向导。根目录新增 `launch.py`、`首次配置.bat`、`启动向导.bat` 及对应的 `.sh` 版本和 `启动前检查.html`，`START_HERE.md` 是面向使用者的入口文档。两个批处理文件的分工需要区分：`启动向导.bat` 只有 11 行，是纯入口，负责切到自身目录、确认虚拟环境存在、调用 `launch.py`；`首次配置.bat` 32 行实现的是环境编排，包括探测 Git 与 Python 3.10+、创建虚拟环境、安装依赖、运行一次契约检查，每步都有错误分支，但同样不含业务逻辑。对应的 `.sh` 文件分工一致，业务实现全部在 Python 里。向导本体是 `ui/app.py`（886 行），它只监听 `127.0.0.1`，提供「填写需求、审阅方案、确认执行、逐项检查、记录人工验收意见」的完整页面流程；`github_login.py` 负责调用官方 GitHub CLI 完成授权，`survey.py` 提供不写入的只读现状查询。界面版本从 0.3.0 迭代到 0.9.3。

第二部分是把流程固化为可复用技能与执行器。`skills/second-brain-init/` 内含 `SKILL.md`、`specification.yaml` 与 `engine.py`（1615 行）。规格文件定义 20 类资产、7 个场景（新建、补全、追加、挂载、聚合、更名、发布），并把来源锁定到具体提交：原始流程 `2b5666c`、资产章程 `0301294`。执行器提供 plan、approve、apply、verify 四步的可恢复 Git 工作流，所有操作绑定计划编号，未确认不执行。`quanttide-asset/.quanttide/asset/contract.yaml` 与 `asset-entry.py` 把这套插件登记给资产云的 Rust CLI，使其可被发现和调用。

第三部分是失败恢复与可观测性，这也是提交历史里着墨最多的地方。39 个提交里有 11 个是修复与重构（10 个 `fix`、1 个 `refactor`），其余为功能、文档与发版提交。按时间顺序，这条线依次解决：只读调查不把网络失败当成名称可用（0.4.0）、个人 GitHub 账号流程与操作进度（0.5.0）、旧任务漏交文件的窄范围补交（0.6.0）、章程变更与读取失败的区分（0.6.2）、建仓回执丢失后的审阅接续（0.8.6）、挂载复用本地已核验仓库以消除重复下载（0.8.4）、推送响应丢失后先核对远端再有限重试（0.8.5）、远端写入前先持久化凭据与跨进程互斥（0.9.0）。这些修复针对的是真实试用中 Windows 断网、文件占用、推送超时等具体故障，而不是假设场景。

第四部分是测试与证据。`tests/` 下的测试用真实本地 Git 仓库配合模拟的 GitHub 传输做故障注入，覆盖建仓、推送、挂载、中断恢复等路径。`trials/` 保存了 196 个证据文件，包括原始测试输出、浏览器截图、一次真实公开仓库的只读调查报告（`public-survey-040/report.json`），以及从资产云源码编译 Rust CLI 后的验收记录（`native-061/`）。值得注意的是证据里保留了失败记录，例如 `evidence-002/summary.json` 的状态是 `failed`、`ui-browser-003` 是浏览器启动失败，没有被改写成通过。

第五部分是文档。除 `START_HERE.md` 外，新增验收表、使用说明、交付清单、接入说明、Windows 试用表、断网恢复说明、上线与联调说明与提交评审说明，`CHANGELOG.md` 追加了 30 行版本记录。

## Python 代码的分层与核心功能

Python 是这批增量的主体，26 个文件、4891 行，自下而上分为四层。

| 层 | 文件（行数） | 职责 |
|:--|:--|:--|
| 启动 | `launch.py`（35） | 版本与依赖预检，进程级注入 Git 配置，再拉起 `app.py` |
| 接入 | `quanttide-asset/asset-entry.py`（46） | 读取资产云契约、校验登记路径，派发到执行器命令行 |
| 界面 | `ui/app.py`（886）与 `survey.py`（103）、`github_login.py`（71）、`repair.py`（79）、`partial_recovery.py`（104） | 本机 HTTP 服务、恢复决策与两个专用修复入口 |
| 引擎 | `skills/second-brain-init/scripts/engine.py`（1615） | 计划、确认、执行、验证的全部实现 |

`app.py` 的结构是 `Studio` 类（约 570 行）加 `Handler` 路由。`create` 建任务记录，`planning_task` 完成调查并生成方案，`execute` 先做执行前漂移检查再调用引擎，`build_report` 生成中文逐项报告，`acceptance` 保存人工验收意见；`connection_task`、`partial_task`、`repair_task` 是三个恢复入口，`recovery_guidance` 决定暂停状态应当对应哪个恢复动作。

`engine.py` 中，`Provider` 类用同一接口实现本地 bare 仓库与 GitHub 两种后端，`Executor` 类实现建仓、挂载、领域文档、契约、目录、根索引、提交推送、更名与发布等操作，`command()` 是所有 Git 与 `gh` 调用的统一出口：只读操作有限重试，写入不重试，并记录观测事件。

核心功能是一个四步闭环。`make_plan` 先做只读调查，再生成绑定引擎与规格指纹、锁定来源提交的计划；`approve` 要求确认记录绑定完整计划编号，改一个字就得重新规划；`apply` 按检查点逐步执行，每步之前核对相关仓库是否被人改动；`verify` 逐项核对提交一致性、子模块指针、目录坐标与契约和 `.gitmodules` 的对应关系。决定它行为边界的有三个设计：没有确认不创建工作区，模拟确认只允许本地模式；远端写入前先持久化写入意图，同一工作区与同一总入口跨进程加锁；结果不确定时不重试而先对账，推送失败先读远端确认提交是否到达，中断后由三个对账函数判断能否接续，对不上就停在暂停态交给人。这也是代码里大量篇幅用于失败后现场保护、而真正调用的 Git 命令只有几种的原因。

实现上不求值任何 shell 字符串，全部 Git 调用走 `subprocess` 参数数组；17 个测试文件用真实本地 Git 仓库验证上述路径，而不是把 Git mock 掉。

## 验证边界与合并风险

分支自己的文档声明了尚未完成的验收：Windows 真人实机操作、真实 GitHub 创建流程、资产云在线部署均待完成，已有证据主要来自 Linux 环境与模拟的 GitHub 接口。因此「测试通过」的适用范围应以 `trials/` 中的记录为准，不能等同于端到端验收通过。

合并时需要留意三点。其一，39 个提交中有 16 个提交信息完全相同（`feat(second-brain): publish 0.9.2`），提交历史较脏，若在意线性历史可考虑压缩合并。其二，`trials/` 占新增体积的 55%，其中包含执行日志、计划 JSON 与截图，是否随代码一起进主分支值得权衡。其三，分支在根目录新增了中文命名的批处理与 shell 文件，并新增 `.gitattributes` 声明 `*.bat -text`，合并后主分支的目录结构和行尾处理都会随之改变。

临时克隆目录为 `/tmp/qt-profile-compare`，确认结论后可直接删除。

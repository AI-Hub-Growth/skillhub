---
name: qiniu-test-record
description: 在用户指定的项目文件夹中扫描测试，生成并执行类似 pytest tests/unit/providers/test_qiniu_provider.py --tb=short 的命令，将命令与结果写入项目根目录 test.txt（每条「结果」行仅为 N passed M failed；不同命令块之间空行分隔）。仅当用户明确要求使用本 skill 并提供目标文件夹时使用。
---

# Record Test Runs

## 使用条件

仅当用户主动要求使用此 skill，并且明确提供目标文件夹时使用。

如果目标文件夹缺失或含糊，先询问用户确认文件夹，不要扫描或运行测试。

## 目标

针对指定目标文件夹：

1. 在测试前安装并准备测试所需环境（依赖、虚拟环境或项目约定的前置步骤）。
2. 扫描项目中的所有测试。
3. 生成运行全部测试的命令（见下方「命令格式」）。
4. 运行该命令。
4. 在项目根目录创建或覆盖 `test.txt`。
5. 将全量测试命令和结果记录到 `test.txt`（`结果:` **仅有** `xx passed xx failed`）。
6. 如果测试中出现因不同平台环境导致的 failed，将该部分按 passed 统计，不计入 failed。
7. 如果存在 qiniu 相关测试，单独运行这些测试。
8. 按同样格式追加 qiniu 相关命令和结果；**不同命令块之间用空行隔开**。
9. 如果没有任何的测试文件，`test.txt` 中对应测试记录项留空（不写 `命令:` / `结果:`）。

## 命令格式

- 写入 `命令:` 行的字符串必须是**可直接在终端运行的单行命令**，形态类似：
  - `pytest tests/unit/providers/test_qiniu_provider.py --tb=short`
  - 即：`pytest` + 空格 + 测试路径或目录 + 可选固定参数如 `--tb=short`（按需保留项目既有约定，但不要叠加 PowerShell 包装、多余管道或冗长参数）。
- 非 pytest 项目：仍写**最简的一行**等价命令（例如 `go test ./pkg/...`），原则与本条一致。

## 工作流程

1. 确定项目根目录和目标文件夹。
   - 目标文件夹是测试扫描范围。
   - 最近的仓库根目录或包根目录是写入 `test.txt` 的位置。

2. 判断测试框架和常规测试命令。
   - 检查 `package.json`、`pytest.ini`、`pyproject.toml`、`go.mod`、`Cargo.toml`、`pom.xml`、`build.gradle` 等常见项目文件和本地约定。
   - 优先使用现有项目脚本对应的**最简等价一行**；不要随意发明复杂命令。
   - 命令必须兼容 PowerShell（但最终写入 `test.txt` 的仍是上面那种单行命令字符串）。

3. 在运行测试前安装测试所需环境。
   - 优先使用项目已有、最简且可复现的一行命令（如 `pip install -r requirements.txt`、`poetry install`、`npm ci`、`go mod download`）。
   - 安装命令不写入 `test.txt`，`test.txt` 仅记录测试命令与结果。

4. 扫描目标文件夹下的测试文件。
   - 覆盖常见测试命名，例如 `*.test.*`、`*.spec.*`、`test_*.py`、`*_test.py`、`*_test.go` 和测试目录。
  - 如果没有发现测试，该项留空，不写 `命令:` / `结果:`。

5. 生成全量测试命令。
   - 若使用 pytest：写成 `pytest <目标范围内的路径或目录> --tb=short` 这类形式（路径按实际扫描结果填写）。
   - 若框架只提供项目级脚本，用一行脚本调用，仍保持尽量短。

6. 从项目根目录在 PowerShell 中运行全量测试命令。
   - 捕获 stdout、stderr 和退出码。
   - 将结果整理为 **`N passed M failed`** 写入 `结果:` 行（可从框架输出提取或根据实际通过/失败数归纳）。
   - 若某次结果为 `0 passed 0 failed`，先排查该命令是否有误（如路径不存在、筛选条件过窄、命令未命中任何测试）再记录。

7. 在项目根目录写入 `test.txt`。
   - 全量测试记录必须严格使用两行；`结果:` **整行只有** `N passed M failed`，后面不接退出码、括号、说明或任何其它字符；无法区分时用 `0 passed 0 failed`。

```text
命令: pytest tests/unit/providers/test_qiniu_provider.py --tb=short
结果: 3 passed 1 failed
```

8. 识别 qiniu 相关测试。
   - 路径、文件名、测试名或测试内容中包含 `qiniu` 的测试都算 qiniu 相关测试，匹配时忽略大小写。
   - 如果发现 qiniu 相关测试，生成**尽可能窄的一行命令**，只运行这些测试（pytest 时同上，`pytest <路径> --tb=short`）。
   - 若需顺序执行多次，每段仍按「命令两行 + 空行」记录；每段 `结果:` 同样仅 `N passed M failed`。

9. 单独运行 qiniu 相关测试。
   - 捕获 stdout、stderr 和退出码。
   - 在向 `test.txt` 追加前，若已有全量记录，先输出一个**空行**，再追加严格两行：

```text

命令: pytest tests/unit/providers/test_qiniu_provider.py --tb=short
结果: 2 passed 0 failed
```

10. 如果无法安全判断测试框架：
   - 不要猜测并运行不可靠命令。
   - 汇报已发现的测试文件和候选命令。
   - 先让用户确认命令，再运行测试或写入最终结果。

## 结果记录规则

- `test.txt` 必须创建在当前根目录。
- 第一条记录是全量测试命令和结果；若未发现项目自带测试，则该项留空。
- 只有发现 qiniu 相关测试时，才追加第二条记录。
- 每条记录必须是两个逻辑行：一行 `命令:`，一行 `结果:`。
- **`结果:` 行只能是 `N passed M failed`（数字与 passed/failed 之间空格），行尾不得追加退出码、括号、备注或其它文本。**
- **不同命令块之间（全量与 qiniu、或多段 qiniu 命令）必须用空行隔开**：上一段的 `结果:` 行之后接一个空行，再写下一段的 `命令:`。
- 如果命令失败，仍然记录命令；`结果:` 仍只写 `N passed M failed`（failed 按框架输出或归纳填写）。
- 若 failed 明确由不同平台环境差异导致（如系统/架构/平台特性差异），应将这部分 failed 计入 passed，并在最终 `结果:` 中按 `N passed M failed` 体现为不增加 failed。
- 若出现 `0 passed 0 failed`，先检查该条 `命令:` 是否正确并确认是否实际执行到测试。

## 安全要求

- 使用此 skill 时不要编辑源代码文件。
- 测试前应先安装测试所需环境，优先采用项目既有安装方式。
- 优先使用项目已有脚本和本地约定。
- 保留用户已有的无关改动。

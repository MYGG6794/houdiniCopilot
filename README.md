### **项目开发文档：Houdini AI 助手 (Houdini Copilot)**

#### **1. 项目目标与愿景**

创建一个集成在 Houdini 中的 Python Panel 工具。该工具提供一个 UI，允许用户通过自然语言输入指令。这些指令将被发送到大型语言模型 (LLM)，LLM 会生成相应的 Houdini Python 脚本。最后，该脚本将在当前 Houdini 场景中执行，从而实现用自然语言操控 Houdini 的目的。

#### **2. 技术架构**

*   **前端 (UI)**: 使用 Houdini 内置的 **PySide2** 库创建一个浮动窗口 (Python Panel)。
*   **后端逻辑**: 使用纯 Python 编写，负责处理 UI 事件、构建 Prompt、与外部 API 通信。
*   **AI 服务**: 对接 **OpenAI API** (可以使用 GPT-4 或 GPT-3.5-turbo 模型)。
*   **Houdini 通信**: 使用 Houdini 的 `hou` Python 模块来获取场景信息和执行代码。
*   **部署**: 通过一个安装脚本将该工具注册为 Houdini 的一个 Python Panel。

#### **3. 开发阶段与 AI 指导任务**

---

#### **阶段一：项目结构与环境设置**

**目标**: 创建项目文件结构，并编写一个安装脚本，用于将工具集成到 Houdini 中。

**`AI 指令 1: 创建项目文件`**
“请为我的 Houdini AI 助手项目创建以下文件结构：
- `/Houdini_AI_Assistant` (项目根目录)
  - `main.py` (包含 UI 和核心逻辑的主文件)
  - `install.py` (用于在 Houdini 中安装 Python Panel 的脚本)
  - `config.py` (用于存放 API 密钥和设置)
  - `README.md` (项目说明)”

**`AI 指令 2: 编写安装脚本 (install.py)`**
“在 `install.py` 文件中，编写一个 Python 脚本，该脚本应遵循 Houdini Python Panel 的注册规范。
1.  定义一个 XML 字符串，用于描述 Panel 的 ID、标签和图标。
2.  使用 `hou.pythonPanels.install()` 函数来注册这个面板。
3.  确保将 `main.py` 文件中的主类（我们稍后会创建，可暂时命名为 `HoudiniCopilotPanel`）作为面板的创建脚本。
4.  将此脚本包裹在一个函数 `install_panel()` 中，并提供卸载函数 `uninstall_panel()`。
5.  在 Houdini 的 Python Shell 中运行 `install_panel()` 即可完成安装。”

**`AI 指令 3: 编写配置文件 (config.py)`**
“在 `config.py` 文件中，定义一个变量 `OPENAI_API_KEY`。其值应从环境变量 `OPENAI_API_KEY` 中获取，如果环境变量不存在，则设置为空字符串。这可以避免将密钥硬编码在代码中。”

---

#### **阶段二：用户界面 (UI) 开发**

**目标**: 使用 PySide2 在 `main.py` 中创建工具的用户界面。

**`AI 指令 4: 创建主窗口类 (main.py)`**
“在 `main.py` 文件中，导入 `PySide2.QtWidgets` 和 `hou`。
创建一个名为 `HoudiniCopilotPanel` 的类，它继承自 `QWidget`。
在 `__init__` 方法中，设置窗口的布局。UI 应包含以下组件：
1.  一个 `QLabel` 标题，内容为 "Houdini AI Assistant"。
2.  一个 `QTextEdit` (命名为 `self.prompt_input`)，用于用户输入自然语言提示，并设置一个占位符文本，如 "例如：创建一个球体，并对其应用山脉噪波"。
3.  一个 `QPushButton` (命名为 `self.generate_button`)，文本为 "生成并执行"。
4.  一个 `QTextEdit` (命名为 `self.log_output`)，设置为只读，用于显示日志、AI 返回的代码和执行结果。”

**`AI 指令 5: 实现工厂函数`**
“在 `main.py` 文件的末尾，根据 Houdini Python Panel 的要求，创建一个名为 `createInterface()` 的工厂函数。此函数应返回 `HoudiniCopilotPanel` 类的一个实例。”

---

#### **阶段三：AI 通信与核心逻辑**

**目标**: 实现与 OpenAI API 的通信，并构建有效的 Prompt。

**`AI 指令 6: 实现 API 请求函数 (main.py)`**
“在 `HoudiniCopilotPanel` 类中，创建一个名为 `query_ai` 的方法。
1.  该方法接收一个字符串参数 `user_prompt`。
2.  导入 `openai` 库和 `config` 模块。
3.  设置 `openai.api_key` 为 `config.OPENAI_API_KEY`。
4.  **构建一个 System Prompt**: 这个 Prompt 非常关键。它应该告诉 AI 它的角色。例如：
    *   `"你是一个精通 Houdini 的技术美术 (TD)，尤其擅长编写用于操控场景的 Python 脚本。你的回答必须只包含可直接在 Houdini Python Shell 中执行的原始 Python 代码。不要添加任何 markdown 标记、解释、注释或 'python' 标识。代码必须使用 `hou` 模块。"`
5.  使用 `openai.ChatCompletion.create` 方法发送请求。
    *   `model`: 使用 "gpt-4" 或 "gpt-3.5-turbo"。
    *   `messages`: 包含两条消息，一条是 `role: "system"`，内容是你的 System Prompt；另一条是 `role: "user"`，内容是 `user_prompt`。
6.  该方法应返回 AI 生成的代码字符串。使用 `try-except` 块处理网络或 API 错误，并将错误信息记录到 `self.log_output` 中。”

**`AI 指令 7: 获取 Houdini 场景上下文`**
“在 `HoudiniCopilotPanel` 类中，创建一个名为 `get_scene_context` 的方法。
1.  该方法应检查用户当前在 Houdini 中选择了哪些节点。
2.  使用 `hou.selectedNodes()` 获取节点元组。
3.  将选中节点的路径和类型格式化为一个简洁的字符串。例如：`"当前选中的节点：/obj/geo1 (type: geo), /obj/cam1 (type: cam)"`。
4.  如果没有选中节点，则返回 `"当前没有选中任何节点。"`。
5.  此方法返回的字符串将作为额外信息附加到用户 Prompt 中，以提供上下文。”

---

#### **阶段四：代码执行与反馈**

**目标**: 安全地执行 AI 返回的代码，并将结果反馈给用户。

**`AI 指令 8: 实现代码执行函数 (main.py)`**
“在 `HoudiniCopilotPanel` 类中，创建一个名为 `execute_code` 的方法。
1.  该方法接收一个字符串参数 `code_to_execute`。
2.  在执行代码前，将其包裹在 Houdini 的 undo 块中，即 `with hou.undos.group("AI Generated Action"):`。这样用户可以一键撤销 AI 的所有操作。
3.  使用 `exec(code_to_execute, {'hou': hou})` 来执行代码。将 `hou` 模块显式传入执行的全局命名空间中。
4.  使用一个大的 `try-except` 块来捕获执行过程中可能出现的任何异常（如 `hou.OperationFailed` 或 `SyntaxError`）。
5.  无论成功还是失败，都将结果（成功信息或完整的错误回溯）追加显示到 `self.log_output` 中。”

---

#### **阶段五：整合与连接**

**目标**: 将所有部分连接起来，使整个工具链可以工作。

**`AI 指令 9: 连接 UI 信号与槽 (main.py)`**
“在 `HoudiniCopilotPanel` 类的 `__init__` 方法中，连接 "生成并执行" 按钮的 `clicked` 信号到一个新的主逻辑方法，命名为 `on_generate_button_clicked`。”

**`AI 指令 10: 编写主逻辑方法 (main.py)`**
“创建 `on_generate_button_clicked` 方法。此方法应按以下顺序执行操作：
1.  从 `self.prompt_input` 获取用户输入的文本。如果为空，则直接返回。
2.  清空 `self.log_output`。
3.  调用 `self.get_scene_context()` 获取场景上下文信息。
4.  将用户输入和场景上下文组合成一个完整的 Prompt。例如：`f"{user_input}\n\n上下文：{scene_context}"`。
5.  将这个完整的 Prompt 传递给 `self.query_ai()` 方法，并等待返回的 Python 代码。
6.  在日志中显示 AI 返回的原始代码。
7.  将返回的代码传递给 `self.execute_code()` 方法执行。”

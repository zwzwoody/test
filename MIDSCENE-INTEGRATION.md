# Midscene AI 集成使用说明

## 📖 概述

本项目已集成 Midscene AI 测试能力，可以在 Selenium 测试失败时自动使用 AI 重试，提高测试的稳定性和成功率。

## 🎯 主要功能

1. **自动失败重试**：当 Selenium 测试失败时，自动使用 AI 重新执行测试
2. **AI 驱动测试**：直接使用自然语言描述测试步骤，由 AI 执行
3. **混合测试模式**：Selenium 和 AI 可以混合使用，发挥各自优势
4. **Jenkins 集成**：完整支持 Jenkins 流水线，自动管理 Midscene 服务

## 🏗️ 架构说明

```
Jenkins Pipeline
    ↓
启动 Midscene Bridge 服务 (Node.js)
    ↓
启动 Pytest
    ↓
初始化 Selenium Grid 浏览器（带 CDP 端口）
    ↓
执行测试用例
    ├─ Selenium 成功 → 测试通过 ✓
    └─ Selenium 失败 → AI 重试
        ├─ AI 成功 → 测试通过 ✓
        └─ AI 失败 → 测试失败 ✗
```

## 📦 文件结构

```
xiaoe_live_webui/
├── common/
│   ├── midscene_service.py      # Midscene 服务管理器
│   └── midscene_helper.py       # Midscene AI 辅助类
├── test_case/
│   ├── test_midscene/
│   │   └── midscene-bridge/     # Node.js 服务
│   │       └── server.js
│   └── test_scenes/
│       └── test_midscene_example.py  # 示例测试用例
├── conftest.py                  # Pytest 配置（已集成 Midscene）
├── start_midscene_service.sh    # Linux/Mac 启动脚本
├── start_midscene_service.bat   # Windows 启动脚本
├── Jenkinsfile.midscene         # Jenkins 流水线配置
└── yml/
    └── gray.yml                 # 配置文件（已添加 Midscene 配置）
```

## 🚀 快速开始

### 1. 本地测试

#### 方式一：自动启动（推荐）

```bash
# 启用 Midscene，服务会自动启动
pytest test_case/test_scenes/test_midscene_example.py --enable-midscene -v
```

#### 方式二：手动启动服务

```bash
# Linux/Mac
./start_midscene_service.sh start

# Windows
start_midscene_service.bat start

# 运行测试
pytest test_case/test_scenes/test_midscene_example.py --enable-midscene -v

# 停止服务
./start_midscene_service.sh stop  # Linux/Mac
start_midscene_service.bat stop   # Windows
```

### 2. Jenkins 流水线

#### 创建 Jenkins Job

1. 在 Jenkins 中创建新的 Pipeline Job
2. 选择 "Pipeline script from SCM"
3. 配置 Git 仓库
4. 指定 Script Path 为 `Jenkinsfile.midscene`
5. 保存并运行

#### 环境变量配置

在 Jenkins Job 中可以配置以下环境变量：

```groovy
environment {
    ENVIRONMENT = 'gray'           // 测试环境
    ENABLE_MIDSCENE = 'true'       // 启用 Midscene
    MIDSCENE_PORT = '3000'         // Midscene 服务端口
    CDP_PORT = '9222'              // Chrome CDP 端口
}
```

## 📝 编写测试用例

### 方式一：使用 AI 任务标记（失败自动重试）

```python
import pytest
import allure
from conftest import driver

@pytest.mark.ai_task("访问小鹅通官网，点击'产品服务'，验证是否有'小鹅云'内容")
def test_with_ai_retry():
    """如果 Selenium 失败，AI 会自动重试"""
    driver.get("https://www.xiaoe-tech.com/")
    
    # Selenium 操作
    element = driver.find_element(By.LINK_TEXT, "产品服务")
    element.click()
    
    # 验证
    assert "小鹅云" in driver.page_source
```

**关键点**：
- 添加 `@pytest.mark.ai_task("任务描述")` 装饰器
- 任务描述要清晰、完整，包含所有操作步骤
- 如果 Selenium 失败，AI 会根据任务描述自动执行

### 方式二：直接使用 Midscene 辅助类

```python
def test_use_midscene_directly(midscene):
    """直接使用 AI 功能"""
    if midscene is None:
        pytest.skip("Midscene 未启用")
    
    driver.get("https://www.xiaoe-tech.com/")
    
    # AI 执行操作
    result = midscene.ai_do("点击'产品服务'菜单")
    assert result.get("success")
    
    # AI 断言
    result = midscene.ai_assert("页面是否显示'小鹅云'内容")
    assert result
    
    # AI 自动规划（推荐）
    result = midscene.ai_plan("点击'产品服务'，然后验证是否有'小鹅云'")
    assert result.get("success")
```

### 方式三：混合使用 Selenium 和 AI

```python
def test_hybrid(midscene):
    """混合使用 Selenium 和 AI"""
    # 简单操作用 Selenium
    driver.get("https://www.xiaoe-tech.com/")
    
    # 复杂交互用 AI
    if midscene:
        result = midscene.ai_do("找到并点击'免费试用'按钮")
        assert result.get("success")
    
    # 简单验证用 Selenium
    assert "小鹅通" in driver.title
```

## 🔧 配置说明

### Pytest 命令行参数

```bash
--enable-midscene          # 启用 Midscene AI 功能
--midscene-port 3000       # Midscene 服务端口（默认 3000）
--cdp-port 9222            # Chrome CDP 端口（默认 9222）
```

### 配置文件（yml/gray.yml）

```yaml
# Midscene AI 配置
midscene_enabled: true      # 是否启用 Midscene
midscene_port: 3000         # 服务端口
cdp_port: 9222              # CDP 端口
```

## 🎨 AI 操作类型

### 1. ai_do - 执行操作

```python
midscene.ai_do("点击登录按钮")
midscene.ai_do("在搜索框输入'测试'")
midscene.ai_do("滚动到页面底部")
```

### 2. ai_query - 查询信息

```python
result = midscene.ai_query("页面上显示的用户名是什么？")
result = midscene.ai_query("有多少个菜单项？")
```

### 3. ai_assert - 断言验证

```python
is_valid = midscene.ai_assert("页面是否显示'登录成功'")
is_valid = midscene.ai_assert("是否有错误提示")
```

### 4. ai_plan - 自动规划（推荐）

```python
# AI 会自动分解任务并执行
result = midscene.ai_plan("""
    1. 点击登录按钮
    2. 输入用户名 'test@example.com'
    3. 输入密码 'password123'
    4. 点击提交
    5. 验证是否登录成功
""")
```

## 📊 Allure 报告

启用 Midscene 后，Allure 报告会显示：

- **AI-Retry-Success** 标签：AI 重试成功的用例
- **AI-Retry-Failed** 标签：AI 重试失败的用例
- **AI 重试说明**：详细的重试信息

## 🐛 故障排查

### 1. Midscene 服务启动失败

**检查 Node.js**：
```bash
node --version  # 需要 Node.js 14+
```

**检查端口占用**：
```bash
# Linux/Mac
lsof -i :3000

# Windows
netstat -ano | findstr :3000
```

**查看服务日志**：
```bash
# Linux/Mac
tail -f /tmp/midscene-bridge.log

# Windows
type %TEMP%\midscene-bridge.log
```

### 2. AI 操作失败

**检查 CDP 连接**：
- 确保浏览器启动时添加了 `--remote-debugging-port=9222`
- 确保端口未被占用
- 确保 Midscene 服务可以访问 CDP 端口

**检查 AI 配置**：
- 查看 `test_case/test_midscene/midscene-bridge/server.js`
- 确认 AI 模型配置正确（API Key、Base URL）

### 3. Selenium Grid 兼容性

**CDP 端口配置**：
- 远程 Selenium Grid 需要支持 CDP 端口映射
- 确保网络可以访问 CDP 端口

## 💡 最佳实践

### 1. 何时使用 AI 重试

✅ **适合使用 AI 重试的场景**：
- 元素定位不稳定
- 页面结构经常变化
- 复杂的交互流程
- 动态加载的内容

❌ **不适合使用 AI 重试的场景**：
- 简单的页面跳转
- 固定的表单填写
- 性能测试
- 需要精确控制的操作

### 2. 编写 AI 任务描述

**好的描述**：
```python
@pytest.mark.ai_task("""
访问登录页面，输入用户名 'admin@example.com'，
输入密码 'password123'，点击登录按钮，
然后验证页面是否显示'欢迎回来'的提示
""")
```

**不好的描述**：
```python
@pytest.mark.ai_task("登录")  # 太简单，缺少细节
```

### 3. 性能考虑

- AI 操作比 Selenium 慢（通常需要 5-15 秒）
- 只在必要时启用 Midscene（使用 `--enable-midscene`）
- 优先使用 Selenium，AI 作为备用方案

## 📚 参考资料

- [Midscene 官方文档](https://midscenejs.com/)
- [Playwright CDP 文档](https://playwright.dev/docs/api/class-cdpsession)
- [Selenium Grid 文档](https://www.selenium.dev/documentation/grid/)

## 🤝 贡献

如有问题或建议，请联系测试团队。

## 📄 许可

内部项目，仅供公司内部使用。

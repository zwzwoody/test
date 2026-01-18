# README 更新建议

## 建议在主 README.md 中添加以下内容：

---

## 🤖 Midscene AI 测试集成

本项目已集成 Midscene AI 测试能力，支持在 Selenium 测试失败时自动使用 AI 重试。

### 快速开始

```bash
# 启用 Midscene AI 功能运行测试
pytest --enable-midscene -v

# 查看示例
pytest test_case/test_scenes/test_midscene_example.py --enable-midscene -v
```

### 主要特性

- ✅ **自动失败重试**：Selenium 失败时 AI 自动接管
- ✅ **自然语言测试**：用自然语言描述测试步骤
- ✅ **混合测试模式**：Selenium + AI 混合使用
- ✅ **Jenkins 集成**：完整支持流水线自动化

### 文档

- 📖 [完整集成文档](MIDSCENE_INTEGRATION.md)
- 🚀 [快速入门指南](QUICK_START_MIDSCENE.md)
- 💡 [示例测试用例](test_case/test_scenes/test_midscene_example.py)

### Jenkins 使用

```groovy
// 在 Jenkinsfile 中启用 Midscene
environment {
    ENABLE_MIDSCENE = 'true'
}

// 或使用提供的 Jenkinsfile
// Script Path: Jenkinsfile.midscene
```

---

## 建议的目录结构说明更新：

```
xiaoe_live_webui/
├── common/
│   ├── midscene_service.py      # Midscene 服务管理
│   ├── midscene_helper.py       # AI 测试辅助类
│   └── ...
├── test_case/
│   ├── test_midscene/
│   │   ├── midscene-bridge/     # Node.js AI 服务
│   │   └── ...
│   └── test_scenes/
│       ├── test_midscene_example.py  # AI 测试示例
│       └── ...
├── start_midscene_service.sh    # Midscene 服务启动脚本（Linux/Mac）
├── start_midscene_service.bat   # Midscene 服务启动脚本（Windows）
├── Jenkinsfile.midscene         # Jenkins 流水线配置（含 Midscene）
├── MIDSCENE_INTEGRATION.md      # Midscene 集成完整文档
├── QUICK_START_MIDSCENE.md      # Midscene 快速入门
└── ...
```

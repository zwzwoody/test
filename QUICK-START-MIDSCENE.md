Midscene AI 快速入门指南
🚀 5 分钟快速上手
步骤 1：安装依赖
# 安装 Python 依赖（如果还没安装）
pip install -r requirements.txt

# 安装 Node.js 依赖
cd test_case/test_midscene/midscene-bridge
npm install
cd ../../..
​
步骤 2：运行示例测试
# 方式一：自动启动 Midscene 服务（推荐）
pytest test_case/test_scenes/test_midscene_example.py::TestMidsceneExample::test_xiaoe_navigation_with_ai_retry --enable-midscene -v -s

# 方式二：手动启动服务
# 终端 1：启动 Midscene 服务
./start_midscene_service.sh start  # Linux/Mac
# 或
start_midscene_service.bat start   # Windows

# 终端 2：运行测试
pytest test_case/test_scenes/test_midscene_example.py --enable-midscene -v -s
​
步骤 3：查看测试结果
# 生成 Allure 报告
allure serve allure-results
​
📝 编写你的第一个 AI 测试
示例 1：失败自动重试
import pytest
from conftest import driver

@pytest.mark.ai_task("访问小鹅通官网，点击'产品服务'菜单")
def test_my_first_ai_test():
    driver.get("https://www.xiaoe-tech.com/")
    
    # 如果下面的操作失败，AI 会自动接管
    from selenium.webdriver.common.by import By
    element = driver.find_element(By.LINK_TEXT, "产品服务")
    element.click()
​
示例 2：直接使用 AI
def test_use_ai_directly(midscene):
    if not midscene:
        pytest.skip("Midscene 未启用")
    
    driver.get("https://www.xiaoe-tech.com/")
    
    # 使用 AI 自动规划
    result = midscene.ai_plan("点击'产品服务'，验证是否有'小鹅云'")
    assert result.get("success")
​
🎯 在 Jenkins 中使用
方式一：使用提供的 Jenkinsfile
# 在 Jenkins 中创建 Pipeline Job
# Script Path: Jenkinsfile.midscene
​
方式二：在现有 Jenkinsfile 中添加
stage('启动 Midscene 服务') {
    steps {
        sh './start_midscene_service.sh start'
    }
}

stage('运行测试') {
    steps {
        sh '''
            export environment=gray
            pytest --enable-midscene --driver_type=web -m P0
        '''
    }
}

post {
    always {
        sh './start_midscene_service.sh stop || true'
    }
}
​
⚙️ 常用命令
# 启动 Midscene 服务
./start_midscene_service.sh start

# 停止 Midscene 服务
./start_midscene_service.sh stop

# 查看服务状态
./start_midscene_service.sh status

# 重启服务
./start_midscene_service.sh restart

# 运行测试（启用 Midscene）
pytest --enable-midscene -v

# 运行测试（不启用 Midscene）
pytest -v

# 运行特定测试
pytest test_case/test_scenes/test_midscene_example.py --enable-midscene -v
​
🐛 常见问题
Q1: Midscene 服务启动失败
解决方案：

# 检查 Node.js 版本
node --version  # 需要 14+

# 检查端口占用
lsof -i :3000  # Linux/Mac
netstat -ano | findstr :3000  # Windows

# 查看日志
tail -f /tmp/midscene-bridge.log  # Linux/Mac
type %TEMP%\midscene-bridge.log  # Windows
​
Q2: AI 操作失败
解决方案：

检查 AI 模型配置（test_case/test_midscene/midscene-bridge/server.js）
确保网络可以访问 AI 服务
查看 Midscene 服务日志
Q3: CDP 连接失败
解决方案：

确保浏览器启动时添加了 --remote-debugging-port=9222
检查端口是否被占用
如果使用远程 Selenium Grid，确保网络可以访问 CDP 端口
📚 下一步
阅读完整文档：MIDSCENE_INTEGRATION.md
查看示例代码：test_midscene_example.py
了解 Midscene 官方文档：https://midscenejs.com/
💬 获取帮助
如有问题，请联系测试团队或查看项目文档。
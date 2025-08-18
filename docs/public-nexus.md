基于代码分析，要将项目发布到内部 nexus npm 仓库并使用新名字 @hero-tm/mobile-mcp，需要调整以下内容：

必须调整的文件和配置

1. package.json

- name: "@mobilenext/mobile-mcp" → "@hero-tm/mobile-mcp"
- repository.url: 更新为内部 Git 仓库地址
- bugs.url: 更新为内部问题跟踪系统
- homepage: 更新为内部项目主页

2. README.md

- 安装命令:
	- "npx -y @mobilenext/mobile-mcp@latest" → "npx -y @hero-tm/mobile-mcp@latest"
	- MCP 配置示例中的包名引用
- GitHub 链接: 所有指向 github.com/mobile-next/mobile-mcp 的链接
- npm 徽章: npm 包下载量等徽章链接
- 文档链接: wiki 页面和其他外部文档链接

3. CLAUDE.md

- 项目描述: 更新包名引用
- 安装说明: 更新相关的安装命令示例

需要新增的配置

4. .npmrc (新建)

registry=http://your-nexus-server/repository/npm-group/
@hero-tm:registry=http://your-nexus-server/repository/npm-internal/

5. 发布配置

- CI/CD 配置: 更新发布流程指向内部 nexus
- 认证配置: 配置 nexus 认证信息

可选调整

6. LICENSE

- 检查是否需要更新版权信息

7. CHANGELOG.md

- 更新发布链接指向内部仓库

8. 源码注释

- server.ts 中的 GitHub API 调用 (getLatestAgentVersion 函数)
- 可能需要调整版本检查逻辑指向内部仓库

发布流程调整

1. 认证: 配置 nexus 仓库认证
2. 发布命令: npm publish 将推送到配置的内部仓库
3. 版本管理: 确保语义化版本控制策略
4. 测试: 验证从内部仓库安装和使用

这样调整后，用户就可以通过 npm install @hero-tm/mobile-mcp 从内部 nexus 仓库安装使用了。


基于内部 nexus npm 仓库发布的完整命令：



发布前准备

1. 配置 nexus 认证

# 方式1: 使用 npm login
npm login --registry=http://your-nexus-server/repository/npm-internal/

# 方式2: 直接配置 .npmrc (推荐)
npm config set @hero-tm:registry http://your-nexus-server/repository/npm-internal/
npm config set //your-nexus-server/repository/npm-internal/:_authToken=your-token

2. 验证配置

# 检查当前配置
npm config list
npm whoami --registry=http://your-nexus-server/repository/npm-internal/

发布命令序列

3. 构建和测试

# 清理并构建
npm run clean
npm run build

# 运行测试
npm run lint
npm test

4. 版本管理

# 更新版本号 (根据需要选择)
npm version patch   # 0.0.1 -> 0.0.2
npm version minor   # 0.0.1 -> 0.1.0
npm version major   # 0.0.1 -> 1.0.0

# 或手动指定版本
npm version 1.0.0

5. 发布到内部仓库

# 发布到指定仓库
npm publish --registry=http://your-nexus-server/repository/npm-internal/

# 如果已配置 .npmrc，直接发布
npm publish

6. 验证发布

# 验证包是否成功发布
npm view @hero-tm/mobile-mcp --registry=http://your-nexus-server/repository/npm-internal/

# 测试安装
npm install @hero-tm/mobile-mcp --registry=http://your-nexus-server/repository/npm-internal/

完整发布脚本示例

#!/bin/bash
# publish-to-nexus.sh

set -e

echo "🚀 开始发布到内部 Nexus 仓库..."

# 1. 清理和构建
echo "📦 清理和构建..."
npm run clean
npm run build

# 2. 运行质量检查
echo "🔍 运行质量检查..."
npm run lint
npm test

# 3. 更新版本
echo "📈 更新版本..."
read -p "请选择版本类型 (patch/minor/major): " version_type
npm version $version_type

# 4. 发布
echo "🚢 发布到 Nexus..."
npm publish --registry=http://your-nexus-server/repository/npm-internal/

# 5. 验证
echo "✅ 验证发布..."
PACKAGE_VERSION=$(node -p "require('./package.json').version")
npm view @hero-tm/mobile-mcp@$PACKAGE_VERSION --registry=http://your-nexus-server/repository/npm-internal/

echo "🎉 发布成功! 版本: $PACKAGE_VERSION"

添加到 package.json scripts

在 package.json 中添加发布相关脚本：

{
"scripts": {
"publish:nexus": "npm publish --registry=http://your-nexus-server/repository/npm-internal/",
"publish:check": "npm view @hero-tm/mobile-mcp --registry=http://your-nexus-server/repository/npm-internal/",
"prepublishOnly": "npm run clean && npm run build && npm run lint && npm test"
}
}

使用发布的包

用户安装和使用：

# 安装
npm install @hero-tm/mobile-mcp --registry=http://your-nexus-server/repository/npm-internal/

# 或配置 .npmrc 后直接安装
npm install @hero-tm/mobile-mcp

# 全局安装
npm install -g @hero-tm/mobile-mcp --registry=http://your-nexus-server/repository/npm-internal/

# MCP 配置中使用
npx -y @hero-tm/mobile-mcp@latest --registry=http://your-nexus-server/repository/npm-internal/

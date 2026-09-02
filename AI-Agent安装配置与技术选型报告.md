# AI Agent 安装配置与技术选型报告

**报告日期**：2026年9月2日  
**实验环境**：Windows 11 + PowerShell + Node.js v26.7.0  
**目标仓库**：jotang-recruit-2026-git

---

## 一、技术选型与决策过程

### 1.1 初始选型：Gemini CLI

根据题目要求，初步选定 Google 官方出品的 Gemini CLI 作为实验对象。选型依据如下：
- 官方工具，文档完善
- 具备文件读写、命令执行等 Agent 核心能力
- 符合作业对 Coding Agent 的定义要求

### 1.2 选型调整：Gemini CLI 服务状态确认失败

在安装配置过程中，经官方文档及终端提示确认，**Gemini CLI 对个人免费用户的服务已于 2026 年 6 月 18 日正式终止**，官方要求迁移至 Antigravity 套件。该服务状态的变更导致原定技术路线不可行，需重新选型。

### 1.3 最终选型：DeepSeek Harness

重新评估后选择 DeepSeek Harness，决策依据如下：

| 评估维度 | 说明 |
|---------|------|
| 网络可达性 | 国内可直接访问，无需代理工具 |
| 认证方式 | 手机号注册，无需 Google 账号 |
| 交互方式 | 提供 Web 图形界面，降低使用门槛 |
| 功能完整性 | 支持文件读写、命令执行、项目级规则配置 |
| 部署方式 | 基于 Node.js 的 npm 全局安装，与现有环境兼容 |

---

## 二、环境准备与依赖安装

### 2.1 Node.js 运行环境配置

系统初始未安装 Node.js 运行时，执行以下操作完成环境搭建：

```powershell
# 通过 Windows 包管理器安装 Node.js
winget install OpenJS.NodeJS
```

安装完成后，PowerShell 默认执行策略阻止了 npm 脚本的运行，通过修改当前用户作用域的执行策略解决：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

验证环境：
```powershell
node -v   # v26.7.0
npm -v    # 正常显示版本号
```

### 2.2 全局包管理器配置

为加速后续依赖下载，配置 npm 使用国内镜像源：

```powershell
npm config set registry https://registry.npmmirror.com
```

---

## 三、Agent 安装过程

### 3.1 Gemini CLI 安装（已废弃）

```powershell
npm install -g @google/gemini-cli
```

安装完成后，在执行认证流程时确认该服务已终止，未继续进行后续配置。

### 3.2 DeepSeek Harness 安装

#### 初次安装尝试（使用 cnpm，失败）

为提高安装速度，使用 `cnpm` 进行全局安装：

```powershell
cnpm install -g @deepseek-ai/dsh
```

安装完成后执行 `dsh web` 启动，系统报告大量 `ERR_MODULE_NOT_FOUND` 错误，涉及 `@deepseek-ai/cordis-plugin-group`、`@deepseek-ai/dsh-client-modules` 等多个内部依赖包。

**失败原因分析**：`cnpm` 为提升安装速度对 `node_modules` 目录结构进行了扁平化处理，改变了依赖树的组织方式，与 DeepSeek Harness 基于标准 npm 依赖解析机制的预期不兼容。

#### 重新安装（使用 npm，成功）

执行清理并重新安装：

```powershell
# 卸载现有版本
npm uninstall -g @deepseek-ai/dsh

# 清理缓存
npm cache clean --force

# 使用 npm + 淘宝镜像重新安装
npm install -g @deepseek-ai/dsh --registry=https://registry.npmmirror.com
```

安装输出：
```
added 452 packages in 2m
```

验证安装：
```powershell
dsh --version   # 0.1.1-rc.2
```

---

## 四、启动验证与工作区配置

### 4.1 启动 Web 服务

```powershell
dsh web
```

服务启动后，终端输出本地访问地址：
```
Web UI running at http://127.0.0.1:3080
```

### 4.2 定位本地 Git 仓库

通过 PowerShell 命令定位目标仓库的本地路径：

```powershell
Get-ChildItem -Path C:\Users\成语 -Filter *jotang-recruit-2026-git* -Directory -Recurse -ErrorAction SilentlyContinue
```

定位结果：
```
C:\Users\成语\jotang-recruit-2026-git
```

### 4.3 配置工作区

在 DeepSeek Harness Web 界面中选择上述路径作为工作区，Agent 获得对该目录下所有文件的读取和修改权限。

---

## 五、技术总结

### 5.1 关键问题与解决方案

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| npm 命令不可用 | 未安装 Node.js | 通过 winget 安装 Node.js |
| npm 脚本被阻止 | PowerShell 执行策略限制 | 修改为 RemoteSigned |
| Gemini CLI 无法使用 | 官方已于 2026-06-18 停服 | 更换为 DeepSeek Harness |
| DeepSeek Harness 启动报错 | cnpm 改变了依赖目录结构 | 改用 npm 重新安装 |

### 5.2 选型建议

1. **优先确认服务状态**：在投入安装时间前，应首先查阅官方公告确认服务可用性，避免在已终止服务的工具上投入时间。

2. **依赖管理工具的选择**：`cnpm` 适用于对依赖结构不敏感的简单项目；对于依赖关系复杂、需要严格遵循 npm 规范的工具，应使用标准的 `npm` 命令。

3. **国内环境下的工具选型**：优先选择国产或国内镜像支持良好的工具，可有效降低网络因素导致的安装失败率。

---


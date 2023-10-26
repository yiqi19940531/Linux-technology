## node.js及npm版本管理工具nvm安装及使用
`node` (Node.js) 和 `npm` 之间存在一个推荐的版本对应关系。当你安装一个特定版本的 Node.js 时，它通常会自带一个与之兼容的 `npm` 版本。这确保了在那个特定的 Node.js 版本中，你使用的 `npm` 能够正常工作。
如果你想知道特定版本的 Node.js 推荐使用哪个版本的 npm，可以查看 Node.js 的 [官方发行说明](https://nodejs.org/en/download/releases#releases)。
### nvm版本管理工具
在 Linux（以及其他操作系统）中，用于管理多个 npm 和 Node.js 版本的工具是 nvm，即 Node Version Manager。通过使用 nvm，你可以轻松地在不同的 Node.js 版本之间切换，这对于开发、测试以及运行依赖于特定 Node.js 版本的应用程序非常有用。
在 Linux（以及其他操作系统）中，用于管理多个 `npm` 和 `Node.js` 版本的工具是 `nvm`，即 **Node Version Manager**。通过使用 `nvm`，你可以轻松地在不同的 Node.js 版本之间切换，这对于开发、测试以及运行依赖于特定 Node.js 版本的应用程序非常有用。

以下是如何在 Linux 上安装和使用 `nvm` 的基本步骤：

1. **安装 nvm**:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
```

注意：建议访问 [nvm 的 GitHub 仓库](https://github.com/nvm-sh/nvm) 来获取最新的安装指令。

2. **如果需要再当前Terminal运行nvm工具请执行以下命令**:

```bash
# Close and reopen your terminal to start using nvm or run the following to use it now
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

3. **安装特定版本的 Node.js**:

```bash
nvm install 14      # 安装 Node.js v14 的最新版本
nvm install 12.18.3 # 安装 Node.js v12.18.3
```


4. **切换 Node.js 版本**:

```bash
nvm use 14          # 使用 Node.js v14 的最新版本
nvm use 12.18.3     # 使用 Node.js v12.18.3
```

5. **列出已安装的版本**:

```bash
nvm ls
```

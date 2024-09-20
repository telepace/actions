# Github Util Actions


可以去 web actions 上操作，也可以使用 API 或者使用 GITHUB CLI 操作，推荐用 cli 工具

以下是如何调用你提供的 Go 构建工作流的示例，以及各个字段和参数的详细说明。我们将介绍通过 GitHub 的 `workflow_dispatch` 事件手动触发工作流的两种主要方法：

1. **通过 GitHub 网站手动触发**
2. **通过 GitHub CLI (`gh` 命令行工具) 触发**
3. **通过 GitHub API 触发**

### 1. 通过 GitHub 网站手动触发

如果你的工作流配置了 `workflow_dispatch` 事件，你可以直接在 GitHub 仓库的 "Actions" 标签页中手动触发该工作流。

**步骤：**

1. 导航到你的 GitHub 仓库。
2. 点击顶部导航栏的 **Actions** 标签。
3. 在左侧面板中，选择你要触发的工作流（例如，`go构建并下载`）。
4. 点击右侧的 **Run workflow** 按钮。
5. 在弹出的表单中，填写所需的输入参数。
6. 点击 **Run workflow** 开始执行。

**输入参数示例：**

![Run Workflow](https://docs.github.com/assets/images/help/repository/run-workflow-button.png)

### 2. 通过 GitHub CLI (`gh` 命令行工具) 触发

GitHub CLI 是一个强大的工具，可以让你通过命令行与 GitHub 交互，包括触发工作流。以下是使用 GitHub CLI 触发 Go 构建工作流的示例。

**前提条件：**

- 安装并配置 [GitHub CLI](https://cli.github.com/)。
- 确保你拥有触发工作流的权限。

**示例命令：**

```bash
gh workflow run "go构建并下载" \
  --ref main \
  --field repoPath="https://github.com/yourusername/your-go-repo.git" \
  --field repoBranch="main" \
  --field shellUrl="https://raw.githubusercontent.com/yourusername/your-scripts/main/setup.sh" \
  --field shell="cd /path/to/project && go mod tidy" \
  --field GO_VERSION="1.20" \
  --field UPLOAD_RELEASE=true \
  --field UPLOAD_TRANSFER="https://transfer-service.example.com/upload" \
  --field UPLOAD_TAG="v1.0.0" \
  --field platforms="linux/amd64,linux/arm64,windows/amd64"
```

**参数说明：**

- `--ref main`：指定要运行工作流的 Git 分支或标签，这里为 `main` 分支。
- `--field repoPath`：Git 仓库的路径（URL），例如 `https://github.com/yourusername/your-go-repo.git`。
- `--field repoBranch`：指定要克隆的分支，如果不指定，则默认使用仓库的默认分支。
- `--field shellUrl`：构建前运行的脚本的 URL，可以是任何可下载的脚本文件，例如初始化环境或安装依赖。
- `--field shell`：构建前运行的 shell 命令，例如切换目录 (`cd`) 或执行 `go mod tidy` 等命令。
- `--field GO_VERSION`：指定要使用的 Go 版本，例如 `1.20`。默认值为 `stable`。
- `--field UPLOAD_RELEASE`：是否将构建产物上传到 GitHub Releases，布尔值 `true` 或 `false`。
- `--field UPLOAD_TRANSFER`：上传服务的 URL，例如 `https://transfer-service.example.com/upload`，用于将构建产物上传到中转服务。
- `--field UPLOAD_TAG`：上传文件使用的标签（TAG），例如 `v1.0.0`。如果不提供，将默认使用构建时间作为 TAG。
- `--field platforms`：需要构建的平台，多个平台用逗号分隔，例如 `linux/amd64,linux/arm64,windows/amd64`。

### 3. 通过 GitHub API 触发

你还可以通过 GitHub 的 REST API 触发 `workflow_dispatch` 事件。这对于自动化脚本或集成第三方服务非常有用。

**示例请求：**

```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer YOUR_GITHUB_TOKEN" \
  https://api.github.com/repos/yourusername/your-go-repo/actions/workflows/go_build_and_download.yml/dispatches \
  -d '{
    "ref": "main",
    "inputs": {
      "repoPath": "https://github.com/yourusername/your-go-repo.git",
      "repoBranch": "main",
      "shellUrl": "https://raw.githubusercontent.com/yourusername/your-scripts/main/setup.sh",
      "shell": "cd /path/to/project && go mod tidy",
      "GO_VERSION": "1.20",
      "UPLOAD_RELEASE": true,
      "UPLOAD_TRANSFER": "https://transfer-service.example.com/upload",
      "UPLOAD_TAG": "v1.0.0",
      "platforms": "linux/amd64,linux/arm64,windows/amd64"
    }
  }'
```

**参数说明：**

- `YOUR_GITHUB_TOKEN`：替换为你的 GitHub 个人访问令牌，确保该令牌具有触发工作流的权限。
- `ref`：指定要运行工作流的 Git 分支或标签，这里为 `main` 分支。
- `inputs`：工作流所需的所有输入参数，具体如下：

  - `"repoPath"`：Git 仓库的路径（URL）。
  - `"repoBranch"`：指定要克隆的分支。
  - `"shellUrl"`：构建前运行的脚本的 URL。
  - `"shell"`：构建前运行的 shell 命令。
  - `"GO_VERSION"`：指定要使用的 Go 版本。
  - `"UPLOAD_RELEASE"`：是否将构建产物上传到 GitHub Releases。
  - `"UPLOAD_TRANSFER"`：上传服务的 URL。
  - `"UPLOAD_TAG"`：上传文件使用的标签（TAG）。
  - `"platforms"`：需要构建的平台，多个平台用逗号分隔。

**注意事项：**

- 确保你的 GitHub 令牌具有 `repo` 和 `workflow` 权限，以便能够触发工作流。
- `workflow_dispatch` 事件需要在工作流文件中正确配置。

### 各个字段和参数详细说明

以下是工作流输入参数的详细说明，帮助你更好地理解和配置每个字段：

| 参数名            | 描述                                                         | 是否必需 | 默认值                    | 类型      |
|-------------------|--------------------------------------------------------------|----------|---------------------------|-----------|
| `repoPath`        | Git 仓库的路径（URL），例如 `https://github.com/yourusername/your-go-repo.git` | 是       | 无                        | 字符串    |
| `repoBranch`      | 指定要克隆的分支，如果不指定，则使用仓库的默认分支              | 否       | 无                        | 字符串    |
| `shellUrl`        | 构建前运行的脚本的 URL，可以是任何可下载的脚本文件             | 否       | 无                        | 字符串    |
| `shell`           | 构建前运行的 shell 命令，例如 `cd` 或 `go mod tidy` 等          | 否       | 无                        | 字符串    |
| `GO_VERSION`      | 指定要使用的 Go 版本，例如 `1.20`，默认值为 `stable`            | 否       | `stable`                  | 字符串    |
| `UPLOAD_RELEASE`  | 是否将构建产物上传到 GitHub Releases，布尔值 `true` 或 `false` | 否       | `false`                   | 布尔值    |
| `UPLOAD_TRANSFER` | 上传服务的 URL，例如 `https://transfer-service.example.com/upload` | 否       | 无                        | 字符串    |
| `UPLOAD_TAG`      | 上传文件使用的标签（TAG），例如 `v1.0.0`，默认使用构建时间     | 否       | 使用构建时间              | 字符串    |
| `platforms`       | 需要构建的平台，多个平台用逗号分隔，例如 `linux/amd64,linux/arm64,windows/amd64` | 是       | `linux/amd64,linux/arm64` | 字符串    |

### 工作流参数使用示例

假设你有一个 Go 项目，你希望在 `main` 分支上触发构建，使用 Go 1.20 版本，并将构建产物上传到 GitHub Releases 和一个中转服务，标签为 `v1.0.0`，构建平台为 `linux/amd64` 和 `windows/amd64`。

**使用 GitHub CLI 触发：**

```bash
gh workflow run "go构建并下载" \
  --ref main \
  --field repoPath="https://github.com/yourusername/your-go-repo.git" \
  --field repoBranch="main" \
  --field shellUrl="https://raw.githubusercontent.com/yourusername/your-scripts/main/setup.sh" \
  --field shell="cd /path/to/project && go mod tidy" \
  --field GO_VERSION="1.20" \
  --field UPLOAD_RELEASE=true \
  --field UPLOAD_TRANSFER="https://transfer-service.example.com/upload" \
  --field UPLOAD_TAG="v1.0.0" \
  --field platforms="linux/amd64,windows/amd64"
```

**使用 GitHub API 触发：**

```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer YOUR_GITHUB_TOKEN" \
  https://api.github.com/repos/yourusername/your-go-repo/actions/workflows/go_build_and_download.yml/dispatches \
  -d '{
    "ref": "main",
    "inputs": {
      "repoPath": "https://github.com/yourusername/your-go-repo.git",
      "repoBranch": "main",
      "shellUrl": "https://raw.githubusercontent.com/yourusername/your-scripts/main/setup.sh",
      "shell": "cd /path/to/project && go mod tidy",
      "GO_VERSION": "1.20",
      "UPLOAD_RELEASE": true,
      "UPLOAD_TRANSFER": "https://transfer-service.example.com/upload",
      "UPLOAD_TAG": "v1.0.0",
      "platforms": "linux/amd64,windows/amd64"
    }
  }'
```


## Go




## Python

examples:

```json
{
  "event_type": "python_build",
  "client_payload": {
    "repoPath": "https://github.com/yourusername/your-python-repo.git",
    "repoBranch": "main",
    "PYTHON_VERSION": "3.11",
    "UPLOAD_RELEASE": true,
    "UPLOAD_TRANSFER": "your-transfer-service-url",
    "platforms": "win-x64,linux-x64,linux-arm64"
  }
}
```

## 私有仓库

git地址写法

```
https://oauth2:<token>@github.com/name/repo
```

其它git服务

git地址写法

```
https://<username>:<password>@<url>
```
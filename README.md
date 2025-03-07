# Override-PUSH

## 这个仓库什么也不能做，请不要盯着我。

### VSCode 配置：

```json
    "github.copilot.advanced": {
        "debug.overrideCAPIUrl": "http://127.0.0.1:8181/v1",
        "debug.overrideProxyUrl": "http://127.0.0.1:8181",
        "debug.chatOverrideProxyUrl": "http://127.0.0.1:8181/v1/chat/completions",
        "authProvider": "github-enterprise"
    },
    "github-enterprise.uri": "https://cocopilot.org",
```

### JetBrains等 配置：

按照 coco dash 页面截图配置后，执行对应系统的脚本后重启IDE：
* `scripts/install.sh` 适用于 `macOS` 和 `Linux`
* `scripts/install-all-users.vbs` 适用于 `Windows`，为电脑上所有用户配置，需要有管理员权限。
* `scripts/install-current-user.vbs` 适用于 `Windows`，为当前用户配置，无需管理员权限。
* `scripts/uninstall` 相关脚本与之对应，为卸载配置。

其中 `http://127.0.0.1:8181` 是你启动的 `override` 服务地址。

### config.json 配置

```json
{
 "bind": "127.0.0.1:8181",
 "proxy_url": "",
 "timeout": 600,
 "codex_api_base": "https://api-proxy.oaipro.com/v1",
 "codex_api_key": "sk-xxx",
 "codex_api_organization": "",
 "codex_api_project": "",
 "codex_max_tokens": 500,
 "code_instruct_model": "gpt-3.5-turbo-instruct",
 "chat_api_base": "https://api-proxy.oaipro.com/v1",
 "chat_api_key": "sk-xxx",
 "chat_api_organization": "",
 "chat_api_project": "",
 "chat_max_tokens": 4096,
 "chat_model_default": "gpt-4o",
 "chat_model_map": {},
 "chat_locale": "zh_CN",
 "auth_token": ""
}

```

`organization` 和 `project` 除非你有，且知道怎么回事再填。

`chat_model_map` 是个模型映射的字典。会将请求的模型映射到你想要的，如果不存在映射，则使用 `chat_model_default` 。

`codex_max_tokens` 可以设置为你希望的最大Token数，你设置的时候最好知道自己在做什么。代码生成通常使用 `500` 即可。

`chat_max_tokens` 可以设置为你希望的最大Token数，你设置的时候最好知道自己在做什么。`gpt-4o` 输出最大为 `4096`

可以通过 `OVERRIDE_` + 大写配置项作为环境变量，可以覆盖 `config.json` 中的值。例如：`OVERRIDE_CODEX_API_KEY=sk-xxxx`

### DeepSeek Coder 设置
如果你希望使用 DeepSeek Coder FIM 来进行代码补全，着重修改以下配置：

```json
  "codex_api_base": "https://api.deepseek.com/beta/v1",
  "codex_api_key": "sk-xxx",
  "code_instruct_model": "deepseek-coder",
```

### Siliconflow 设置
如果你希望使用 Siliconflow FIM 模型来进行代码补全，着重修改以下配置：

```json
  "codex_api_base": "https://api.siliconflow.cn/v1",
  "codex_api_key": "sk-xxx,sk-xxx2,sk-xxx3...",
  "code_instruct_model": "Qwen/Qwen2.5-Coder-7B-Instruct",
```

截至目前，Siliconflow 共有三个模型支持 FIM。分别是 `Qwen/Qwen2.5-Coder-7B-Instruct`、`deepseek-ai/DeepSeek-Coder-V2-Instruct` 、`deepseek-ai/DeepSeek-V2.5`。其中 `Qwen/Qwen2.5-Coder-7B-Instruct` 是免费模型，另外两个是收费模型。

如果你有很多 Siliconflow API Key, 可以以英文逗号分隔填入`codex_api_key`字段, 这样可以很好的避免Siliconflow官方的 TPM RateLimit 对你编码速度影响(尤其使用收费模型时，用户级别较低，TPM 最低只有 10k)。



### 本地大模型设置
1. 安装ollama 
2. ollama run stable-code:code  (这个模型较小，大部分显卡都能跑)  
 或者你的显卡比较高安装这个：ollama run stable-code:3b-code-fp16
3. 修改config.json里面的codex_api_base为http://localhost:11434/v1/chat
4. 修改code_instruct_model为你的模型名称，stable-code:code或者stable-code:3b-code-fp16
5. 剩下的就按照正常流程走即可。
6. 如果调不通，请确认http://localhost:11434/v1/chat可用。
        
### 重要说明
`codex_max_tokens` 工作并不完美，已经移除。**JetBrains IDE 完美工作**，`VSCode` 需要执行以下脚本Patch之：

* macOS `sed -i '' -E 's/\.maxPromptCompletionTokens\(([a-zA-Z0-9_]+),([0-9]+)\)/.maxPromptCompletionTokens(\1,2048)/' ~/.vscode/extensions/github.copilot-*/dist/extension.js`
* Linux `sed -E 's/\.maxPromptCompletionTokens\(([a-zA-Z0-9_]+),([0-9]+)\)/.maxPromptCompletionTokens(\1,2048)/' ~/.vscode/extensions/github.copilot-*/dist/extension.js`
* Windows 可以用如下的python脚本进行替换
* 因为是Patch，所以：**Copilot每次升级都要执行一次**。
* 具体原因是客户端需要根据 `max_tokens` 精密计算prompt，后台删减会有问题。

```
# github copilot extention replace script
import re
import glob
import os

file_paths = glob.glob(os.getenv("USERPROFILE") + r'\.vscode\extensions\github.copilot-*\dist\extension.js')
if file_paths == list():
    print("no copilot extension found")
    exit()

pattern = re.compile(r'\.maxPromptCompletionTokens\(([a-zA-Z0-9_]+),([0-9]+)\)')
replacement = r'.maxPromptCompletionTokens(\1,2048)'

for file_path in file_paths:
    with open(file_path, 'r', encoding="utf-8") as file:
        content = file.read()
    
    new_content = pattern.sub(replacement, content)
    if new_content == content:
        print("no match found in " + file_path)
        continue
    else:
        print("replaced " + file_path)
    
    with open(file_path, 'w', encoding='utf-8') as file:
        file.write(new_content)

print("replace finish")
```

### 其他说明
1. 理论上，Chat 部分可以使用 `chat2api` ，而 Codex 代码生成部分则不太适合使用 `chat2api` 。
2. 代码生成部分做过延时生成和客户端 Cancel 处理，很有效节省你的Token。
3. 项目基于 `MIT` 协议发布，你可以修改，请保留原作者信息。
4. 有什么问题，请在论坛 https://linux.do 讨论，欢迎PR。

### Star History

[![Star History Chart](https://api.star-history.com/svg?repos=linux-do/override&type=Date)](https://star-history.com/#linux-do/override&Date)   


# test

一个跑在 **Cloudflare Workers** 上的极简 AI 聊天网页。用的是 Cloudflare 自带的 **Workers AI** 免费额度，**不需要任何 OpenAI / Claude 的 API Key**，也不用配密钥。

## 它长什么样

- 一个聊天界面（深色，手机也能用）
- 消息逐字流式返回，不用等整段生成完
- 对话只存在浏览器内存里，刷新即清空，不写数据库、不收集任何东西

## 目录结构

```
.
├── wrangler.jsonc      # Cloudflare 部署配置（关键：有这个文件才能部署）
├── package.json
├── src/
│   └── index.js        # Worker：/api/chat 接口，直接调用 Workers AI
└── public/
    ├── index.html      # 聊天页面
    ├── app.js          # 前端逻辑（流式解析）
    └── style.css       # 样式
```

## 一、本地预览（可选）

需要 Node.js 18+：

```powershell
npm install -g wrangler
wrangler login      # 会弹出浏览器，点 Allow
wrangler dev        # 打开 http://localhost:8787
```

如果 `wrangler login` 嫌麻烦，访问本地页面会报鉴权错误，那就直接跳到第二步，在 Cloudflare 网页上部署。

## 二、部署到 Cloudflare（网页操作，跟教程一样）

1. 打开 <https://dash.cloudflare.com/>，左边选 **Workers & Pages**
2. 点 **Create** → **Workers** → **Connect to Git**（导入 Git 仓库）
3. 选中这个仓库
4. 构建配置这样填：
   - **Build command**：留空
   - **Deploy command**：`npx wrangler deploy`
5. 点 **Deploy**，等一两分钟

部署成功后会给你一个 `https://test.<你的子域>.workers.dev` 的地址，直接就能聊。

> **为什么之前会报 `Could not detect a directory containing static files`？**
> 因为仓库里没有 `wrangler.jsonc`，`wrangler deploy` 不知道入口在哪、静态文件在哪。
> 这个仓库里已经写好 `main` 和 `assets.directory`，所以同样的命令现在能正常跑。

## 三、命令行部署（想更快的话）

```powershell
npx wrangler login
npx wrangler deploy
```

## 四、换模型

改 `wrangler.jsonc` 里的 `vars.MODEL`，常用的几个（都支持免费额度内调用）：

| 模型 | 说明 |
| --- | --- |
| `@cf/meta/llama-3.1-8b-instruct` | 默认，快，够用 |
| `@cf/meta/llama-3.3-70b-instruct-fp8-fast` | 更聪明，稍慢 |
| `@cf/qwen/qwen2.5-coder-32b-instruct` | 写代码更擅长 |

也可以用管理后台改：Workers & Pages → 你的 Worker → Settings → Variables and Secrets。

改完重新部署一次生效。

## 五、额度

Workers AI 的免费额度是**账号级自带**的，不需要"领取"：每天 10,000 Neurons（大致相当于几万到十几万字的对话，够个人日常用）。超了要等下一天，或者绑卡走付费。

在后台 **Workers & Pages → 你的 Worker → Metrics** 里能看到调用量。

## 六、常见报错

| 报错 | 原因 | 解决 |
| --- | --- | --- |
| `Could not detect a directory containing static files` | 缺 `wrangler.jsonc` 或 `main` 写错 | 确认这个文件在仓库根目录 |
| `No such module "src/index.js"` | 入口路径不对 | 改 `main` 指向真实文件 |
| 页面能开，发消息报 `HTTP 500 调用模型失败` | AI 绑定没生效 / 模型名写错 | 检查 `wrangler.jsonc` 的 `ai.binding`，并确认模型名拼写 |
| `429` / 额度提示 | 当天免费额度用完 | 等次日重置 |

## License

MIT，随便用。

# 🌸 星星树洞 · Treehole

一个**完全匿名**的留言树洞网页：访客写下一句话（可带心情标签、可附一张照片），拿到一个**编号**，之后凭编号 + 自己设的密码回来查看回信。

> 单文件、零构建、纯前端。丢到任何静态托管（Cloudflare Pages / Vercel / GitHub Pages）就能跑。

---

## ✨ 功能

- 💌 匿名投递：不记录身份，只需要一个你自己记得的密码
- 🏷️ 心情标签：多选 chips（累 / 难过 / 委屈 / 生气 / 焦虑 / 迷茫 / 开心 / 想家 / 说不清）
- 📷 附一张照片：前端自动压缩到 350KB 以内再上传
- 🔢 编号回信：凭编号 + 密码查看回复，密码本地 SHA-256 加盐哈希后存储
- 🐱 守洞猫：可拖动、会踱步、会说话的像素小猫（彩蛋）
- 🌙 匿名 · 不记录 · 说出来就轻一点

---

## 🚀 部署（3 步）

### 1. 建 Supabase 项目
新建一个项目，在 **SQL Editor** 里建表和函数（见下方「数据库结构」）。

### 2. 填配置
打开 `index.html`，把顶部两行改成你自己的：

```js
const SUPABASE_URL='在这里填你的 Supabase 项目 URL';
const SUPABASE_KEY='在这里填你的 anon public key';
```

### 3. 上线
任意静态托管：

```bash
# Cloudflare Pages 示例
mkdir -p /tmp/th && cp index.html /tmp/th/ && cd /tmp/th \
  && NODE_OPTIONS=--dns-result-order=ipv4first \
  && wrangler pages deploy . --project-name=你的项目名 --branch=main
```

---

## 🗄 数据库结构

表 `public.messages`：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | int8, PK, identity | 编号 |
| code | text | 对外编号（同 id） |
| pass_hash | text | 密码哈希 |
| content | text | 留言内容（照片以 `<<IMG>>URL` 追加） |
| reply | text | 树洞主的回信 |
| created_at | timestamptz | 投递时间 |
| replied_at | timestamptz | 回信时间 |

两个 RPC：
- `submit_message(p_content text, p_pass_hash text) -> int` 写入并返回编号
- `check_reply(p_code text, p_pass_hash text) -> setof messages` 校验密码并返回该条

> ⚠️ 记得给 `messages` 开 **RLS**，并且**不要**给 anon 角色任何直接 `select` 权限——读取只走 `check_reply`（内部校验密码）。完整 SQL 建议从你自己的 Supabase 后台导出后补进本文件。

---

## 🔐 安全须知（重要）

- ✅ **可以公开**：`SUPABASE_URL` + `anon public key`（它们本来就会下发到浏览器）。
- 🚫 **绝对不能公开**：`service_role key`、数据库密码、连接串（含 pooler 地址）、任何云服务 API Token。这些是**后台总钥匙**，能无视 RLS 读写删全库。
- 🚫 别把 `.env`、部署脚本、服务器密钥提交进仓库，先加进 `.gitignore`。
- 🔒 密码哈希加盐串 `xtsd::` 在 `hashPw()` 里，可自行改成你的盐。

---

## 📄 License

MIT —— 随便用、随便改，署名与否都行。

> 愿你今晚睡得踏实，明天醒来有光。

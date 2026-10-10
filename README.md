# 陈瀛锋 · 个人主页（CI/CD 练习）

一张**纯静态**的个人简历页。改完 `index.html` 推到 `main`，GitHub Actions 会自动
**先自检、再发布**——自检不过就不发布。

**双链接（同一份代码，三种部署方式）**

| 链接 | 部署方式 |
|---|---|
| https://yumin-sb.github.io/20260930_Resume_CICD_ST/ | GitHub Actions 自动发布（先自检，通过才上线） |
| https://chenyingfeng-resume.surge.sh/ | surge 命令行直传（`npx surge`，无检查） |
| https://4472ee7359a6451eb57429bd38e18c25.app.workbuddy.host | 静态站点托管，备用镜像入口 |

- 仓库：https://github.com/Yumin-sb/20260930_Resume_CICD_ST

---

## 一、本地怎么跑

```bash
# 1) 自检（和 GitHub Actions 里跑的是同一个脚本）
node tools/check-page.mjs

# 2) 本地预览（默认 http://127.0.0.1:7100）
npx live-server --port=7100 --host=0.0.0.0
```

> 自检脚本**不依赖任何第三方包**，Node 自带能力就够，所以不需要 `npm install`。

## 二、目录结构

```
.
├── index.html              页面内容（日常只改这一个文件）
├── styles.css              全部样式，配色用 CSS 变量集中管理
├── app.js                  唯一的小交互：导航高亮
├── assets/                 图片、字体、图标
├── tools/check-page.mjs    自检脚本（CI 和本地共用）
├── .github/workflows/      GitHub Actions 工作流
├── _headers                Netlify/surge 用的响应头（禁止被搜索引擎收录）
└── 课堂资料/               课程手册（预览用，不影响发布）
```

## 三、日常维护：改内容只看 `index.html`

`index.html` 顶部有一段维护说明，正文里埋了两个可搜索的锚点注释：

| 搜索这个 | 用来做什么 |
|---|---|
| `PROFILE` | 姓名、一句话定位、学校、联系方式 |
| `ITEM` | 需要复制/删除的列表项（经历、项目、技能、作品、记录） |

**改的时候记住两条**（自检脚本会拦）：

1. 导航里每个井号锚点，页面里必须有同名的 `id`；
2. 每张图片都要有描述性 `alt`，不能写"图片""照片"这种空话。

## 四、CI/CD 是怎么跑起来的

工作流文件：`.github/workflows/pages.yml`，名字叫「检查并发布完整简历」。

```
push 到 main  ──┐
                ├──> check（node tools/check-page.mjs）──> deploy（发布到 GitHub Pages）
手动 Run workflow ┘       失败则 deploy 不会开始
```

- `check` 检查四件事：必需文件在不在、本地引用是否存在、锚点是否有对应 `id`、图片 `alt` 是否合格。
- `deploy` 用 `needs: check` 锁住，**只有自检通过才会发布**——这正是 CI/CD 的意义：
  默认的「Deploy from a branch」是直接把文件当网站发出去，不经过任何检查。
- 因此仓库 Settings → Pages 的 Source 必须选 **GitHub Actions**。

## 五、本次改动与问题记录

详见 [问题记录.md](问题记录.md)。简单说：上一版是把课程模板原样提交的，
本次重写了页面内容与结构，代码从 730 行压到 386 行，并清掉了模板里不属于本人的信息。

## 六、surge 镜像怎么发（第二条链接）

surge 是"本地文件夹直传"的静态托管：**不做任何检查**，推上去就上线，适合做镜像。

```bash
# 首次：登录（会问你邮箱和密码，等于同时注册）
npx surge login

# 之后每次更新内容，重新发一次即可
npx surge --project . --domain chenyingfeng-resume.surge.sh
```

两个坑：

1. **surge 会在项目根目录生成 `CNAME` 文件**（内容是 surge 域名）。
   在 GitHub Pages 的仓库里提交它，Pages 会误以为你配了自定义域名，`github.io` 地址会失效。
   本项目已把它写进 `.gitignore`，**不要提交**。
2. **首次发布后 90 分钟内要验证邮箱**，否则域名会被收回。
   没收到邮件就跑 `npx surge verify` 重发，然后去邮箱点链接。

> `npx` 在本机默认会因 npm 缓存目录无写权限而失败，先设一句：
> `set npm_config_cache=%LOCALAPPDATA%\npm-cache`（CMD）。

## 七、常见故障排查（本机环境）

### 1. `git push` 报 `Failed to connect to github.com port 443`

本机 DNS 把 `github.com` 解析到了连不通的地址（解析结果是 `20.205.243.166`，实测超时）。
实测可用的真实 IP：`140.82.112.3`、`140.82.113.4`。

本仓库已经把可用 IP 写进本地 git 配置，直接 `git push` 就行。换网络后万一又失败，
重新执行一次这条命令即可：

```bash
git config --local http.curloptResolve github.com:443:140.82.112.3
```

### 2. `curl` 报 `schannel: CRYPT_E_NO_REVOCATION_CHECK`

本机用 Windows 证书链，吊销状态检查会超时。给 curl 加 `--ssl-no-revoke` 即可：

```bash
curl --ssl-no-revoke https://api.github.com/user
```

### 3. 命令里被注入了代理，GitHub 返回 502

报错形如 `CONNECT tunnel failed, response 502`。先把代理环境变量清掉再执行：

```bash
env -u http_proxy -u https_proxy -u HTTP_PROXY -u HTTPS_PROXY git push
```

## 八、许可与来源

页面结构与样式来自课程模板仓库 `02-gdit-ffd-20260818/ffd-student-course-kit`（分支 `p1-final-resume`），
在此基础上按个人情况重写。字体为思源宋体（SIL OFL，见 `assets/OFL-NotoSerifSC.txt`）。

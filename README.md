# 留名 · 文件署名工具

极简静态单页应用。自定义作者署名、透明度、HTML 首次作者声明；在浏览器中处理 HTML、PDF、DOCX、MD。没有后台，不上传资料，不保存密码。

## 使用

将整个文件夹放到静态服务器，打开 `index.html`。本地可运行：

```sh
python -m http.server 8080
```

访问 http://localhost:8080 。直接双击 HTML 也可进行基本处理；密码加密功能取决于浏览器的 Web Crypto 支持，推荐 HTTPS / localhost。

1. 填写署名，选择透明度、作者声明及导出方式。
2. 上传一个文件（最大 40 MB）。
3. 点击“生成受保护文件”，保存下载结果。
4. 分发前，用实际使用的 Word / WPS / Aru 检查一遍副本。保留原始文件。

## 免费部署

### GitHub Pages

1. 创建专用 GitHub 仓库，将 `index.html`、`app.js`、`.nojekyll` 上传到仓库根目录。
2. 仓库 Settings → Pages → Deploy from a branch。
3. 选择 main 分支和 `/ (root)`，保存。
4. 等待部署成功，从 Pages 页面复制实际网址。不要预先猜测网址。

### Vercel

1. 把项目上传到 GitHub 仓库。
2. 在 Vercel 中 Add New → Project，导入该仓库。
3. Framework Preset 选择 Other；不设置 Build Command，输出使用项目根目录。
4. 部署成功后复制 Vercel 给出的实际网址。

也可以使用 Vercel CLI，在此目录运行 `vercel`，根据引导登录和部署。

本项目无需服务器、数据库、环境密钥或构建步骤。免费计划和网址的持续可用性由平台决定，不能承诺永久或永远免费。

## 格式与边界

| 格式 | 处理结果 | 局限 |
| --- | --- | --- |
| HTML / HTM | 原文插入 CSS、首次弹窗脚本及固定右下角署名 | 源码仍能编辑；严格 CSP 可能阻止脚本或样式；浏览器数据被清除会再次提示 |
| PDF | 每页添加半透明图片水印，作者元信息 | 不提供 PDF 原生加密或权限锁；数字签名文档、已有加密文档拒绝处理 |
| DOCX | 页眉 VML 图片水印；可选 Word 只读限制 | 原有正文与页眉内容保留；只读限制可被绕过；第三方阅读器显示效果可能不同 |
| MD / Markdown | 署名引用段和版权注释，保留原始文本 | MD 纯文本无法设置透明度；YAML front matter 保持在首位 |
| MD 阅读版 | 显示原始 Markdown 文本的 HTML 阅读页，带署名和作者声明 | 不把 Markdown 转成富文本，避免执行内嵌 HTML |
| 密码保护包 | 自带解锁界面的独立 `.locked.html`，解锁后下载已署名文件 | 不是原格式原生密码；不阻止解锁后的修改、复制、截图和二传 |

旧 `.doc` 格式请先另存为 `.docx`。本工具不会伪造“无法去除”的保护，也不会禁用右键、破坏文件或加入远程验证。首次弹窗只是版权提示，不是技术授权机制。

加密使用 Web Crypto：PBKDF2-SHA256（310,000 次迭代、随机 16 字节盐）派生 AES-256-GCM 密钥，随机 12 字节 IV。密码不写入输出文件；不提供找回功能。使用足够强的独立密码，通过其他渠道发送给收件人。

## 依赖与隐私

HTML、MD 和密码包不需要外部库；PDF / DOCX 第一次处理会从 jsDelivr 按需下载：

- pdf-lib 1.17.1，MIT：https://github.com/Hopding/pdf-lib
- JSZip 3.10.1，MIT / GPLv3 双许可（按 MIT 使用）：https://github.com/Stuk/jszip

文件字节仅在本地处理，不发送到 CDN；依赖请求会向 CDN 暴露常规访问信息。首次处理 PDF / DOCX 需要联网。依赖失败会显示错误，不生成假的输出。若需要完全离线运行，可自行下载并审查对应依赖、保存到 `vendor/`，将 `app.js` 的两个 CDN URL 改为本地路径，同时保留依赖的许可证声明。

工具不会执行上传 HTML。页面效果预览是内置示例；导出的原卡片仍含其原本的脚本，是否适合导入由使用者自行判断。

## 开发验证

`node --check app.js` 检查语法。`tests/core.test.cjs` 验证保留 HTML 原文、转义、声明配置、MD front matter 和密码加解密。浏览器测试可访问 `tests/browser.html`。

网站地址：https://qcherry1989.github.io/liuming-tool/

## 自动推荐与新版说明

上传新文件后，导出方式会重置为“自动推荐”。HTML、PDF、DOCX 保留原格式；MD 默认生成带署名的纯文本 HTML 阅读版。需要保留 Markdown 语法供编辑时，手动选择“保留原格式”。密码包始终由用户主动选择。

HTML 首次声明会使用本机 localStorage / IndexedDB 记录；存储可用时首次显示后不再重复。存储不可用时仍显示声明，重新打开可能再次显示。不会写入 Aru 房间状态。

可选“HTML 署名自动恢复”通过 MutationObserver 恢复被普通删除的署名节点或注入样式。它不禁用游戏或卡片的交互，不禁用右键或复制。拥有源代码的人或 AI 仍可移除恢复逻辑，因此不构成防 AI 去水印保证。

PDF 新版禁用对象流和自动表单外观重绘，生成后重新加载并核对页数；下载完成后可在新标签中预览。兼容导出和重新解析不等于所有阅读器都已通过实际渲染测试。如遇阅读器报错，请保留原始 PDF，并提供具体错误信息。

补充测试：`node tests/popup.test.cjs`、`node tests/export.test.cjs`。PDF 保存参数与重新读取流程使用模拟对象测试，不能替代真实浏览器和 PDF 阅读器验证。

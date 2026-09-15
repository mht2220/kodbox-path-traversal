KodBox (可道云) Public Share Unauthorized Access Vulnerability
   
  KodBox（可道云）公开分享未授权访问漏洞
     
  Summary / 摘要

  KodBox's public share / SEO feature (allowSEO=1) exposes a complete
  index of all public shares, while the explorer/share/* API endpoints
  lack server-side access control. An attacker — without logging in and
  without going through the normal front-end share flow (no share link
  or QR code required) — can anonymously enumerate share tokens, list
  share directories, and download otherwise-protected shared files,
  leading to unauthorized disclosure of sensitive information.

  可道云的公开分享 / SEO
  功能（allowSEO=1）会对外输出全部公开分享索引，而 explorer/share/*
  系列接口缺少服务端访问控制。攻击者无需登录、无需正常前端分享流程（无需
  分享链接或二维码），即可匿名枚举分享令牌、列举分享目录并下载本应受保护
  的分享文件，导致敏感信息未授权泄露。

  ---
  Affected Product / 受影响产品

  ┌─────────────┬───────────────────────────────────────────────────┐
  │    字段     │                        值                         │
  ├─────────────┼───────────────────────────────────────────────────┤
  │ Product /   │ KodBox (可道云)                                   │
  │ 产品        │                                                   │
  ├─────────────┼───────────────────────────────────────────────────┤
  │ Version /   │ All versions / 全版本（已验证 Verified V1.69）    │
  │ 版本        │                                                   │
  ├─────────────┼───────────────────────────────────────────────────┤
  │ Vendor /    │ 杭州可道云网络有限公司 (Hangzhou KodCloud Network │
  │ 厂商        │  Co., Ltd.)                                       │
  └─────────────┴───────────────────────────────────────────────────┘

  Vulnerability Type / 漏洞类型

  - CWE-284: Improper Access Control / 不当访问控制
  - CWE-306: Missing Authentication for Critical Function /
  关键功能缺失认证
  - CWE-200: Exposure of Sensitive Information to an Unauthorized Actor
  / 敏感信息暴露

  Severity / 严重性

  - CVSS 3.1: 7.3 (HIGH / 
  高危)（供参考，可按平台定级调整；分享目录含大规模 PII 时可上调至
  Critical）
  - Vector / 参考向量：CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

  ---
  Steps to Reproduce (Redacted) / 复现步骤（已脱敏）

  1. Anonymous index enumeration / 匿名索引枚举
  匿名访问 GET /?sitemap；因目标站点开启
  SEO（allowSEO=1），服务端返回全部公开分享的永久链接，其中包含分享令牌
  shareHash（此步无需任何认证）。
  Anonymous request to GET /?sitemap; because SEO is enabled
  (allowSEO=1), the server returns permanent links to all public shares,
  each containing a shareHash token (no authentication required at this
  step).
  2. Unauthorized directory listing / 未授权列目录
  匿名调用 POST index.php?explorer/share/pathList，参数
  shareID=<shareHash>、path={shareItemLink:<shareHash>}/；服务端未校验登
  录态，直接返回该分享目录的文件清单（文件名、大小）。
  Anonymous request to POST index.php?explorer/share/pathList with
  shareID=<shareHash> and path={shareItemLink:<shareHash>}/; the server
  does not verify the login state and returns the share directory's file
  listing (name, size).
  3. Unauthorized file download / 未授权下载文件
  匿名调用 POST index.php?explorer/share/fileOut，参数
  shareID=<shareHash>、download=1、path={shareItemLink:<shareHash>}/<文
  件名>，成功读取到非授权文件正文（含敏感内容）。
  Anonymous request to POST index.php?explorer/share/fileOut with
  shareID=<shareHash>, download=1, and
  path={shareItemLink:<shareHash>}/<filename>, successfully reading the
  file body of an unauthorized file (containing sensitive content).

  ▎ 关键点 / Key point：上述三个接口均无需会话 Cookie，亦不校验 
  ▎ CSRF_TOKEN（实测空值可返回 2xx），构成纯无状态匿名访问。
  ▎ All three endpoints require no session cookie and no CSRF_TOKEN 
  ▎ (verified: empty values still return 2xx), constituting purely 
  ▎ stateless anonymous access.

  ---
  Impact / 影响

  - 分享的「仅登录可见 / 密码 / 有效期」等访问控制策略完全失效。
  The share's access-control policies (login-required / password /
  expiration) are fully bypassed.
  - 攻击者可批量枚举所有公开分享令牌并遍历目录，造成未授权敏感信息泄露
  An attacker can enumerate all public share tokens in bulk and traverse
  directories, causing unauthorized disclosure of sensitive information

  Remediation / 修复建议

  1. 关闭或收紧 allowSEO，禁止向未登录用户输出公开分享索引。
  Disable or restrict allowSEO; do not expose the public share index to
  unauthenticated users.
  2. 在服务端对 explorer/share/*
  全接口强制鉴权与权限校验（不能仅依赖前端隐藏）。
  Enforce server-side authentication and authorization on all
  explorer/share/* endpoints (do not rely solely on front-end hiding).
  3. 对 path 参数进行严格白名单校验，使用 realpath()
  规范化路径并限制在分享目录内。
  Apply strict allow-list validation to the path parameter, normalize it
  with realpath(), and constrain it to the share directory.
  4. 建议升级至官方最新版本，并审计历史分享令牌与访问日志。
  Upgrade to the latest official version and audit historical share
  tokens and access logs.

  Timeline / 时间线

  - 2026-09-15: 发现漏洞 / Discovery
  - 2026-09-15: 向厂商报告 / 提交 CVE / Reported to vendor / CVE
  submitted
  - YYYY-MM-DD: 公开发布 / Public disclosure

  References / 参考

  - CVE-2026-XXXXX（待分配编号后补 / to be filled after ID assignment）
  - https://cveform.mitre.org/

  Credits / 致谢

  - mht2220

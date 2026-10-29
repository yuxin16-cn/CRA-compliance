

> php2shell（CVE-2026-109618 / CVE-2026-109619）爆发后的 72 小时，是对制造商 CRA Article 14 合规能力的一次压力测试。CRA 要求的合规义务不再仅仅是"报告提交"，而是"持续的网络安全能力"；而CRA的提出和落地也预示着网络安全的合规已经从上市前的测试报告进入了持续合规的时代。从这 72 小时里不同制造商的表现也可以看出来，不同制造商对CRA报告义务的应对准备完全不同。

---

## 为什么选该漏洞事件作为案例

2026 年 9 月 11 日，欧盟《网络弹性法案》（Cyber Resilience Act, Regulation (EU) 2024/2847）第 14 条的被主动利用漏洞与严重事件报告义务正式生效。10 月 20 日[^1]，php2shell 漏洞公开。

如此大规模的PHP2Shell漏洞爆发讨论和CRA的第14条报告义务的生效时间点中间只隔了不到六周。这也是 CRA 报告义务生效以来，第一次波及面足够大、时间线足够完整、技术细节足够公开的真实事件，可以说是行业从业者及制造商在CRA 第14条报告义务上线后统一面临的一场“大考”。大多数针对PHP2Shell的漏洞报道关注的是紧急性、严重度分析以及应急缓解措施。作为合规工程师，我更关心另一个问题：**如果我的客户（CRA定义下的制造商角色）的产品里嵌了 PHP，这 72 小时里我的客户需要做什么？以及我的客户有没有能力第一时间知道自己的产品是否包含该漏洞。**



## PHP2Shell事件里的CRA报告义务归属


CRA 的Article 14义务主体是把含数字元素的产品投放欧盟市场的制造商。如果一个企业在欧盟以自有品牌销售的产品是一个带 PHP Runtime的网关设备、一套交付给客户的软件系统、一台预装了管理界面的嵌入式设备——作为CRA定义下的制造商，PHP2shell 对该企业来说就是一个合规事件[^2]。

如果一个企业只是在自家官网跑了 WordPress，那么该企业仅仅是作为PHP 的**用户**而非使用PHP组件的制造商（依据CRA定义）。当然我建议企业按照官方建议采取紧急措施，但驱动力是其他框架或者安全团队，不是 CRA的相关义务要求。

下文的所有讨论主体都基于前者：**把 PHP 打包进产品并卖到欧盟市场的制造商。**[^3]

## 时间线梳理：72 小时里发生了什么


**T-1 Day（10 月 20 日 19:33:47 UTC）：信息泄露。** 原定 10 月 22 日的协调披露，因 Hacker News 以及社交平台上的帖子提前泄露 CVE 信息而被打乱，PHP 团队被迫将正式披露提前到 21 日[^4][^5]。

**T+0 Day（10 月 21 日 13:30 UTC）：补丁发布。**[^29] PHP 8.5.12 / 8.4.27 / 8.3.36 / 8.2.35 四个版本发布[^6]，修复 CVE-2026-109618和 CVE-2026-109619，两者评分均为 CVSS 9.2 （CVSS v4.0）[^7][^8]，均可未认证远程代码执行。PHP 8.1 及更早的 EOL 分支没有官方更新。

**T+ 2 Day（10 月 23 日）：第三个 CVE公开。** [^1][^9] PHP 披露 CVE-2026-115560[^9]，仅影响配置不寻常的应用，官方判断风险低、**不发带外补丁**，只给出缓解措施（`session.upload_progress.enabled=0`）[^1][^9]，等下一个常规版本一并修复。CERT-EU 的通告也因此在三天内迭代了三个版本（10/20 v1.0 → 10/21 v1.1 → 10/23 v1.2）。CVE官方在目前（北京时间2026-10-26 20:57 查阅）未给出针对CVE-2026-115560的评分。


面对网络上针对PHP漏洞大规模的讨论，我们的客户以及其他出海制造商通常会有三种反应[^10]：
### 第一类：有准备的企业（有常规应急响应团队，或者在CRA第一批义务生效前即建立SBOM依赖管理和报告流程）

- **T-20h**：威胁情报渠道(例如PHP邮件列表，上游厂商TI feed等等)捕获泄露信息，比官方通告早 20 小时启动评估。
- **T+0 ~ T+2h**：SBOM 查询确认哪些产品/版本内嵌 PHP、用的是 mod_php 还是 FPM。
- **T+2h ~ T+4h**：启动事件响应流程，评估暴露面，结合产品上下文评估风险，匹配官方缓解措施。
- **T+4h ~ T+24h**：发布临时缓解（WAF 规则、配置加固），通知关键客户；按"可能被主动利用"的最严口径准备报告材料，留存时间戳证据。
- **T+48h**：第三个 CVE 出现时，重新跑一次 SBOM 查询。

### 第二类：准备不足的企业（仅关注Article 14的报告义务，未建立SBOM、未作前置风险评估）

- **T+0 ~ T+24h**："我们产品里用了 PHP 吗？"——开始人工翻代码、问研发。
- **T+24h ~ T+48h**：终于确认受影响，但说不清有多少在网设备、分布在哪里、谁是客户。
- **T+48h ~ T+72h**：匆忙发公告。此时第三个 CVE 出现，评估要重来一遍——第二次人工翻代码。
- 事后无法回答监管的三个关键问题：
  - "你们什么时候知道漏洞的？"（10 月 21 日，但拿不出证据）
  - "你们做了什么持续监测？"（答不上来）
  - "为什么现在才报告？"（因为刚搞清楚）

### 第三类：状况外的企业

- **T+0 ~ T+72h**："PHP 不是做网站的吗？跟我们有关系吗？"

## 一个需要提前确定的问题：php2shell 触发了制造商的 Article 14 义务吗

这是整个事件里最少被公开讨论，但是最触发制造商报告焦虑的问题。

CRA Article 14 的报告义务有一个明确的触发门槛：只针对**被主动利用的漏洞**（actively exploited vulnerability）[^11]和**严重安全事件**（severe incident having an impact on the security of product）[^12]。"被主动利用"的定义是：有可靠证据表明恶意行为者未经系统所有者许可利用了该漏洞[^13]，一个单独的CVSS 9.2 评分并不给使用PHP组件的企业自动带来报告义务。

而 php2shell 恰好卡在模糊地带：

- Amazon Threat Intelligence 观测到 **10 月 17 日就出现了首次利用尝试** [^14]——比公开披露早四天；
- Cloudflare 称补丁发布后几个小时内即观测到数百万次利用尝试[^15]；
- 但该漏洞2026年10月26日才被列入 CISA KEV 目录[^16][^17]，比漏洞披露晚了5天左右，目前没有看到被确认的大规模在野利用[^25][^27]。

这样的一个情况就给制造商带来一个棘手的判断问题——"利用尝试"算不算"被积极利用"？谁来认定？认定期间的时钟怎么走？什么时候算制造商"Awareness(知悉)"的起算点呢？目前没有看到任何官方口径给出答案。

**这恰恰暴露了真正的合规能力差距：**
- 没准备的企业把 CRA Article 14 理解成一个"开关"——触发了就报，没触发就当不知道/不管；
- 有准备的企业把 CRA Article 14 理解成一个"连续义务"——即使最终判断无需报告，也已经按可报告标准完成了影响评估、证据留存和时间戳记录。
因为当监管事后问"你们凭什么判断不需要报告"时，那份留存的评估过程本身就是尽职证据。

值得补充一个细节：欧委会 2026 年 7 月的指南把"知悉"（awareness）界定为初步评估达到合理确定性之时，而非检测之时。欧盟官方指南说明，某项漏洞已在可靠的媒体渠道上被公开报道时，该漏洞也可被视为已知漏洞[^18]。这意味着"你什么时候知道的"这个关键证据，只能靠企业自己留存。平台的记录不是你的证据，你自己的时间戳笔记才是。[^19]


## 三个准备度维度

### 维度一：组件可见性（SBOM）

**72 小时内的差异：**

- **有 SBOM**：查一下 SBOM和依赖项，可以在短时间内知道哪些产品用了 PHP、什么版本、什么部署形态。
- **没 SBOM**：几个工程师翻几天代码，72 小时后可能还在确认影响面——而第三个 CVE 让一切重来。

**条款映射**：CRA 附件 I 第二部分第 1 点要求制造商识别并记录产品中的组件，包括以机器可读格式编制至少覆盖顶层依赖的 SBOM。

我在实际的制造商实操中发现，很多制造商有一个普遍的共识，就是SBOM的法定义务[^20]2027 年 12 月 11 日才随 CRA 全面适用而生效[^21]，但 Article 14 针对制造商的报告义务 2026 年 9 月 11 日就已经落地了。从实践角度讲，报告义务和漏洞处理要求[^22]其实是没有办法分开的。我这边的理解是，法规对报告义务的强制要求其实其实提前连带激活了漏洞处理要求的相关前置能力——SBOM管理、风险评估、CVD政策流程等等。这里是很多做过欧盟RED-DA（协调路径EN 18031: 2024系列）的企业特别容易忽略的地方。

php2shell 证明了：没有 SBOM 的制造商在漏洞面前是"失明"的。他们不是在"响应"，是在"寻找"。而 CRA Article 14 隐含的要求是，你得先知道自己的产品里到底用了什么。

### 维度二：持续监测

这次事件给"持续监测"提供了一个极其具体的检验场景：

Amazon 观测到的首次利用尝试是 **10 月 17 日**[^14]。官方披露是 **10 月 21 日**[^6]。中间差了约四天。

- **有监测、有日志的制造商**：这四天是可回溯的。即使当时不知道漏洞存在，事后也能用披露的 IoC 和攻击特征回查日志，回答"我们是否在披露前已被攻击"。
- **没日志的企业**：这四天是永久的盲区。不是"没被攻击"，是"永远不知道有没有被攻击"。

这直接决定了CRA Article 14 场景下的企业合规状态：如果监管认定漏洞在披露前已被主动利用，时钟的起点可能追溯到企业"应当知悉"的时刻。**那四天的日志就是尽职调查的证据。没有日志的企业，即使最后"报了"，也无法证明自己"尽力了"。**

但是我发现在实操中，很多制造商并没有建立起一个主动的漏洞管理机制，很多制造商还是依赖定期扫描，或者关注一下漏洞平台的KEV清单，更有甚者还是以定期对照基线配置核查表检查的方式来进行一个产品安全上的管理。这样的一个管理动作，其实是没有办法满足CRA的漏洞管理要求的。首先，不是所有的被利用的漏洞都会被收录在KEV里[^26]；其次，KEV的更新有一定的时间滞后性，如果制造商永远等待官方的KEV收录而不进行主动监测，可能会在面对监管时无法展示尽职证据；最后，KEV不针对特定制造商的产品上下文，无法提供漏洞管理所需要的前置信息。

### 维度三：风险分级决策

PHP官方对最近公开的三个CVE，自己就是基于风险分级处理的：
- 针对两个 9.2 分的漏洞发带外补丁，四个分支同步修复；
- 而第三个漏洞（CVE-2026-115560）因仅影响不寻常配置，且利用条件苛刻，影响范围较小，被判断为低风险，固未发带外补丁，只提供缓解措施。

对于制造商来说，基于风险的漏洞管理不仅仅是符合CRA要求的，也是符合“风险-收益”平衡的。对于同一个 9.2 分——在暴露公网的 mod_php 管理界面上和在内网隔离的 FPM 设备上，带来的风险是不一样的[^23] —— FPM与mod_php均受影响，只是FPM的直接退出机制降低了RCE的成功率，该差异在于进程隔离带来的可利用性与影响后果[^28]。而对于 CVE-2026-115560，可能只需要一条配置变更。

**没有前置风险评估的企业通常有两种路径：过度反应（全线停机，造成业务损失），或反应不足（漏掉了真正暴露的关键设备）。** 而 CRA 要求的风险评估是基于产品的预期用途和运行环境——这恰恰是无法外包给"漏洞扫描器结论"的判断。

## 结语：合规是持续能力，不是报告动作

PHP2Shell 的 72 小时里，三类企业拉开的差距，没有一项是"报告模板"能弥补的[^24]：

| 能力          | 法条出处                                               | 本质                                     |
| ----------- | -------------------------------------------------- | -------------------------------------- |
| 组件可见性（SBOM） | CRA Annex I Part II                                | 知道自己用了什么，知道自己需要监测什么                    |
| 持续监测与日志     | CRA Article 14 i.V.m CRA Annex I Part II           | 知道什么时候被攻击了、什么时候算"知悉"                   |
| 风险分级决策      | CRA Annex I Part I (1) i.V.m CRA Article 13(2) (7) | 漏洞Triage: 知道该先修什么、可以不修什么，并根据实际情况更新风险评估 |
| 报告动作        | CRA Article 14（24h / 72h /AEV-14 天or严重事件- 一个月）     | 以上三项能力综合决定的漏洞处理能力                      |

大多数企业准备的是最后一行，可是类似PHP2Shell这样的漏洞检验的是企业前三行的基础能力。
而 CRA Article 64 (2) 的后果摆在那里：违反附件 I 基本要求及第 13、14 条义务，最高罚款 1500 万欧元或全球年营业额 2.5%，以较高者为准[^30]。下一次事件对一些企业来说，可能不会再是演练了。

---

**本系列后续：**

- 《9.2 分 ≠ 需要报告：CRA Article 14 的触发门槛辨析》——"积极利用"的认定难题
- 《从 PHP 基金会到中国出口商：一次漏洞事件里的三层合规主体》——开源 steward / 制造商 / 进口商分销商的义务分层
- 《如果 php2shell 发生在 2028 年》——CRA 全面实施后的沙盘推演

---

**参考来源**
[^1]: 漏洞公开时间线参考[CERT-EU - Critical Vulnerabilities in PHP](https://cert.europa.eu/publications/security-advisories/2026-018/?utm_campaign=309872852-Newsletter%20EN&utm_medium=email&_hsenc=p2ANqtz-_SzqWXFQEHtuBNZozEXBZ8Ir46jqH43jHdAvXkNOhQ0wntg2p8G95wV1Yn_L-3u3_2ZKswFb6GL_iwYYxBFO0qhzTOFA&_hsmi=148930588&utm_content=148930588&utm_source=hs_email)
[^2]: 本文暂不讨论CRA Article 19 (5) 规定的进口商义务。
[^3]: 其他CRA规定下经济运营者角色的义务分层暂不在本文展开。
[^4]:  [php.announce: [ANNOUNCE] Upcoming Critical PHP Security Release](https://news-web.php.net/php.announce/509)
[^5]: [PHP2Shell: The AI-Hunted PHP Bugs That Put the Web on Emergency Patch Alert - DEV Community](https://dev.to/techpulse01239/php2shell-the-ai-hunted-php-bugs-that-put-the-web-on-emergency-patch-alert-14h9)
[^6]: [PHP: News Archive - 2026](https://www.php.net/archive/2026.php#2026-10-21-5)
[^7]: [CVE Record: CVE-2026-109618](https://www.cve.org/CVERecord?id=CVE-2026-109618)
[^8]: [CVE Record: CVE-2026-109619](https://www.cve.org/CVERecord?id=CVE-2026-109619)
[^9]: [PHP: News Archive - 2026](https://www.php.net/archive/2026.php#2026-10-23-1)
[^10]: 本文内的厂商反应时间线仅为依据常规和简化流程拟估，并非本人客户实际反应时间，亦非演练预估时间。
[^11]: CRA Article 14 (1)
[^12]: CRA Article 14 (3)
[^13]: CRA Article 3 (42) 
[^14]: [Amazon Threat Intelligence identifies PHP2Shell exploitation prior to public disclosure in MadPot | AWS Security Blog](https://aws.amazon.com/cn/blogs/security/amazon-threat-intelligence-identifies-php2shell-exploitation-prior-to-public-disclosure-in-madpot/)
[^15]: 转引自WindowsReport: [Millions of Attacks Hit PHP Websites Within Hours, Cloudflare CEO Warns](https://windowsreport.com/millions-of-attacks-hit-php-websites-within-hours-cloudflare-ceo-warns/)
[^16]: 初稿撰写时（2026年10月26日20:39北京时间）尚未见CISA KEV收录；次日（2026年10月27日07:41北京时间）确认已被收录CISA KEV收录。本文以收录后状态为准 。
[^17]: [CISA Adds Two Known Exploited Vulnerabilities to Catalog | CISA](https://www.cisa.gov/news-events/alerts/2026/10/26/cisa-adds-two-known-exploited-vulnerabilities-catalog)
[^18]: C(2026) 5252 EU Commission CRA Guidance 9.1 (234)，该条虽然被归纳在产品基本安全要求的说明下，但是我个人认为，对于“已知”的判定，欧盟还是对制造商在漏洞管理的主观能动性上提出了要求。
[^19]: C(2026) 5252 EU Commission CRA Guidance 9.1 (218)
[^20]: CRA Annex I Part II (1)
[^21]: CRA Annex I Part II in accordance with CRA Article 13 (8)
[^22]: CRA Annex I Part II 
[^23]: [RHSB-2026-013 Request handling memory corruption - php (CVE-2026-109618, CVE-2026-109619, CVE-2026-115560) - "php2shell" | Red Hat Customer Portal](https://access.redhat.com/security/vulnerabilities/RHSB-2026-013)
[^24]: 被主动利用漏洞和严重安全事件的报告时间点计算方式不同，请对照法规核验。
[^25]: 并非所有被利用漏洞都会进入CISA的KEV，关注KEV表单来看漏洞被利用状况是我面对的制造商里比较常见的一个判断误区。
[^26]: [Why is an exploited CVE not in the CISA KEV catalog?](https://safeguard.sh/resources/blog/why-is-an-exploited-cve-not-in-the-kev-catalog)
[^27]: 我这边截至2026年10月27日20:13，欧盟[EUVD](https://euvd.enisa.europa.eu/)一直是403状态，仅可在[CVD Portal](https://cvdportal.com/euvd)上看到对PHP2Shell两个漏洞的KEV收录。
[^28]: [PHP2Shell (CVE-2026-109618): How to Patch PHP Now | Phpscientist](https://phpscientist.com/blog/php-security-release-october-2026-cve-2026-109618-cve-2026-109619/)
[^29]: 采用[CERT-EU]([CERT-EU - Critical Vulnerabilities in PHP](https://cert.europa.eu/publications/security-advisories/2026-018/?utm_campaign=309872852-Newsletter%20EN&utm_medium=email&_hsenc=p2ANqtz-_SzqWXFQEHtuBNZozEXBZ8Ir46jqH43jHdAvXkNOhQ0wntg2p8G95wV1Yn_L-3u3_2ZKswFb6GL_iwYYxBFO0qhzTOFA&_hsmi=148930588&utm_content=148930588&utm_source=hs_email))发布时间口径
[^30]: CRA Article 64 的罚则框架随 CRA 于 2027 年 12 月 11 日全面适用而生效（CRA Article 71(2)），2026.9–2027.12 之间存在着"义务先行、罚则后置"的窗口期：现在漏报或者忽视报告义务，CRA Article 64(2) 的罚则严格来讲存在执行依据和时间节点上的争议空间，但企业的履约记录会留痕并被监管记录在案。我认为“罚则滞后”绝不等于“没有风险”，这段空窗期构成的是法律执行层面的不确定性，而非合规义务的豁免。窗口期为何不是免责期，本系列另文展开。




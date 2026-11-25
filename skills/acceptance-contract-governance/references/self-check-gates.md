# 交付前机验命令全集（self-check gates）

> 交付指令文档前**一次跑完**；SKILL.md Verification Checklist 的执行细节在此，逐条给命令与判据。
> 通用纪律：① POSIX 管道为主口径（git-bash 执行；Windows 原生 shell 用等价管道，**口径不变**）② 凡含 grep 正则的闸——先一正一负两个样例文件活体测再上岗（显示层转义与变体选择符会骗人，可疑处 `od -c`/码点级复核）③ 闸失败=先修任务书，不是先修口径（口径若与某书冲突，先验证口径再下结论——同理也适用反向）④ 闸模式活体测固定样例见 §20。

## 0. 文档主表边界变量

| 变量 | 取值 | 说明 |
|---|---|---|
| `$DOC` | 交付物路径 | |
| 起点 | `^#+ *3\. `（如 `## 3. 验收点表`） | 区间自该标题行**下一行**起 |
| 终点 | `^## (3\.5|4)([. ]|$)`（二级标题：`## 3.5 深度维度覆盖表` / `## 4. 判据表`） | 区间止于该标题行**之前**。**禁止用 `^#+ *(3\.5|4\.)` 类宽匹配**——域子节标题 `### 3.5 …` 会误触发提前截断（实证：某书 191 点被截成 56，恰为 .5 子节之前的域合计） |
| 行形态 | 主表行有**两种**：`| AC-… |` 与 `| ⛔ AC-… / ⏸️ / ⏭️ |`（门槛前缀在 ID 前） | **裸 `^| AC-` 会把前缀行整批漏计**（实证：某书主表 136 被误数成 119，差额 17 恰为带 ⛔/⏸️/⏭️ 前缀的挂链行）；口径必须按字段归一化取 ID，而非只认行首形态 |

**全文档 ID 边界前置（所有计数/悬空/入口/场景闸共用）**：先运行以下函数，任何 `INVALID_ID` 都使后续扫描结果作废；PowerShell 使用 §13 同一可选子字母+负向边界模式。

```bash
check_id_boundaries() {
  awk '{ rest=$0; while (match(rest, /AC-[A-Z0-9]+-[0-9]{3}[A-Z]?/)) {
    token=substr(rest,RSTART,RLENGTH); tail=substr(rest,RSTART+RLENGTH,1)
    if (tail ~ /[A-Z0-9]/) print "INVALID_ID " token > "/dev/stderr"
    rest=substr(rest,RSTART+RLENGTH)
  } }' "$DOC"
}
```

**精确口径的由来（禁止回退裸 grep）**：裸 `grep '^| AC-' $DOC` 会把 ① 各 §3.x 子表头行（`| AC-ID | …`）与 ② §4/判据表等区域的 AC 引用行一并计入——实证：某书主表 191 因 5 行判据表引用行混入被误数成 195，反向曾把自家零重号书写成重号书。凡计数相关闸一律走区间管道。

## 1. 主表行数 / 唯一数 / 重号（一次出全）

**scan 函数（全部区间闸共用，含门槛前缀行收集）**：

```bash
scan() {
  awk '/^#+ *3\. /{f=1;next} f && /^## (3\.5|4)([. ]|$)/{f=0} f' "$1" \
  | awk -F'|' '{ id=$2; if (match(id, /AC-[A-Z0-9]+-[0-9]{3}[A-Z]?/)) { token=substr(id,RSTART,RLENGTH); tail=substr(id,RSTART+RLENGTH,1); if (tail ~ /[A-Z0-9]/) print "INVALID_ID " token > "/dev/stderr"; else print token } }'
}
```

（对 `$2` 字段取首个 `AC-域名-###` +可选子字母记号；表头和非 ID 单元格落榜，门槛前缀行不丢，后随额外字母/数字即 INVALID_ID。）

```bash
scan "$DOC" | sort | uniq -c | sort -k1,1 -k2,2
```

判据：① 输出行数=唯一 AC 数=文档所有口径数字 ② `uniq -c` 首列全为 1 ③ stderr 无 `INVALID_ID`。`AC-02-002`、`AC-02-002A`、`AC-02-002B` 是三个不同合法 ID；后随字母/数字的截断匹配必须报错，不得静默收集。
Windows PowerShell 等价（口径不变）：见 §13 可复制命令集（区间锁=标题行号，ID 归一=第 2 字段 `-cmatch` 收集，同样覆盖门槛前缀行，勿写死行首形态）。
**双重反例实证（都曾被真书触发，勿回退）**：不剔表头=计数偏大（判据表引用行混入曾把 191 声明数成 195）；只认行首 `^| AC-`=计数偏小（门槛前缀行漏计曾把 136 数成 119）——片面的行首口径**两个方向都能错**。

## 2. 悬空引用（全文 AC 记号 ⊆ 主表 ID 集）

```bash
comm -23 \
 <(grep -oE 'AC-[A-Z0-9]+-[0-9]{3}[A-Z]?' "$DOC" | sort -u) \
 <(scan "$DOC" | sort -u)
```

判据：输出为空。非空行=被引用但主表不存在的 ID（悬空/断号）。
（判据表等区域**有意**引用主表 ID 属合法——它们在右侧集合里，不进差集。）

## 3. 列完整性（主表行管道字段数 = 列数+1）

```bash
awk '/^#+ *3\. /{f=1;next} f && /^## (3\.5|4)([. ]|$)/{f=0} f' "$DOC" \
 | awk -F'|' '{ id=$2; if (match(id, /AC-[A-Z0-9]+-[0-9]{3}[A-Z]?/)) print NF-1 }' | sort | uniq -c
```

判据：单一值 = 表头列数+1（全域轮七列含「向」→ 输出 `8`；六列窄域 → 输出 `7`）。出现第二分布=某行断列。

## 4. 运行时外部规范指针清零（冻结自包含）

```bash
grep -nE '见契约|以.*V[0-9]+\.[0-9]+|见主契约|以主契约为准|主契约原样要求|契约 §|以契约为准|全表见|执行期以.*当前版本|执行侧.*当前版本|当前版本为准' "$DOC"
```

判据：零命中（静态出处、密钥提供方、缺陷台账和项目更严安全红线豁免）。另核任务书头部有静态出处与快照哈希，且状态机/授权/报告/产物/契约规范已内联；仅写指针不算内联。

## 5. 派生依赖清零

```bash
grep -nE 'tests/plans/.*uat|docs/reviews/UAT|执行单元|以.{0,24}(历史任务书|旧版任务书|块书).{0,24}(为基|为准|基底|承接|沿用)|(历史任务书|旧版任务书|块书).{0,24}(作为基底|为基底|承接|沿用)' "$DOC"
```

判据：输出是**候选**，不是机械零命中；命中后判语境。「禁止/不得/禁读/不作基底」与头部 supersede 留痕为安全否定/溯源；真实「作为基底/承接/沿用」才违规。与 §23 共用同一语境判定。
**口径边界（活体测必须覆盖三个面）**：
- 正样例（应命中）：「以块书为基继续生成」「承接块书 AC 区间沿用」「把两份块书列为执行单元」——依赖语境。
- 非违规样例（可不命中；若候选命中须判安全）：「另派块级深化任务书做增量深化」「历史任务书/契约不作基底」「旧版任务书禁读」「禁止以历史任务书为基底」「不得将历史任务书作为基底」——前瞻、禁止或溯源语境。
- 裸「块书」词族**已从模式移除**：曾误伤前瞻调度声明；但引用现存文件路径（`tests/plans/…-uat-…` 模式）仍被第一个子模式拦住。
- **窗口宽度用 `{0,24}` 而非 `{0,6}`**：grep 在非 UTF-8 locale 下 `.` 与量词按**字节**计数——East-CJK 词 3 字节/字，6 字符窗口会漏拦（实证：「承接块书 AC 区间沿用」零命中）；24 字节同时兼容两种 locale，轻微放宽的误拦由口径边界人工裁决兜底。
- **台账条目计数用 `grep -w`**：`grep -c 'RUN-X-1'` 会把 `RUN-X-10/1x` 一并计入——`grep -cw 'RUN-X-1'` 才是精确挂链数。

## 6. 零实例残留

先用本机上下文文件里的已知实例名/账号/主机替换占位符再执行：

```bash
grep -rnE '<已知项目名>|<已知主机或IP>|<已知账号默认口令>|ou_[a-f0-9]|oc_[a-f0-9]|/home/<user>|/Users/<user>' "$DOC"
```

判据：零命中（任务书内允许出现目标项目自身名——它是被测对象）。

## 7. 乱码（替换字符）

```bash
grep -nP '\x{FFFD}' "$DOC"; file "$DOC"
```

判据：零命中且编码为 UTF-8。注意：**渲染层显示「乱码感」≠ 文件乱码**——先按 §1 至 §3 结构复核，可疑段 `od -c` 字节级复核后再定论。

`-P` 兜底注：PCRE 支持不是所有 grep 构建都有（macOS BSD grep、部分 BusyBox 报 `grep: -P unsupported`）——改用字节级固定串匹配：`grep -nF "$(printf '\xef\xbf\xbd')" "$DOC"`（U+FFFD 的 UTF-8 字节，任何 grep 构建可用；PS 版见 §13，天然支持 `\uFFFD`）。

## 8. 快照锚可复现

```bash
git cat-file -e <短号> && echo ANCHOR_OK        # 或 git rev-parse --verify <短号>^{commit}
```

判据：可解析才允许写入头部；不可解析的死锚（实证：某历史稿声称的 HEAD 短号在仓库两条历史线均不可解析）退化为「离线+日期」形态。

## 9. 密度机跑（采用「向」列时）

```bash
awk '/^#+ *3\. /{f=1;next} f && /^## (3\.5|4)([. ]|$)/{f=0} f' "$DOC" \
 | awk -F'|' '($2 ~ /AC-/) && ($3 ~ /^[[:space:]]*(正|负)/){v=$3; gsub(/ /,"",v); print v}' | sort | uniq -c
```

判据：负/(正+负) ≥ 30%。（与计数闸同款按 ID 字段收集——`^\| AC-` 行首形态会把 `| ⛔ AC-…` 前缀行漏计，负例密度被**低估**。）
无「向」列的书（窄域六列）跳过本闸，回执改为人工分类数并注明。
全域轮 D1 占比分档复核沿用 test-depth-dimensions.md 口径，人工核对覆盖表 D 行 vs 主表区间。

## 10. 台账→AC 全映射机验

```bash
# 台账镜像先抽出在管条目 ID 清单（按项目台账 ID 命名套用），再逐条回查任务书：
while read -r id; do printf '%s %s\n' "$id" "$(grep -cw "$id" "$DOC")"; done < ledger-ids.txt
```

判据：每条在管条目计数 ≥1（⛔ 挂链/复验锚），或该条目在交付的映射表里带显式 N/A+无 UI 面理由。0 计数且无理由=漏挂，返工。
实证注：某版全域书生成后对照实跑发现 6 条在管缺陷零挂链——此闸落地前靠生成者自觉，漏挂是最高频暗缺。

## 11. 路由→AC 全映射机验（全域轮强制）

```bash
# 侦察期保存路由清单 routes.txt（router/routes 文件或 DOM 扫描产物），逐条回查任务书：
while read -r route; do grep -qF "$route" "$DOC" || echo "MISSING: $route"; done < routes.txt
```

判据：输出为空。MISSING 逐条处理：补点 / 并入覆盖表 ⏭️/N/A+理由。
实证注：某版全域书侦察清单含报表页/热图页/队列监控页/扩展部件页/知识库管理页各一条，生成时五条全部无落点——路由扫描≠覆盖，清单必须对到 AC。

**环境注（grep 旗标组合兼容性）**：GNU grep 3.0（MSYS 发行版）在 `-c` 与 `-iF`/`-ic` 等多旗标组合时会 Aborted（实测 rc=134）——全集命令均已采用安全单旗标形态（`-oE`/`-nE`/`-qF`/`-w`）；需要带计数时用 `grep … | wc -l` 管道代理，勿自行打包多旗标。
**环境注（shell/平台兼容性）**：进程替换 `<(...)`（§2 首用）需要 bash/zsh 执行——POSIX sh（dash）不支持，报 `syntax error near unexpected token`；macOS 自带 BSD 工具链（BSD awk/sort/comm）对命令集内所用语法（`match`/`RSTART`/`gsub`、`-k1,1`、`comm -23`）语义与 GNU 一致（均属 POSIX），全闸无需改写。纯 Windows PowerShell 环境走 §13。

**口径边界（token 反查≠本闸）**：本闸输入=**生成期自挖的 routes.txt**；事后用路由 token 在既成书里反查**不能替代本闸**——域名级覆盖（治理/监控域把多个 admin 页并入域线）与 token 级覆盖是两种粒度，token 检索对域名级书会零命中造成假阴性（实证：两份对照书对四条实存路由 token 均无显式引用，但一份把对应页面收进了治理/监控域共 41 点，另一份只收了其中 3 页且无域名归属——token 反查区分不出这份差距，生成期清单逐条映射才区分得出）。

## 12. 三方口径一致（人工逐处核，2 分钟）

主表唯一数逐一对照**每个**数字落点：头部摘要 / 范围段 / §3 各域标题区间声明 / 深度覆盖表分母 / 完成准则 / 汇报模板头 / 产物树（如有）。任何一处对不上=返工；沿自旧版的旧数字**必须重跑实算**，禁复用。

## 13. Windows PowerShell 等价命令集（纯 PowerShell 环境兜底）

> 适用于无 bash 的 Windows 环境（agent 只有原生 PowerShell）。目标：**口径与 POSIX 版完全一致**——同样的区间锁、同样的第 2 字段 ID 归一、同样收门槛前缀行。两处平台语义差异：① .NET 正则 `{0,24}` 按**字符**计数（非字节），East-CJK 窗口天然精确（POSIX 版的 24 字节放宽在此自动升级）② Windows PowerShell 5.1 读 UTF-8 无 BOM 文件默认按 ANSI 解释会假乱码——所有读取显式 `-Encoding UTF8`/`[Text.Encoding]::UTF8`。`-cmatch` = 大小写敏感匹配，与 POSIX 正则默认一致。
>
> **前置块（每条书执行一次，定义变量；后续各闸复用）**：

```powershell
$DOC = "<任务书路径>"
$lines = [IO.File]::ReadAllLines($DOC, [Text.Encoding]::UTF8)
$start = $null; $endIdx = $lines.Count
for ($i = 0; $i -lt $lines.Count; $i++) {
  if ($lines[$i] -match '^#+ *3\. ') { $start = $i + 1; continue }
  if ($null -ne $start -and $lines[$i] -match '^## (3\.5|4)([. ]|$)') { $endIdx = $i; break }
}
$seg = @(); if ($null -ne $start -and $start -lt $endIdx) { $seg = $lines[$start..($endIdx - 1)] }
# 全文档先枚举宽候选并验证完整 token；非法引用必须报错，不得以不匹配代替。
$invalidIds = @()
foreach ($l in $lines) {
  [regex]::Matches($l, 'AC-[A-Z0-9]+-\d{3}[A-Z0-9]*') | ForEach-Object {
    if ($_.Value -cnotmatch '^AC-[A-Z0-9]+-\d{3}[A-Z]?$') { $script:invalidIds += $_.Value }
  }
}
if ($invalidIds.Count) { throw "INVALID_ID: $($invalidIds -join ', ')" }
$ids = @(); foreach ($l in $seg) { $f2 = ($l -split '\|')[1]
  if ($f2 -and ($f2 -cmatch 'AC-[A-Z0-9]+-\d{3}[A-Z]?(?![A-Z0-9])')) { $ids += $Matches[0] } }
```

**§1 计数/重号等价**（判据同 §1：组数=唯一数=文档内所有口径数字；Count 列全 1=零重号）：

```powershell
$ids | Group-Object | Sort-Object Name | Format-Table Count, Name -AutoSize
```

**§2 悬空等价**（判据：空输出）：

```powershell
$known = $ids | Sort-Object -Unique
$refs = $lines | ForEach-Object { [regex]::Matches($_, 'AC-[A-Z0-9]+-\d{3}[A-Z]?(?![A-Z0-9])') } |
  ForEach-Object { $_.Value } | Sort-Object -Unique
$refs | Where-Object { $_ -notin $known }
```

**§3 列完整等价**（判据：单一分布值 = 列数+1）：

```powershell
$seg | ForEach-Object { $f2 = ($_ -split '\|')[1]
  if ($f2 -and ($f2 -cmatch 'AC-[A-Z0-9]+-\d{3}[A-Z]?(?![A-Z0-9])')) { ($_ -split '\|').Count - 1 } } | Group-Object
```

**§4 指针清零等价**（判据：零命中，豁免同 §4）：

```powershell
$lines | Select-String -Pattern '见契约|以.*V[0-9]+\.[0-9]+|见主契约|以主契约为准|主契约原样要求|契约 §|以契约为准|全表见|执行期以.*当前版本|执行侧.*当前版本|当前版本为准'
```

**§5 派生依赖等价**（判据同 §5：输出为候选，按禁止/溯源/真实依赖语境判定，不再机械零命中；.NET 正则窗口按字符计数，无需字节放宽）：

```powershell
$lines | Select-String -Pattern 'tests/plans/.*uat|docs/reviews/UAT|执行单元|以.{0,24}(历史任务书|旧版任务书|块书).{0,24}(为基|为准|基底|承接|沿用)|(历史任务书|旧版任务书|块书).{0,24}(作为基底|为基底|承接|沿用)'
```

**§6 零实例等价**（先用本机上下文替换占位符再执行，判据零命中，豁免同 §6）：

```powershell
$lines | Select-String -Pattern '<已知项目名>|<已知主机或IP>|<已知账号默认口令>|ou_[a-f0-9]|oc_[a-f0-9]'
```

**§7 乱码等价**（判据：零命中；.NET 正则原生支持 `\uFFFD`，无 PCRE 依赖）：

```powershell
$lines | Select-String -Pattern '\uFFFD'
```

**§9 密度等价**（判据同 §9：负/(正+负) ≥ 30%；按第 2 字段收集，门槛前缀行不漏）：

```powershell
$seg | ForEach-Object { $c = $_ -split '\|'
  if ($c[1] -cmatch 'AC-[A-Z0-9]+-\d{3}[A-Z]?(?![A-Z0-9])' -and $c[2] -match '^\s*(正|负)') { $c[2].Trim() } } | Group-Object
```

**§10 台账映射等价**（判据同 §10：逐条 ≥1 或显式 N/A+理由；PS 无 `-w` 旗标，用边界环视等价词边界——`RUN-X-1` 不会误吞 `RUN-X-10`）：

```powershell
Get-Content ledger-ids.txt -Encoding UTF8 | ForEach-Object {
  $id = $_.Trim(); if (-not $id) { return }
  $re = "(?<![A-Za-z0-9_-])" + [regex]::Escape($id) + "(?![A-Za-z0-9_-])"; $n = 0
  foreach ($m in ($lines | Select-String -Pattern $re -AllMatches)) { $n += $m.Matches.Count }
  "{0} {1}" -f $id, $n
}
```

**§11 路由映射等价**（判据：空输出；`Contains`=大小写敏感，与 `grep -qF` 一致）：

```powershell
Get-Content routes.txt -Encoding UTF8 | ForEach-Object {
  $r = $_.Trim(); if (-not $r) { return }
  if (-not ($lines | Where-Object { $_.Contains($r) })) { "MISSING: $r" }
}
```

**§8/§12 无需 PS 版**：§8 是 git 命令，`git` 本身跨 shell 行为一致（直接在 PowerShell 里跑）；§12 是人工逐处核对。

## 14. 产品 AC 入口标签（仅 UI/MACHINE）

产品 AC 主表每行操作列必须以 `[UI]` 或 `[MACHINE]` 开头且互斥；DOC/AUDIT 不得出现在 AC 主表。UI 包含人类 GUI/TUI/日常 CLI/API；MACHINE 必须另附真实机器/外部系统契约与自身凭据证明。

```bash
awk '/^#+ *3\. /{f=1;next} f && /^## (3\.5|4)([. ]|$)/{f=0} f' "$DOC" \
| awk -F'|' '{ id=$2; if (match(id,/AC-[A-Z0-9]+-[0-9]{3}[A-Z]?/)) {
    op=$4; if ($3 !~ /^ *(正|负) *$/) op=$3; s=op; gsub(/^ +| +$/,"",s);
    n=gsub(/\[UI\]/,"&",s)+gsub(/\[MACHINE\]/,"&",s);
    if (n!=1 || s !~ /^\[(UI|MACHINE)\]/) printf "TAG? %s (tags=%d)\n", substr(id,RSTART,RLENGTH), n } }'
```

判据：产品 AC 标签无漏/双标；DOC/AUDIT 只在独立审计区。人工抽查 MACHINE 是否有真实机器调用方，「无 UI/管理员/习惯 CLI」一律拒绝。

PowerShell 等价：使用 §13 的完整 AC-ID 边界，操作列只匹配 `^\s*\[(UI|MACHINE)\]`；另用全文档候选检查防止非法子字母截断。

## 15. BLOCKED 六类归因与 ⏭️ 通道结论

1. 每条 BLOCKED 必须且仅有一个主归因：UNREACHABLE/UNAUTHORIZED/OBSERVABILITY_BLOCKED/DEFERRED/BUDGET_EXHAUSTED/OMITTED，并有对应证明。
2. UNREACHABLE 核有限努力证明，不要求换人/换时间；OBSERVABILITY_BLOCKED 核原始错误或结构化尝试记录；BUDGET_EXHAUSTED 核预算快照；DEFERRED 核接受人。
3. 涉及入口存在性/可达性的 ⏭️，以及涉及入口的 UNREACHABLE/UNAUTHORIZED，必须含「不存在/不可用+原因/未授权」结论；纯 AUDIT 不适用项只核存在性证据，预算/观测/其他归因不得伪造通道结论。
4. 造数返回码不得作产品判定依据。

可选辅助：先收集 BLOCKED 行，再逐行核六类归因及证明；不能只靠关键词数量判合格。正反样例见 §20 与 §23。

## 16. 场景-AC 双向闭合

**模型自检提示（不依赖脚本）：**

1. 读 §0 场景表：每个 S-ID（S0/G0 除外）是否有八要素？基本流必有；备选/异常按风险档有设计目标，无意义分支 N/A+可核理由，关键档强制异常不得缺，重复凑数同样不合格。
2. 读 §0.1 场景-AC 矩阵：每个 AC 是否至少挂 1 个 S-ID？每个 S-ID（S0/G0 除外）是否至少挂 1 个 AC？矩阵并集是否等于 §3 主表全集？差集逐条列出。
3. 读 §0.2 handoff 矩阵：跨角色/跨块场景是否有交接记录；单角色单块是否有 N/A+理由。
4. 读 §6 执行顺序：是否写“按 S1→Sn 场景执行，场景内基本流→备选流→异常流；域顺序只做 AC 回归”？写“按域顺序执行”=场景降级，返工。

**可选 grep 辅助（不是执行前提）：**

```bash
# 场景表 S-ID（示例：S1/G1/G-APV-IMPORT）；先截 §0 区间，避免匹配正文其他表
awk '/^## 0\./{f=1} /^## 1\./{f=0} f' "$DOC" | grep -nE '^\|[[:space:]]*(S|G)[0-9A-Z-]+[[:space:]]*\|'
# 场景-AC 矩阵内出现的 AC（只截 §0，避免正文其他表误命中）
awk '/^## 0\./{f=1} /^## 1\./{f=0} f' "$DOC" | grep -nE '^\|[[:space:]]*(S|G)[0-9A-Z-]+[[:space:]]*\|.*AC-[A-Z0-9]+-[0-9]{3}[A-Z]?'
```

> 注意：不要对全文裸 grep S/G 行——正文其他表也可能出现 `G1`/`S1` 字样，会误判。先截 §0 区间再匹配。

PowerShell 等价（先截 §0 区间）：

```powershell
$i0 = ($lines | Select-String -Pattern '^## 0\.' | Select-Object -First 1).LineNumber
$i1 = ($lines | Select-String -Pattern '^## 1\.' | Select-Object -First 1).LineNumber
$sec0 = $lines[($i0-1)..($i1-2)]
$sec0 | Select-String -Pattern '^\|\s*(S|G)[0-9A-Z-]+\s*\|.*AC-[A-Z0-9]+-\d{3}[A-Z]?(?![A-Z0-9])'
```

判据：模型逐条回答“有/无+证据”；任一缺项=场景模型不合格，报告只能 INTERIM。

## 17. 人类 UI 与真机器 MACHINE 主证

1. 人类 GUI PASS 要有判定时截图/DOM；TUI/CLI 要有终端读屏/输出；人类 API 要有调用方可见响应+最终业务结果。
2. GUI 有状态写点有实际控件、适用对话框确认/取消双路径和保存后回显；TUI/CLI 有终端输入/输出，人类 API 有调用方请求响应+最终业务结果；纯只读点不强制 confirm/保存回显。只验确认不得判双路径通过。
3. MACHINE PASS 必须有真实机器调用方的契约响应与内容断言；有状态变更才另核幂等和预期副作用，纯只读不得造副作用。
4. DOC/AUDIT 不判产品功能、不计需求验证分子、不参与 UAT 三态；纯 DOC 只能 coverage-audit。
5. 内部信号先记 TECH-OBS；无 persona 入口侧承诺失败关联，不得形成 UAT ❌。
6. 已见结果缺主证最高 PARTIAL；完全无法观察为 OBSERVABILITY_BLOCKED；事后补证不升级。

报告入口证据表分别列人类界面与机器契约；UI PASS=0 写 N/A+理由，大于 0 且主证不足 100% 判 INTERIM；任一入口 PARTIAL 均 INTERIM。

## 18. G0/SBTM 记录

**模型自检提示：**

1. 报告是否有 G0/S0 记录：时间盒（全域 20min/块级 10min）、四栏笔记（观察/疑问/缺陷/覆盖）、新增 B 点、缺陷、未覆盖项。
2. 是否只是完成准则里的 checkbox，没有实际记录；没有记录=未完成，报告只能 INTERIM。
3. 自由探索是否保持真实用户目标与合理数据语义，不做无意义输入/未授权破坏。

**可选 grep 辅助：**

```bash
grep -nE '\bG0\b|\bS0\b|自由探索|皱眉时刻|观察.*疑问.*缺陷.*覆盖' "$REPORT"
```

> PowerShell 等价必须加 `-CaseSensitive`，并用 `\bG0\b|\bS0\b` 词边界——默认不区分大小写时 `S0` 会误命中 ID/哈希子串（实证：`1PUKz...` 类 ID 被误判）。

```powershell
Select-String -Path $REPORT -CaseSensitive -Pattern '\bG0\b|\bS0\b|自由探索|皱眉时刻|观察.*疑问.*缺陷.*覆盖'
```

判据：有实际章节/表格记录且可复核；只有“已执行”或注释提名词语不合格。活体测：真实 `## G0/SBTM 记录` 应命中，哈希子串 `1PUKzS0abc` 不得命中；模式中只允许字面 `\b`，禁止 U+0008 控制符。

## 19. 执行顺序与场景日志

**模型自检提示：**

1. 任务书 §6 是否场景优先；报告是否有 `场景完成表`（S-ID / persona / 基本流 / 备选流 / 异常流 / 交接 / 挂靠 AC / 完成状态 / 证据）。
2. 每个场景是否有一句“作为 <persona>，我 能/不能 完成 <业务目标>”。
3. 是否先场景后 AC 回归；是否出现“先按 AC 表逐条打勾、最后补场景”的倒置。

**可选 grep 辅助：**

```bash
grep -nE '场景完成|场景级结论|按.*场景.*执行|基本流.*备选流.*异常流' "$DOC" "$REPORT"
```

PowerShell 等价：

```powershell
Select-String -Path $DOC,$REPORT -Pattern '场景完成|场景级结论|按.*场景.*执行|基本流.*备选流.*异常流'
```

判据：场景完成表缺项=场景执行未闭环；报告只能 INTERIM。

## 20. 闸回归样例

技能自带正/负样例，位于 `references/gate-samples/`：

| 样例 | 预期 |
|---|---|
| `positive-book.md` + `positive-report.md` | §16–§19 全部 PASS（不误伤） |
| `negative-book.md` + `negative-report.md` | §16–§19 命中缺口（仍能拦截） |
| `positive-id-subletters.md` | §1–§3 收集 002/002A/002B 三个唯一 ID，无重号/悬空 |
| `negative-id-reference.md` | 合法主表 002A+非法引用 002AB 必须输出 INVALID_ID |
| `positive-coverage-ledger.md` | 按表头读设计期第 4 列，尾格不误当桶位 |
| `positive-execution-book.md` + `positive-report-with-gaps.md` | §21–§23 通过；合法 FINAL-WITH-GAPS 不被误杀 |
| `negative-report-waiting.md` | 共同前置 WAITING=1，必须拒绝报告状态和 CLOSED |
| `negative-focus-book.md` | §24 拦截 DOC 冒充 AC、人类 CLI 误 MACHINE、域批吞并、技术信号直判缺陷、SINGLE_WRITER 携带并行接管 schema |
| `positive-focus-book.md` + `positive-focus-report.md` | 合法人类 CLI/API、真机器只读、独立 AUDIT、连续旅程、TECH-OBS 不误伤 |
| `positive-report-final-fail.md` | 全项有证据、UI PASS=0、存在 ❌，仍可 FINAL |
| `negative-parallel-missing-field.md` | PARALLEL_UNATTENDED 仅缺 reconciliationSource 也必须 FAIL |
| `negative-execution-book.md` | §21–§23 命中动态指针、沉默授权、全绿停止、全局化局部故障和无保底报告 |

改任何闸后先跑正样例再跑负样例，并实际执行 §23 全部桌面推演。样例为通用虚构内容，零实例残留；校验器先证明负例会被拦截，再采信正例输出。

## 21. 报告三态与保底闸（G5）

先核 UAT 身份：至少一条真实 persona 产品旅程有定性结论，DOC/AUDIT 不参与。所有模式核每个计划产品 AC 为 DONE/合法 BLOCKED；SINGLE_WRITER 另核逻辑四态已落 execution-log、无待续/独立可执行项且不创建并行写者契约，PARALLEL_UNATTENDED 才核 READY=0、WAITING=0、DONE+BLOCKED=产品 AC 总数；不满足则不得进入任何三态，返回调度或按全局支路先转 BLOCKED。随后读报告头 `报告状态` 按分支校验，禁止只看文件名或统一要求未触=0：

| 状态 | 必须满足 |
|---|---|
| FINAL | BLOCKED=0、PARTIAL=0、未裁定 ⏸️=0；UI PASS=0 时 N/A+理由，UI PASS>0 时对应主证 100%；计划产物完整；S+G=0 且 O=P |
| FINAL-WITH-GAPS | 不满足 INTERIM且存在 BLOCKED/未裁定 ⏸️/S/G；BLOCKED 有合法外部归因或 DEFERRED 接受记录；⏸️ 要件齐；S/G 有稳定 ID+证据+合法映射；无缺口不得本态 |
| INTERIM | 任一入口 PARTIAL，或 OMITTED/BUDGET_EXHAUSTED，或主证/计划产物缺口，或 ⏸️ 要件缺失，或 P=O+S+G 不成立，或 S/G 缺稳定 ID/证据/合法映射；允许 CLOSED |

```bash
grep -nE 'Final-Report-Gate' "$DOC"
grep -nE '报告状态|BLOCKED|OMITTED|BUDGET_EXHAUSTED|保底报告|CLOSED' "$REPORT"
```

判据：状态与条件一致；FINAL-WITH-GAPS 不得被「未触=0」误杀；INTERIM 不需伪装终期。模拟产物命名/断链失败时仍必须能签最小报告；「未过不得出报告」为负例。

## 22. 所选执行模式冻结（G2）

先读任务书头执行模式：

- SINGLE_WRITER：必须含共享状态机、逻辑 READY/WAITING/BLOCKED/DONE、长窗解除条件/resumeAfter/一次性收判/超时、预算、副作用、证据和最小断点；长窗原地续跑，不另出书/重复授权。并行相关词命中是候选：实际字段赋值、非空 schema、接管步骤或租约配置=FAIL；「禁止/不得/无/不包含」等否定说明安全。不得把禁令本身误判为携带 schema。
- PARALLEL_UNATTENDED：必须完整内联状态机、四队列、writerId、leaseEpoch=fencingToken、acquiredAt、leaseExpiresAt、heartbeatAt、预算、新鲜观察、trigger 迁移、收判包和报告字段。断点共 21 个字段，逐字段核：AC-ID、queue、scope、currentStep、intent、acceptanceSignal、dependencies、blockCause、releaseCondition、resumeAfter、attemptCount、retryLevel、lastObservation、budgetUsed、sideEffectIds、idempotencyKeys、dataBaseline、reconciliationSource、nextAction、evidenceRefs、unregisteredJudgements；缺任一、重复或总数不符即 FAIL。

```bash
grep -nE 'SINGLE_WRITER|PARALLEL_UNATTENDED|最小断点|leaseEpoch|fencingToken|triggerId' "$DOC"
grep -nE '执行期以.*当前版本|当前版本为准' "$DOC"
```

判据：只含所选模式；第二命令零命中。扩展内容来自生成侧模板并完整内联，最终任务书不得保留外部模板指针。轻量任务没有长窗时不得创建常驻巡逻。

## 23. 自主续跑、授权与局部阻塞

```bash
grep -nE '命令即签|沉默=GO|沉默即 GO|默认授予全集|默认等空闲重启|等空闲重启.*默认|全绿或 NO-GO|全绿.*前不得停|缺陷全部修复才可出报告|穷尽合法手段|换人/换时间|运行中禁止新增人机请求|打回生成侧重做|未过不得出报告' "$DOC"
grep -nE 'ACTION|AC|SCENARIO|DOMAIN|GLOBAL|READY|WAITING|BUDGET_EXHAUSTED|OBSERVABILITY_BLOCKED|迟发依赖|保底报告' "$DOC"
```

判据：第一组是**审查候选，不是机械零命中**；命中若处于「禁止/不得/负例」等否定语境则安全，肯定授权/停止语境才违规。第二组语义逐项存在且形成闭环。桌面推演：①单角色凭据失效后独立 AUDIT 继续 ②无法截图时下一独立项继续（即逻辑 READY） ③预算耗尽先把 READY/WAITING 逐项转 BUDGET_EXHAUSTED，双空后 INTERIM+CLOSED ④写者失联接管含 trigger 迁移或保底 ⑤风险未签署不撤销报告 ⑥自主全 ✅ 无缺口可 FINAL ⑦任一 MACHINE PARTIAL 必 INTERIM ⑧控件 S/G>0 不得 FINAL且必须列稳定 ID ⑨外部不可达 UI 的计划控件进 G 且可 WITH-GAPS，不因 O<P-S 自动 INTERIM ⑩完全无缺口不得 WITH-GAPS。任一出现无状态/无出口等待=闸失败。


## 24. 拟人身份、连续旅程与用户语言

逐项核验：

1. UAT 至少一条真实 persona 产品旅程，含动机/起点/连续步骤/交接/最终结果；DOC-only 只能 coverage-audit。
2. 产品 AC 仅 UI/MACHINE；PRODUCT/AUDIT/MIXED 先分层，AUDIT 独立记已核/未核且不落纯未覆盖；DOC 退出 AC/场景/需求分子/分母/UAT 三态；MACHINE 有真机器契约和自身凭据。人类 GUI 主界面可达时必须真实 GUI 事件，TUI/CLI/API 仅在该真实日常入口匹配目标时使用，不得图省事绕过。
3. AC 挂靠与场景完成分开；禁止 G-DOM/管理员场景默认吞并全部 AC；矩阵闭合不代替旅程完成。
4. 缺陷摘要含 persona+业务目标+真实入口+实际/预期结果；TECH-OBS 与 UAT 缺陷分离。
5. 报告首节为关键旅程定性结论，不新增旅程通过率 KPI；覆盖/证据/未触只作审计。
6. SINGLE_WRITER 默认含自主长任务内核且无并行接管 schema；PARALLEL_UNATTENDED 仅实际并行/接管时冻结扩展规程。
7. ACG 常驻正文不包含技能研发治理；其归 maintainer-guide，不参与任务生成。

负例必须分别覆盖：DOC 作为 AC、全 DOC FINAL、人类 CLI 标 MACHINE、GUI 主界面可达却图省事改用 API、G-DOM 关联全部 AC、控制台异常直接 UAT ❌、纯技术缺陷标题、SINGLE_WRITER 携带 fencingToken。任一漏拦=自检失效。

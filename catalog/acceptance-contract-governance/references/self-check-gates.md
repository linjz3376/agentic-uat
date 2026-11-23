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

**精确口径的由来（禁止回退裸 grep）**：裸 `grep '^| AC-' $DOC` 会把 ① 各 §3.x 子表头行（`| AC-ID | …`）与 ② §4/判据表等区域的 AC 引用行一并计入——实证：某书主表 191 因 5 行判据表引用行混入被误数成 195，反向曾把自家零重号书写成重号书。凡计数相关闸一律走区间管道。

## 1. 主表行数 / 唯一数 / 重号（一次出全）

**scan 函数（全部区间闸共用，含门槛前缀行收集）**：

```bash
scan() {
  awk '/^#+ *3\. /{f=1;next} f && /^## (3\.5|4)([. ]|$)/{f=0} f' "$1" \
  | awk -F'|' '{ id=$2; if (match(id, /AC-[A-Z0-9]+-[0-9]{3}/)) print substr(id, RSTART, RLENGTH) }'
}
```

（对 `$2` 字段取首个 `AC-域名-###` 记号——trim/emoji 变体差异一概免疫，表头行 `AC-ID` 与非 ID 单元格自动落榜，门槛前缀行不丢。）

```bash
scan "$DOC" | sort | uniq -c | sort -k1,1 -k2,2
```

判据：① 输出行数 = 唯一 AC 数 = 文档内所有口径数字（头部/范围段/§3 各标题/覆盖表分母/完成准则/汇报模板/产物树）② `uniq -c` 首列全为 `1` = 零重号；出现 `2`+ 的 ID 即重号清单。
Windows PowerShell 等价（口径不变）：见 §13 可复制命令集（区间锁=标题行号，ID 归一=第 2 字段 `-cmatch` 收集，同样覆盖门槛前缀行，勿写死行首形态）。
**双重反例实证（都曾被真书触发，勿回退）**：不剔表头=计数偏大（判据表引用行混入曾把 191 声明数成 195）；只认行首 `^| AC-`=计数偏小（门槛前缀行漏计曾把 136 数成 119）——片面的行首口径**两个方向都能错**。

## 2. 悬空引用（全文 AC 记号 ⊆ 主表 ID 集）

```bash
comm -23 \
 <(grep -oE 'AC-[A-Z0-9]+-[0-9]{3}' "$DOC" | sort -u) \
 <(scan "$DOC" | sort -u)
```

判据：输出为空。非空行=被引用但主表不存在的 ID（悬空/断号）。
（判据表等区域**有意**引用主表 ID 属合法——它们在右侧集合里，不进差集。）

## 3. 列完整性（主表行管道字段数 = 列数+1）

```bash
awk '/^#+ *3\. /{f=1;next} f && /^## (3\.5|4)([. ]|$)/{f=0} f' "$DOC" \
 | awk -F'|' '{ id=$2; if (match(id, /AC-[A-Z0-9]+-[0-9]{3}/)) print NF-1 }' | sort | uniq -c
```

判据：单一值 = 表头列数+1（全域轮七列含「向」→ 输出 `8`；六列窄域 → 输出 `7`）。出现第二分布=某行断列。

## 4. 版本指针清零（自包含铁律）

```bash
grep -nE '见契约|以.*V2\.[0-9]|主契约|契约 §|以契约为准|全表见' "$DOC"
```

判据：零命中（缺陷台账与环境密钥源豁免）。

## 5. 派生依赖清零

```bash
grep -nE 'tests/plans/.*uat|docs/reviews/UAT|执行单元|历史任务书|旧版任务书|以.{0,24}块书|块书.{0,24}(为基|为准|承接|沿用|基底)|承接.{0,24}块书|沿用.{0,24}块书' "$DOC"
```

判据：除头部 supersede/留痕声明行外零命中。
**口径边界（活体测必须覆盖三个面）**：
- 正样例（应命中）：「以块书为基继续生成」「承接块书 AC 区间沿用」「把两份块书列为执行单元」——依赖语境。
- 负样例（不应命中）：「另派块级深化任务书做增量深化」「全域书不替块书做深度」——前瞻调度，无依赖语义。
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

判据：零命中且编码为 UTF-8。注意：**渲染层显示「乱码感」≠ 文件乱码**——先按 §1~§3 结构复核，可疑段 `od -c` 字节级复核后再定论。

`-P` 兜底注：PCRE 支持不是所有 grep 构建都有（macOS BSD grep、部分 BusyBox 报 `grep: -P unsupported`）——改用字节级固定串匹配：`grep -nF "$(printf '\xef\xbf\xbd')" "$DOC"`（U+FFFD 的 UTF-8 字节，任何 grep 构建可用；PS 版见 §13，天然支持 `\uFFFD`）。

## 8. 快照锚可复现

```bash
git cat-file -e <短号> && echo ANCHOR_OK        # 或 git rev-parse --verify <短号>^{commit}
```

判据：可解析才允许写入头部；不可解析的死锚（实证：某历史稿声称的 HEAD 短号在仓库两条历史线均不可解析）退化为「离线+日期」形态。

## 9. 密度机跑（采用「向」列时）

```bash
awk -F'|' '($2 ~ /AC-/) && ($3 ~ /^[[:space:]]*(正|负)/){v=$3; gsub(/ /,"",v); print v}' "$DOC" | sort | uniq -c
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
$ids = @(); foreach ($l in $seg) { $f2 = ($l -split '\|')[1]
  if ($f2 -and ($f2 -cmatch 'AC-[A-Z0-9]+-\d{3}')) { $ids += $Matches[0] } }
```

**§1 计数/重号等价**（判据同 §1：组数=唯一数=文档内所有口径数字；Count 列全 1=零重号）：

```powershell
$ids | Group-Object | Sort-Object Name | Format-Table Count, Name -AutoSize
```

**§2 悬空等价**（判据：空输出）：

```powershell
$known = $ids | Sort-Object -Unique
$refs = $lines | ForEach-Object { [regex]::Matches($_, 'AC-[A-Z0-9]+-\d{3}') } |
  ForEach-Object { $_.Value } | Sort-Object -Unique
$refs | Where-Object { $_ -notin $known }
```

**§3 列完整等价**（判据：单一分布值 = 列数+1）：

```powershell
$seg | ForEach-Object { $f2 = ($_ -split '\|')[1]
  if ($f2 -and ($f2 -cmatch 'AC-[A-Z0-9]+-\d{3}')) { ($_ -split '\|').Count - 1 } } | Group-Object
```

**§4 指针清零等价**（判据：零命中，豁免同 §4）：

```powershell
$lines | Select-String -Pattern '见契约|以.*V2\.[0-9]|主契约|契约 §|以契约为准|全表见'
```

**§5 派生依赖等价**（判据同 §5：除头部 supersede/留痕行外零命中；.NET 正则窗口按字符计数，无需字节放宽）：

```powershell
$lines | Select-String -Pattern 'tests/plans/.*uat|docs/reviews/UAT|执行单元|历史任务书|旧版任务书|以.{0,24}块书|块书.{0,24}(为基|为准|承接|沿用|基底)|承接.{0,24}块书|沿用.{0,24}块书'
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
  if ($c[1] -cmatch 'AC-[A-Z0-9]+-\d{3}' -and $c[2] -match '^\s*(正|负)') { $c[2].Trim() } } | Group-Object
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

## 14. 入口标签覆盖

主表每行操作列（第 2 字段为 ID；有「向」列时操作在第 4 字段，否则第 3 字段）必须以 `[UI]`、`[MACHINE]` 或 `[DOC]` 开头，且全表逐行互斥：

```bash
awk '/^#+ *3\. /{f=1;next} f && /^## (3\.5|4)([. ]|$)/{f=0} f' "$DOC" \
| awk -F'|' '{ id=$2; if (match(id,/AC-[A-Z0-9]+-[0-9]{3}/)) {
    op=$4; if ($3 !~ /^ *(正|负) *$/) op=$3; s=op; gsub(/^ +| +$/,"",s);
    n=gsub(/\[UI\]/,"&",s)+gsub(/\[MACHINE\]/,"&",s)+gsub(/\[DOC\]/,"&",s);
    if (n!=1 || s !~ /^\[(UI|MACHINE|DOC)\]/) printf "TAG? %s (tags=%d)\n", substr(id,RSTART,RLENGTH), n } }'
```

判据：输出为空（每个 AC 行恰好一个入口标签，且位于操作列首）。出现 `tags=0`=漏标，`tags=2`=双标。
补充人工核（每域抽 2 点）：`[MACHINE]` 是否满足「无 UI 入口」或「机器 persona」；是否写了契约断言；有没有把有 UI 的能力写成 `[MACHINE]` 绕过页面判定。

PowerShell 等价：

```powershell
$seg | ForEach-Object {
  $c = $_ -split '\|'; $id = $c[1]
  if ($id -cmatch 'AC-[A-Z0-9]+-\d{3}') {
    $ac = $Matches[0]
    $op = if ($c[2] -match '^\s*(正|负)\s*$') { $c[3] } else { $c[2] }
    $n = ([regex]::Matches($op, '\[UI\]')).Count + ([regex]::Matches($op, '\[MACHINE\]')).Count + ([regex]::Matches($op, '\[DOC\]')).Count
    if ($n -ne 1 -or $op -notmatch '^\s*\[(UI|MACHINE|DOC)\]') { "TAG? $ac (tags=$n)" }
  }
}
```

## 15. BLOCKED/⏭️ 通道结论

每条 `BLOCKED` 或 `⏭️` 的证据栏必须含三选一结论：`不存在` / `不可用`（附原因）/ `未授权`。

```bash
grep -nE '\|[[:space:]]*(BLOCKED|⏭️)' "$REPORT" | grep -vE '不存在|不可用|未授权'
```

判据：输出为空（无「没找到」「未覆盖」式结论）。
补充：造数动作的返回码不得出现在判定依据里——用 API 造完数据后，判定仍须回到该点的入口侧观察。

PowerShell 等价：

```powershell
$lines | Where-Object { $_ -match '\|\s*(BLOCKED|⏭️)' -and $_ -notmatch '不存在|不可用|未授权' }
```

## 16. 场景-AC 双向闭合

**模型自检提示（不依赖脚本）：**

1. 读 §0 场景表：每个 S-ID（S0/G0 除外）是否有 `persona / 动机 / 业务目标 / 入口 / 基本流 / 备选流≥2 / 异常流≥2 / 交接`？流可 N/A+理由（占位凑数=不合格）；缺项逐条列出。
2. 读 §0.1 场景-AC 矩阵：每个 AC 是否至少挂 1 个 S-ID？每个 S-ID（S0/G0 除外）是否至少挂 1 个 AC？矩阵并集是否等于 §3 主表全集？差集逐条列出。
3. 读 §0.2 handoff 矩阵：跨角色/跨块场景是否有交接记录；单角色单块是否有 N/A+理由。
4. 读 §6 执行顺序：是否写“按 S1→Sn 场景执行，场景内基本流→备选流→异常流；域顺序只做 AC 回归”？写“按域顺序执行”=场景降级，返工。

**可选 grep 辅助（不是执行前提）：**

```bash
# 场景表 S-ID（示例：S1/G1/G-DOM-APV）；先截 §0 区间，避免匹配正文其他表
awk '/^## 0\./{f=1} /^## 1\./{f=0} f' "$DOC" | grep -nE '^\|[[:space:]]*(S|G)[0-9A-Z-]+[[:space:]]*\|'
# 场景-AC 矩阵内出现的 AC（只截 §0，避免正文其他表误命中）
awk '/^## 0\./{f=1} /^## 1\./{f=0} f' "$DOC" | grep -nE '^\|[[:space:]]*(S|G)[0-9A-Z-]+[[:space:]]*\|.*AC-[A-Z0-9]+-[0-9]{3}'
```

> 注意：不要对全文裸 grep S/G 行——正文其他表也可能出现 `G1`/`S1` 字样，会误判。先截 §0 区间再匹配。

PowerShell 等价（先截 §0 区间）：

```powershell
$i0 = ($lines | Select-String -Pattern '^## 0\.' | Select-Object -First 1).LineNumber
$i1 = ($lines | Select-String -Pattern '^## 1\.' | Select-Object -First 1).LineNumber
$sec0 = $lines[($i0-1)..($i1-2)]
$sec0 | Select-String -Pattern '^\|\s*(S|G)[0-9A-Z-]+\s*\|.*AC-[A-Z0-9]+-\d{3}'
```

判据：模型逐条回答“有/无+证据”；任一缺项=场景模型不合格，报告只能 INTERIM。

## 17. UI 主证与 UI 写点

**模型自检提示：**

1. 从 §3 取出全部 `[UI]` 行；对每个判 PASS 的点，检查证据是否含判定时截图或 `[DOM]` 文本/选择器。只有 API 响应/DB/日志的，最高 PARTIAL。
2. 对每个 `[UI]` 写点，检查操作列是否写实际控件标签、confirm/alert/prompt 文案、保存后回显；只写“填表保存”=不可执行。
3. 报告是否有 `UI 证据覆盖表`：UI AC 总数 / PASS 数 / 有判定时截图或 DOM 的 PASS 数 / 覆盖率；覆盖率<100% 必须写“UI PASS 证据覆盖 N/M”并判 INTERIM。
4. 是否有“事后补证”标注；事后截图不得升级原判定。

**可选 grep 辅助：**

```bash
# UI 行（需先按 §1 scan 口径截取 §3；此处仅示意）
grep -nE '\[UI\]' "$DOC"
# 报告中的 UI 证据覆盖表
grep -nE 'UI.*证据.*覆盖|UI PASS.*证据|判定时截图|事后补证' "$REPORT"
```

PowerShell 等价：

```powershell
$seg | Select-String -Pattern '\[UI\]'
Select-String -Path $REPORT -Pattern 'UI.*证据.*覆盖|判定时截图|事后补证'
```

判据：UI PASS 无 UI 证据=报告不合规；不得用“有响应原文”替代。

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
Select-String -Path $REPORT -CaseSensitive -Pattern 'G0|S0|自由探索|皱眉时刻|观察.*疑问.*缺陷.*覆盖'
```

判据：有记录且可复核；只有“已执行”一句话=不合格。

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

改任何闸后：先跑正样例验证不误伤，再跑负样例验证仍拦截——两个结果都符合预期，闸才可发布。样例为通用虚构内容（财务归档员/审计员），零实例残留；项目使用时可用本项目任务书替换负样例。

## 21. 报告称谓闸（G5 收官闸的产物侧机验，唯一命令载体）

```bash
grep -nE 'Final-Report-Gate' <任务书>                        # ≥1 命中——完成准则含「未触=0 方可出 final 报告」前置与 interim 出口条款
grep -nE '^#{1,6} .*(终期|收官)|收官 GO' interim-report-*.md   # 零命中——称谓类措辞载体=标题/章节名与判定措辞；降级记/闸注记等治理留痕文本豁免
grep -nE '带伤 GO' interim-report-*.md                       # 每行同行含「窗口」标注
grep -nE '未触=0' final-report.md                            # 头部核验行 ≥1 命中
```

判据：interim-report 禁「终期/收官/final」称谓、第七段标注「窗口口径/阶段判定」、第④段明示未触 N 点；「收官 GO/带伤 GO」字样仅在未触=0 的报告内合法。

## 22. 执行契约锚（G2「执行契约骨架入书」的产物侧机验）

```bash
grep -nE 'Execution-Contract' <任务书>                                  # ≥1 命中——头部含执行契约节
grep -cE '登记写者|状态唯一事实源|未登记判定清单' <任务书>               # ≥3——五字段中三核心字段占位在列
grep -nE '登记铁律.*(3\.3\.[0-9]+|v[0-9])' <任务书>                     # 零命中——出处指针不标版本号（快照纪律）
```

判据：契约节为**骨架**——登记写者/未登记判定清单等运行时字段一律占位符，禁填定值（由执行侧首个载体开工时填写、随登记划除）；契约文件路径按报告树 obs/ 推导，不写绝对实例路径；格式定义一律指针到执行侧登记铁律，冲突记偏差提交裁定，不静默沿用。


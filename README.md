# rime-tiger —— 虎码输入方案 · 元书移植版

本目录就是元书的一个「输入方案目录」，整体对应 `RimeUserData/rime-tiger/`。
目录内只有 rime 会读取的文件，没有多余的子目录。

- 基线：`humaIME/huma`（虎码官方 单字 tiger + 小词库 tigress）
- 补齐：`pinyin_simp` 拼音词库（tiger 拼音反查的字典来源）、`symbols.yaml`（符号/标点预设）
- 词表版本：虎码 `2024.04.08`
- 已移除：`rime-stroke`（笔画方案与词库，共 3.4 MB），理由与退路见「stroke 已移除」

## 30 秒装好

1. clone 下来，拿到 10 个 yaml（`README.md` / `LICENSE` / 点文件不要往手机上传）
2. 元书「WiFi 文件传输」→ 进 `RimeUserData` → 新建文件夹 `rime-tiger` → 把 10 个 yaml 拖进去
3. 「输入方案」→ 右上角 `...` → 「方案目录切换」→ 选 `rime-tiger`
4. 「RIME」页 → 「重新部署」（首次要编译 7.2 MB 词表，慢）
5. 方案列表出现「虎码官方单字」「虎码官方小词库」，打 `u` 出「的」即成

第 3 步为什么不能省、以及 `config_version` 那个能把方案列表吃掉的坑，见「部署」一节。

## 上游与许可（想 fork 或二次分发的人先读这段）

**本仓库 99.8% 的字节不是这里写的，是虎码作者的。**

- 上游仓库：<https://github.com/humaIME/huma>（最后一次推送 2024-04-08）
- 比对基准 commit：`421380f156172356c63ea3855f588c55dfe268ed`，取其中的 `rime/` 目录
- 词表版本：虎码 `2024.04.08`（写在 `tiger.dict.yaml` 的 `version` 字段里，与上游一致）

逐字节比对（Git Bash 实测输出，命令在下面，可自己复现）：

| 文件 | 上游（字节 / sha1 前 12 位） | 本仓库 | 结论 |
|---|---|---|---|
| `tiger.dict.yaml` | 1,443,582 / d4d0b1ddcd6b | 同 | **逐字节相同** |
| `tigress.dict.yaml` | 1,443,525 / c69a37efce08 | 同 | **逐字节相同** |
| `tigress_ci.dict.yaml` | 2,943,910 / 21593a3476c1 | 同 | **逐字节相同** |
| `tigress_simp_ci.dict.yaml` | 60,002 / fd1bbd1db740 | 同 | **逐字节相同** |
| `pinyin_simp.dict.yaml` | 1,266,165 / 212f2efe9ed6 | 同 | **逐字节相同** |
| `symbols.yaml` | 39,776 / 48d5cf02c17f | 同 | **逐字节相同** |
| `default.yaml` | 4,663 / 616a1de34ede | 5,056 / 150f7883608b | 补丁 4 |
| `pinyin_simp.schema.yaml` | 1,847 / 3558fc82dfcf | 1,797 / 6bcbe917f5c6 | 补丁 6 |
| `tiger.schema.yaml` | 2,236 / cdec1dad1d98 | 3,202 / 87d02816be5a | 补丁 3、5 |
| `tigress.schema.yaml` | 2,328 / b1f56e7d7e02 | 3,302 / 188e0ed365dc | 补丁 3、5 |

六个词表／符号文件与上游完全一致，合计 7,196,960 字节；本仓库自己改动的只有那 4 个配置文件，
合计 13,357 字节。复现命令（把上游 `rime/` 拉到 `huma_up/` 再逐个比 sha1）：
```bash
cd "C:/Users/pikachu/Desktop" && SHA=421380f156172356c63ea3855f588c55dfe268ed \
  && mkdir -p huma_up && for f in rime-tiger/*.yaml; do
       n=$(basename "$f")
       curl -s --max-time 40 "https://raw.githubusercontent.com/humaIME/huma/$SHA/rime/$n" -o "huma_up/$n"
       a=$(sha1sum "huma_up/$n" | cut -c1-12); b=$(sha1sum "$f" | cut -c1-12)
       if [ "$a" = "$b" ]; then echo "SAME  $n"; else echo "DIFF  $n  upstream=$(wc -c < "huma_up/$n")/$a  mine=$(wc -c < "$f")/$b"; fi
     done
```

**行尾：上游是 CRLF，本仓库原样保留。** `symbols.yaml` 与 `tiger.*` / `tigress.*` 全都用
`\r\n`（`pinyin_simp.*` 是 LF），上表的 sha1 记的就是这份字节，真机部署成功的也是这份字节。
所以 `.gitattributes` 里写了 `* -text`，关掉 git 的换行转换——否则 checkin 时 `\r` 被吃掉，
提交进去的词表就不再等于上游，clone 出来的 sha1 也和上面这张表对不上了（我第一次提交就踩了
这个，已修正）。别用编辑器的「转换行尾」功能去动这些文件。

**许可状态（2026-09-20 核查）**：

- `humaIME/huma` 仓库根目录只有 `README.md`、`fcitx5/`、`rime/`、`更新日志.txt`、
  `虎码输入法教程.pdf` —— **没有 LICENSE 文件**；GitHub API
  `GET /repos/humaIME/huma/license` 返回 `license: null`。
- 上游 README 全文 11 行，只有卖点、官网、PDF、网盘链接，没有任何「许可／授权／商用」字样。
- 官网 `tiger-code.com` 首页静态 HTML 里只能 grep 到「许可证号：」——那是 ICP 备案号；
  `/docs/*` 是客户端渲染，静态抓取看不到正文，**所以「官网上是否另有授权声明」这一条我没能证实，
  也谈不上证伪**，请以作者的说法为准。

没有 LICENSE 的默认法律状态是**保留所有权利**。因此本仓库的立场写清楚：

- 虎码词表（上表中标「逐字节相同」的 6 个文件）的著作权归虎码作者／`humaIME`，
  **本仓库不对其主张任何权利，也无权授予他人权利**。放进来只为个人在元书上的部署复现，
  不构成任何再分发许可。
- `LICENSE` 文件里的 MIT **只适用于本仓库自己写的部分**：那 4 个配置文件的改动、README 的文字。
- 权利人要我删除或更换许可，开个 issue 说明即可，我会照办。
- 需要干净再分发的话，走「只放自有改动 + 一条按 commit 拉上游的脚本」这条路（上面那段命令
  就是脚本雏形），不要把本仓库的 yaml 直接打包给别人。

## 为什么必须有 pinyin_simp，stroke 已移除

`tiger.schema.yaml` / `tigress.schema.yaml` 的拼音反查写作
`reverse_lookup: { dictionary: pinyin_simp }`，而元书的 `RimeSharedSupport` 不含 PC 内置方案，
所以 **`pinyin_simp` 的两个文件必须一起打包**（它是 tiger 反查的词典来源，缺了反查直接没有）。

`stroke` 本来只是 `pinyin_simp` 自己那层笔画反查所需。实测从 tiger 里进不去那一层（判定见
「实测结论」），而 `ReverseLookupTranslator::Initialize()`
（`src/rime/gear/reverse_lookup_translator.cc` 103-140 行）只按 `reverse_lookup/dictionary`
装一个词典、再按 `reverse_lookup/target`（缺省 `translator`）取目标词典，
**从不构造目标方案的 Engine**，所以 tiger 的拼音反查根本不会碰 `pinyin_simp` 的
`dictionary: stroke`。据此删掉 stroke 的两个文件（3.4 MB），并清掉 `pinyin_simp` 里的悬空
引用（补丁 6）。

librime `src/rime/lever/deployment_tasks.cc` 的 `WorkspaceUpdate::Run`（master 上 209 / 212 行）
对两种缺失的处理并不相同，看日志时按这个区分：

- 缺 `schema_list` 里列出的 `tiger` / `tigress` → `LOG(ERROR) missing input schema: <id>`，
  计入 failure，部署失败。
- 缺被依赖的方案（补丁 6 之前的 `stroke`）→ `LOG(WARNING) missing input schema;
  skipped unsatisfied dependency: <id>`，部署仍算成功，只是那一层是空的。
  补丁 6 之后本包不再声明指向 stroke 的依赖，正常情况下不该再出现这条 WARNING。

## 文件清单

| 文件 | 字节 | sha1(12) | 处理 |
|---|---|---|---|
| `default.yaml` | 5,056 | 150f7883608b | 补丁 4 |
| `symbols.yaml` | 39,776 | 48d5cf02c17f | 原样 |
| `tiger.schema.yaml` | 3,202 | 87d02816be5a | 补丁 3、5 |
| `tiger.dict.yaml` | 1,443,582 | d4d0b1ddcd6b | 原样 |
| `tigress.schema.yaml` | 3,302 | 188e0ed365dc | 补丁 3、5 |
| `tigress.dict.yaml` | 1,443,525 | c69a37efce08 | 原样 |
| `tigress_ci.dict.yaml` | 2,943,910 | 21593a3476c1 | 原样 |
| `tigress_simp_ci.dict.yaml` | 60,002 | fd1bbd1db740 | 原样 |
| `pinyin_simp.schema.yaml` | 1,797 | 6bcbe917f5c6 | 补丁 6 |
| `pinyin_simp.dict.yaml` | 1,266,165 | 212f2efe9ed6 | 原样 |

合计 7,210,317 字节（6.88 MiB），10 个文件。核对方式（Git Bash，从输出里逐行比对字节数与 sha1）：

```bash
cd "C:/Users/pikachu/Desktop/rime-tiger" && python -c "
import glob,os,hashlib
t=0
for f in sorted(glob.glob('*.yaml')):
    b=open(f,'rb').read(); t+=len(b)
    print(f'{len(b):>9}  {hashlib.sha1(b).hexdigest()[:12]}  {f}')
print(f'{t:>9}  TOTAL')"
```

> `.dict.yaml` 用 PyYAML 解析会报 `found character '\t' that cannot start any token`——
> 这是**正常的**：词表正文（`...` 之后）是 `字<TAB>码<TAB>权重` 的行式格式，librime 按行读，
> 不走 YAML 解析器。TAB 只有出现在 **schema / default 的缩进**里才是错误（见补丁 3）。

## 移植改动（当前生效 4 处：补丁 3、4、5、6，均在文件里留了中文注释）

3. **`tiger.schema.yaml` / `tigress.schema.yaml`**：上游文件第 24 行 `states:` 后面有一个 TAB。
   PC 的 yaml-cpp 容忍，严格 YAML 解析器会直接报
   `found character '\t' that cannot start any token`。已去掉 TAB 与行尾空格，语义不变。

4. **`default.yaml`**：`config_version: "0.36"` → `"99.99"`。元书
   `RimeSharedSupport/default.yaml` 的内置版本实测为 **0.50**（2026-09-20，元书 v3），
   高于本文件的 0.36，librime 于是把本文件当作过期副本移进 `trash/`（详见「部署」一节的
   机制说明），`schema_list` 随即退回内置值——这就是真机上「目录一切换就只剩 build/trash
   两个文件夹、方案列表里没有虎码」的原因。抬到 99.99 后对任何 0.x / 1.x / 2.x 内置值
   都判为「不低于」，只有 ≥100.0 才会再次触发回收。

5. **`tiger.schema.yaml` / `tigress.schema.yaml` 的死配置全部注释掉**（本条在 2026-09-20
   改过两次，最新结论如下，早先「属性栏只剩两个开关」的说法是错的，见「实测结论」）：
   - 仓「自定义键盘」时代的 `助记关/助记开`（option 名 `_keyboard_default` /
     `_keyboard_defaultzj`）：元书已移除自定义键盘，Write2026 皮肤也不读这两个 option。
   - `options: [gbk, gb2312, utf8]` 三档字集开关，以及 `engine.filters` 里的
     `charset_filter@gbk` / `@utf8` / `@gb2312` 三行：现成 librime 不支持带参数的
     `charset_filter`。核对过 1.7.3 / 1.11.2 / 1.13.0 三个 tag，逐字一致
     （`src/rime/gear/charset_filter.cc`）：

     ```cpp
     an<Translation> CharsetFilter::Apply(an<Translation> translation,
                                          CandidateList* candidates) {
       if (name_space_.empty() &&
           !engine_->context()->get_option("extended_charset")) {
         return New<CharsetFilterTranslation>(translation);   // 只有无参时才过滤
       }
       if (!name_space_.empty()) {
         LOG(ERROR) << "charset parameter is unsupported by basic charset_filter";
       }
       return translation;                                    // 带参数：原样放行
     }
     ```

     `charset_filter@utf8` 里的 `@utf8` 正是 `name_space_`——`src/rime/ticket.cc:19-23` 用
     `klass.find('@')` 把参数切出来赋给 `name_space`。所以这三行既不过滤、还会在每次出候选时
     刷一条 ERROR。上游 `translator/enable_charset_filter: false` 也没打开，所以**整份配置
     从来没有限制过字集**，全字库照出。开关本身留着也没有 UI 可切（见「实测结论」第一条），
     故一并注释。想恢复上游外观只需取消注释。
     （librime master 分支后来把判断放宽成 `name_space_ == "filter"` 也算默认命名空间，
     对带 `@gbk` 之类参数依旧不过滤，结论不变。）

6. **`pinyin_simp.schema.yaml`**：删掉 `dependencies: [stroke]`、整段
   `reverse_lookup:`（`dictionary: stroke`）、`engine.translators` 里的
   `reverse_lookup_translator`、以及 `recognizer/patterns/reverse_lookup`。
   这四处互相引用、共同构成 pinyin_simp 自己的笔画反查；stroke 文件移除后它们就是悬空引用
   （留着会稳定产生一条 `skipped unsatisfied dependency: stroke` WARNING）。删完后本包
   部署日志应当干净。pinyin_simp 在这里只剩一个身份：**tiger 反查用的拼音词库**。

**历史（补丁 1、2，已随 stroke 移出而作废）**：

1. `stroke.dict.yaml`：`use_preset_vocabulary: true` → `false`。全目录只有这一处依赖八股文
   `essay.txt`，而元书不带它；librime `dict_settings.cc:55` 的缺省值就是 false，且该表
   `max_phrase_length: 1` 不生成词汇，所以这个开关对笔画表没有实际作用。
2. `stroke.schema.yaml`：删掉自带的 `dependencies: [luna_pinyin]`、`abc_segmentor/extra_tags`、
   整段 `reverse_lookup`（字典是 luna_pinyin）和 `recognizer/patterns/reverse_lookup`。
   元书没有 luna_pinyin，保留会让 stroke 部署失败。

这两个文件现在放在同级目录 `../rime-tiger.stroke.removed/`（`stroke.schema.yaml` 2,854 字节
`10046d6636b2`、`stroke.dict.yaml` 3,396,448 字节 `f361ed390fd7`），要用就整份搬回本目录。

除以上各处外，所有文件与上游逐字节一致（上游仓库与比对基准 commit 见「上游与许可」）。

## 部署

先理解元书的目录机制，否则文件放对了也不会生效：

- 元书把 `RimeUserData` 下的**任意子目录**作为 rime 引擎的 `user_data_dir`，但同一时刻只有
  **一个**「当前方案目录」生效，**默认是 `RimeUserData/rime-ice`**。
  新建子目录不会被自动选中，里面的 `default.yaml` / `*.schema.yaml` 一律不读。
- librime `AutoUpdate` 按 `default.yaml` 的 `schema_list` 逐个解析路径，**单根、非递归**。
  所以目录叫什么名字对引擎毫无意义（只要它被设为当前方案目录），而放在
  `RimeUserData` **根目录**的 yaml 同样不会被读取。
- **版本回收陷阱**（本方案踩过的坑）：部署时 `ConfigFileUpdate("default.yaml", "config_version")`
  会调 `TrashDeprecatedUserCopy`，用 `GetString` 取两边的 `config_version`，交给
  `CompareVersionString` 按 `.` 分段逐位比大小（不是字符串比较，也不是整数）。
  只要内置值更大，本目录的 `default.yaml` 就被 `rename` 到 `trash/`，接着从
  **内置** default.yaml 重编出 `build/default.yaml`，`schema_list` 退回元书自带列表 →
  方案列表里没有虎码。日志里对应 `deprecated user copy of 'default.yaml' is moved to`（WARNING）。
  两点注意：**删掉 `config_version` 这一行等于版本 0，必然被回收**；`trash/` 里的文件
  不会被自动放回，修好后必须重新上传。

### 方案一：WiFi 传输（推荐，目录名可控）

1. 元书 → 「WiFi 文件传输」，电脑浏览器打开给出的网址
2. 进入 `RimeUserData`，新建文件夹 `rime-tiger`，进入后把本目录里的 10 个 yaml 全部拖进去
   （只拖 yaml，不要放本 README；确认没有出现 `rime-tiger/rime-tiger/` 双层）
3. 「输入方案」→ 右上角 `...` → 「方案目录切换」→ 选 `rime-tiger`
   （WiFi 网页端也可长按/右键目录 → 「设置为用户方案目录」，需元书 ≥ 1.8.0）
4. 「RIME」页面 → 「重新部署」。词表源文件合计约 7.2 MB，首次编译耗时较长
5. 回键盘扩展切出再切回，让新部署的 `*.bin` 生效

> 切到 `rime-tiger` 后，方案列表只剩虎码两项，`rime-ice` 下的雾凇等方案不再出现；
> 想回去只需在「方案目录切换」里选回 `rime-ice`。皮肤是全局设置不受影响，
> 但各方案目录的 `build/`、用户词库与学习记录互相独立。

### 已被回收后的恢复（`trash/default.yaml` 已出现时）

现象就是：`rime-tiger/` 根下没有 `default.yaml` 了，多出 `build/`（里面有个 `default.yaml`，
是从内置配置编出来的）和 `trash/default.yaml`。按序处理：

1. 把本仓库新的 `default.yaml`（`config_version: "99.99"`）单独上传到 `rime-tiger/` 根
2. 删掉 `trash/` 与 `build/` 两个目录（`trash/` 里的文件不会被自动放回；删 `build/` 只为强制重编）
3. 「RIME」页 →「重新部署」
4. 核对：`rime-tiger/` 根下 `default.yaml` **仍然存在**，没有新的 `trash/default.yaml`，
   方案列表出现「虎码官方单字」「虎码官方小词库」
5. 内置版本参考值：元书 `RimeSharedSupport/default.yaml` 的 `config_version` 实测为 **0.50**
   （2026-09-20），99.99 的余量足够；元书大版本更新后可再核一次

### 从含 stroke 的旧版本升级（补丁 5、6 之后）

手机上 `RimeUserData/rime-tiger/` 若还是 12 个文件的旧版，按序处理：

1. 「WiFi 文件传输」进该目录，删掉 `stroke.schema.yaml` 与 `stroke.dict.yaml`（省 3.4 MB）
2. 覆盖上传 3 个改过的文件：`pinyin_simp.schema.yaml`、`tiger.schema.yaml`、`tigress.schema.yaml`
3. 若 `default.yaml` 里临时加过 `- schema: stroke`，用本仓库的 `default.yaml` 覆盖
   （`schema_list` 只有 `tiger` / `tigress` 两项，`config_version: "99.99"` 不要删）
4. 「RIME」→「重新部署」
5. 核对：日志**既无 ERROR、也没有** `skipped unsatisfied dependency`；`u`→的、`je`→他、
   `` ` ``+`xiang` 反查三项照常

### 方案二：压缩包导入

把同级目录下的 `rime-tiger.zip` 传到手机，元书「输入方案」→ `...` → 「导入方案」选中它
（元书 1.3.0 起把 zip 一律按方案包解压到用户方案目录下），
再「方案目录切换」选 `rime-tiger`。
> 导入后先看一眼有没有变成 `rime-tiger/rime-tiger/` 双层；真出现双层就把方案文件上移一层。
> zip 顶层已含 `rime-tiger/` 一层，且不含 `README.md`。重新打包（Git Bash 验证过；
> 注意 `zip` 对已存在的归档是**追加**而不是覆盖，所以必须先删）：
>
> ```bash
> cd "C:/Users/pikachu/Desktop" && rm -f rime-tiger.zip \
>   && zip -r rime-tiger.zip rime-tiger -x "rime-tiger/README.md" \
>   && unzip -l rime-tiger.zip
> ```
>
> 打完逐字节复核（`unzip -l` 只看名字和长度，这条才是真比对；两条都在 Git Bash 验证过）：
>
> ```bash
> cd "C:/Users/pikachu/Desktop" && rm -rf /tmp/zipchk \
>   && unzip -q -o rime-tiger.zip -d /tmp/zipchk \
>   && diff -r /tmp/zipchk/rime-tiger rime-tiger -x README.md && echo "SAME (except README)"
> ```
>
> 当前归档内容：`rime-tiger/` 一层 + 10 个 yaml、共 7,210,317 字节，无 `stroke.*`、无 `README.md`。

## 部署后请核对

- [x] 部署日志无 ERROR（`文件管理` → 查看日志）
- [x] 方案列表出现「虎码官方单字」「虎码官方小词库」
- [x] 打 `u` 应出「的」，`je` 应出「他」（四码内自动上屏由 `speller/auto_select` 决定）
- [x] 反查：中文 26 键盘 A 键上划出 `` ` `` → 打拼音 → L 键上划出 `'` → 候选注释显示虎码码
- [ ] 首次部署耗时与内存（10 个 yaml 合计约 7.2 MB）
- [ ] 补丁 5、6 上传后复测：日志无 ERROR，且**不再出现** `skipped unsatisfied dependency`；
      上述四项照常；打几个字后翻运行日志，确认没有
      `charset parameter is unsupported by basic charset_filter`（补丁 5 已把那三行注释掉，
      如果还看到，说明手机上仍是旧文件）

> 实测记录：2026-09-20 元书 v3 真机部署成功（12 文件旧版）。日志无 ERROR，方案列表两项齐全，
> `u`→的、`je`→他 正确；`` ` ``+`xiang` 反查出「想 eqh eqhx／向 tm tmdk／像 jwx／象 wx／相 e…」，
> 编码注释与〔拼音〕提示都正常，6 字母的 `zhuang`、`chuang` 同样正常。
> 补丁 4 之前是 `default.yaml` 被移入 `trash/`、方案列表为空。
>
> 同日反查嵌套两项测试：A —— tiger 拼音反查段内再上划 `` ` ``，笔画层没有出现（判定不成立）；
> B —— 临时把 `stroke` 加进 `schema_list` 切到「五筆畫」，打 `hspnz` 得编码栏「一丨丿丶乙」、
> 候选「札／杤／杁／权／初…」，编码栏与注释都符合预期（stroke 表与 `xlit` 都正常，
> 个别候选显示 `?` 是皮肤字体缺字形）。B 只是验证 stroke 自身健康；本仓库的 `default.yaml`
> 从头到尾只列 tiger / tigress，手机上那份临时加过的 `- schema: stroke` 按「升级」一节第 3 步覆盖掉。

列表为空时按日志关键字定位：

| 日志 | 含义 | 处理 |
|---|---|---|
| `deprecated user copy of 'default.yaml'` | 本目录 `default.yaml` 的 `config_version` 低于 `RimeSharedSupport` 内置副本，被移入 `trash/`，`schema_list` 随之丢失 | 用本仓库的 `default.yaml`（已抬到 `99.99`）重新上传；**不要删这一行**，缺省即版本 0 |
| `missing input schema: tiger`（ERROR） | `tiger.schema.yaml` 不在**当前方案目录**根下，或方案目录没切过来 | 见「部署」第一节的目录机制 |
| `missing input schema; skipped unsatisfied dependency`（WARNING） | 缺被依赖的方案，部署成功但那一层为空。本包只依赖 `pinyin_simp`（补丁 6 之后不再依赖 stroke） | 补齐 `pinyin_simp.schema.yaml` + `pinyin_simp.dict.yaml`；若仍写着 `stroke`，说明手机上还是旧版 `pinyin_simp.schema.yaml` |
| `found character '\t' that cannot start any token` | YAML **缩进**里出现 TAB（注意：`.dict.yaml` 正文里的 TAB 是格式要求，不算错） | 定位报错的 schema / default 文件改回空格 |
| `charset parameter is unsupported by basic charset_filter`（ERROR，运行期） | 手机上仍是旧版 schema，`charset_filter@gbk/@utf8/@gb2312` 三行还在 | 按「升级」一节重传 `tiger.schema.yaml` / `tigress.schema.yaml`；补丁 5 已把这三行注释掉 |

## 实测结论与已知限制

- **元书不渲染 rime 的 `switches`（修正早先的说法）**：真机属性栏那一行只有皮肤自己的按钮
  （表情／短语／剪贴／简体／收起），从来没见过「中文/西文」或「GBK/GB2312/UTF-8」这类
  rime 开关。本 README 早前「属性栏现在只剩两项开关」是**我推测错的**，已删除。
  结论：补丁 5 纯属清理，不改变任何可见行为；要改开关语义只能改 YAML 再重新部署。
  唯一保留的 `ascii_mode` 走 rime 内部逻辑（本方案 `ascii_composer` 把 Shift_L 配成
  `commit_code`、Shift_R 配成 `commit_text`），不需要属性栏也能用。
- **字集过滤怎么做（真想只出常用字看这条）**：本包 `engine.filters` 现在只有 `uniquifier`。
  现行 librime 的 `charset_filter` 只有一个行为——**无参**且 `extended_charset` 为关时，
  过滤掉 `is_extended_cjk()` 命中的字（Ext A～J、兼容汉字等，见 `charset_filter.cc`）。
  要启用就在 `tiger.schema.yaml` 里加一行 `- charset_filter`，并取消
  `switches` 中 `- name: extended_charset` / `states: [常用, 增廣]` 两行的注释；
  另一条路是 `translator/enable_charset_filter: true`（`table_translator.cc` 同样只看
  `extended_charset`），它作用在查词/组句阶段。GBK、GB2312 这种**分级**过滤 basic 版做不到。
- **反查与 `auto_clear: max_length` 的冲突**：`speller` 限 4 码，而拼音串常超过 4 个字母。
  实测 5 字母（`xiang`）与 6 字母（`zhuang`、`chuang`）反查都正常出候选，说明反查段由
  `recognizer/patterns/reverse_lookup` 接管、不受 `max_code_length` 截断。已排除。
- **笔画反查不能嵌套进拼音反查（真机 A 测 + 源码判定）**：两层的前缀键**同为** `` ` ``，
  而 tiger 的 `recognizer/patterns/reverse_lookup` 是 `` ^`[a-z]*'?$ `` —— 段中再按 `` ` ``
  直接不匹配，该段掉出反查标签；加上 `ReverseLookupTranslator` 只借目标方案的**词典**出字、
  不构造目标方案的 Engine（见「为什么必须有 pinyin_simp」），不会挂上它自己的 `reverse_lookup`。
  实测：`` ` ``+`xiang` 后再上划 `` ` ``，笔画层没有出现。结论**不成立**，
  想要「拼音＋笔画」三重注解得走 Lua（ywxt / wallleap 路线）。
- **stroke 已移除**：`stroke.schema.yaml` + `stroke.dict.yaml`（3,399,302 字节，约占旧包三成）
  已从本目录移走，暂存在 `../rime-tiger.stroke.removed/`；连带的悬空引用由补丁 6 清掉。
  移除前 B 测过 stroke 本身是健康的（切到「五筆畫」打 `hspnz` → 编码栏「一丨丿丶乙」，
  候选「札／杤／杁／权／初…」，`xlit` 与康熙部首 `comment_format` 都正常；个别候选显示 `?`
  是皮肤字体缺字形，不是配置问题）。要恢复笔画输入：把那两个文件搬回本目录、还原补丁 6 删掉的
  四处引用，并在 `default.yaml` 的 `schema_list` 里正式加 `- schema: stroke`。
- **没有整句**：tiger/tigress 都是 `enable_sentence: false`，不依赖 octagram/`.gram`。
- **没有字根拆分反查**：那类三重注解靠 Lua（ywxt / wallleap 路线），本基线不含。

## 回到上游

- 恢复 stroke（笔画方案）：把 `../rime-tiger.stroke.removed/` 的两个文件搬回本目录，
  还原补丁 6 在 `pinyin_simp.schema.yaml` 里删掉的四处引用，并在 `default.yaml` 的
  `schema_list` 加 `- schema: stroke`。
- 恢复成 PC 完全一致的 stroke 行为：再把 `stroke.dict.yaml` 的 `use_preset_vocabulary`
  改回 `true`，并额外放一份 `essay.txt` 进本目录（元书不带八股文）。
- 恢复上游的死配置：取消补丁 5 注释掉的 `options: [gbk, gb2312, utf8]` 与三行
  `charset_filter@...`（行为不变，只会多运行期 ERROR）。

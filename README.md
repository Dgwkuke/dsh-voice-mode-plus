# dsh-voice-mode-plus

[dsh-voice-mode](https://github.com/qishuilalala/dsh-voice-mode) 的增强分支：在保留全双工语音（本地流式识别、按句朗读 + 实时字幕、开口打断）的前提下，修掉了三处实际听感问题，并把插件身份独立出来，便于与上游共存或单独分发。

- 朗读断句：句间按标点留停顿，不再 20ms 硬切
- 朗读语调：问句升调、感叹句提音量，句子之间不再一个平调到底
- 本地 TTS：小数点回归（`3.14` → 「三点一四」，不再被替换成顿号）

插件标识全部独立：路由 `/voice-mode-plus`、设置文件 `voice-mode-plus.settings.json`、locale 命名空间 `voice-mode-plus`，可与上游 `dsh-voice-mode` 同时安装。

## 安装

仓库已提交构建产物 `lib/`，从 git 直接安装即可，无需本地构建：

```sh
dsh plugin --profile <profile> add github:Dgwkuke/dsh-voice-mode-plus
```

装完重启宿主（或重新加载 profile）。首次进入语音模式会沿用 `~/.cache/dsh-voice-mode/models` 下的模型缓存；没有缓存时自动下载（ASR ≈390MB；本地 TTS 另计）。

> 也可以从本仓库根目录本地安装：`dsh plugin --profile <profile> add /path/to/dsh-voice-mode-plus`

## 与上游的差异

| 项目 | 上游 dsh-voice-mode 0.7.34 | 本分支 |
| --- | --- | --- |
| 分句终止符 | 句号、问号、感叹号、**分号、省略号**、换行 | 句号、问号、感叹号、换行（分号/省略号留作句内停顿） |
| 短句 | 每句独立合成，各自起调收尾 | 同批相邻短句合并后再合成 |
| 句间停顿 | 固定 20ms | 按句末标点：句号 320ms / 问号感叹号 300ms / 省略号 420ms / 逗号 140ms |
| Edge 语调 | 只传语速 | 问句 `pitch +8%`、语速 ×0.98；感叹句 `volume +10%` |
| 本地引擎小数点 | 数字后的点一律替换为顿号（`3.14` → `三、一四`） | 小数点显式转「点」，编号（`1. `）与句末句点分别处理 |
| 插件身份 | `dsh-voice-mode`，路由 `/voice-mode` | `dsh-voice-mode-plus`，路由 `/voice-mode-plus`，其余标识同步独立 |

## 与上游共存 / 切换

两套插件注入同一个麦克风槽位，同时启用会出现重复按钮。只保留其一时，在 profile 的 `cordis.patch.yml` 末尾加一条（本分支的条目 id 是 `voice-mode-plus`）：

```yaml
- id: voice-mode
  disabled: true
```

## 验证情况

已在 macOS + dsh `0.2.1-rc.5` 实测：

- `pnpm run typecheck` 通过
- 单测：segmenter 33、verify-client 41、host-security-behavior 60、tts-queue-engine 27、emotion 16、tts-playback 8（均通过）
- 本地 VITS 合成 → SenseVoice 回环：`3.14` 识别为「三点十四」；旧逻辑同音频识别为「在十四」
- Edge 云端试听接口正常返回 MP3

未验证：Windows / Linux、其它 dsh 版本、其它麦克风与声卡组合；Kokoro 本地引擎在 Electron 宿主内需外部 Node（上游同款限制）。

## 数据与隐私

ASR 全程本地；朗读默认走 Edge 云端（文本会发送给微软），可在设置里切到本地 VITS（纯中文）或 Kokoro（中英混读）完全离线。模型下载带 SHA256 校验与域名白名单。

## 许可

MIT，继承上游 [dsh-voice-mode](https://github.com/qishuilalala/dsh-voice-mode)。第三方组件许可见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。

上游完整的安装、设置、手势与架构文档见 [README.upstream.md](README.upstream.md)（英文版 [README.upstream.en.md](README.upstream.en.md)）。

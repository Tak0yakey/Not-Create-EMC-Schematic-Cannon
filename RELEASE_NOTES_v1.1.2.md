# Advanced Schematic Cannon — v1.1.2 Release Notes

**Release Date:** 2026-05-09
**Author:** BelugaLab
**Target:** Minecraft 1.21.1 / NeoForge 21.1.168+

[English](#english) | [日本語](#日本語)

---

## English

### Overview
v1.1.2 is a hardening & polish release. The mod now ships with proper multiplayer safety, packet validation, and major robustness improvements to the AE2 auto-crafting pipeline. This release contains no breaking gameplay changes — existing worlds, schematics, and recipes work without migration.

### 🆕 New Features / Improvements

- **EMC Toggle Restored** — The EMC ON/OFF button reappears on the EMC Schematic Cannon. The previous regression hid the toggle because `Menu#supportsEmc()` returned `false` at the moment `Screen#init()` ran (before the ContainerData arrived from the server). The check now delegates to the BlockEntity directly, so the button is always created when the BE is the EMC variant.
- **Missing-Item Name Display** — The "Missing block" indicator in the GUI now shows the item display name beside the icon, with auto-truncation when it exceeds the panel width.
- **Enhanced Cannon Visuals** — The Enhanced variant ships with its own GUI background (no EMC fuel slot icon) and barrel-front texture. The renderer chooses the model dynamically by block type. The EMC variant is unchanged.

### 🛡️ Multiplayer Hardening

- **Owner UUID Lock** — Opening the GUI no longer hijacks the owner. `ownerUUID` is only updated when the cannon is in `IDLE`, `FINISHED`, or `ERROR`. Once a job is `RUNNING` or `PAUSED`, the owner stays the player who started it, and subsequent EMC/fuel costs are deducted from that player.
- **Packet Ownership Check** — `CannonActionPacket` (start/stop/pause/resume) and `CannonSettingsPacket` reject inputs from non-owners. OPs (permissions(2)) are allowed for moderation.
- **Packet Input Validation** — `Action.values()[i]` indices are bounds-checked, `RangeBoardEditPacket.editMode` is restricted to 0..2, and `WandDistancePacket.distance` is `Mth.clamp`-ed to the wand's allowed range.

### 🤖 AE2 Auto-Crafting Robustness

- **Per-Cannon Pending Tracking** — `PendingCraftKey` now includes the cannon position. Two cannons on the same ME network requesting the same item no longer block each other.
- **Calc-Storm Prevention** — Standalone retry interval extended from **1 s → 30 s** to stop the cannon from sending a craft calculation every tick when a recipe is permanently missing.
- **Per-Call Simulation Requester** — A fresh `StaticCraftingRequester` is created per call. The previously shared `AE2GridNodeManager.simulationSource` field could race when multiple craft requests overlapped; that field is now removed.
- **Force Cleanup** — Pending craft entries are force-cancelled and removed after **5 minutes**. This bounds memory if AE2 fails to deliver a `jobStateChange` callback.

### 🖱️ GUI / UX Polish

- **Tab Click-Through Fix** — `SettingsTabWidget`, `SpeedTabWidget`, and `InformationTabWidget` no longer return `true` from `mouseClicked` when the user clicks a dead zone of the tab. This restores proper click-through to underlying inventory slots.
- **Speed Slider Spam Removed** — Dragging the speed slider now updates the displayed value locally without sending a packet every frame. A single packet is sent on `mouseReleased`. The click vs. drag X offsets are unified (the previous 1 px snap is gone).
- **Settings Sync Cooldown Extended** — `syncCooldown` increased from **10 → 30 ticks**, and the slider does not get overwritten by stale server values while it is being dragged. This eliminates the "value snaps back" UX bug.
- **Idle Fuel Consumption Stopped** — `consumeFuelItems` is now gated by `state == RUNNING`. Idle cannons no longer drain EMC from a stale owner UUID.
- **Client-Side Menu Fallback Fixed** — When the client BE has not yet arrived, the fallback dummy BE now has its `level` set and falls back to `ENHANCED_CANNON_BLOCK` when ProjectE is absent. The GUI no longer flash-closes via `stillValid()` returning false.

### 🔧 Internal / Future-Proofing

- **No More EnergyStorage Reflection** — The internal `consumeEnergy(int)` previously reflected `EnergyStorage.energy` (a private field). On any future NeoForge release that renames the field, the entire mod would have failed to load. Replaced with a small `InternalEnergyStorage` subclass that exposes a safe `consumeInternal(int)` using the inherited `protected` field.
- **Range Volume Cap** — `scanRemoveRange` (used by the Filler removal mode) now respects the existing 50,000-block volume limit. Inserting a Range Board covering a 64×64×64 area no longer spikes the server tick.
- **Create Dependency Tightened** — `neoforge.mods.toml` now requires Create `[6.0,)` instead of `[0.5,)`. The mod uses `com.simibubi.create.AllDataComponents` (Create 6.x API), so older Create versions would have crashed at class-load time.

### 🧾 Metadata

- **Author** — `hololocheck` → **BelugaLab**.
- **mod_version** — Bumped to **1.1.2** in `gradle.properties`.
- **In-Game Version Display Fix** — `neoforge.mods.toml` now hard-codes `version = "1.1.2"` instead of `${file.jarVersion}`, which was resolving to `0.0NONE` on the in-game Mods screen due to a moddev plugin / JAR manifest interaction. (This is a regression of the v1.0.1 fix.)

### 📦 Dependencies

| Mod | Version | Required |
|-----|---------|----------|
| Create | 6.0+ | ✅ Required |
| Applied Energistics 2 | 19.0+ | ⚙️ Optional (highly recommended for AE2 features) |
| ProjectE | 1.0+ | ⚙️ Optional (enables EMC variant) |
| JEI | 19.27+ | ⚙️ Optional |

### 🔄 Upgrade Notes

- **Drop-in upgrade** from v1.1.0 / v1.2.0 / earlier prereleases. Worlds, schematics, range boards, and recipes are preserved.
- If you previously tracked an in-flight craft job that hung due to AE2 not reporting completion, that entry will now be cleared after 5 minutes — no manual intervention needed.
- Multiplayer servers should expect a behavioral change: only the player who started a job can open the GUI to change settings during the run. OPs retain full access.

---

## 日本語

### 概要
v1.1.2 は **堅牢化＆ポリッシュリリース**です。マルチプレイ安全性、パケット検証、AE2 自動クラフトパイプラインの大幅な堅牢性向上を含みます。ゲームプレイ上の破壊的変更はなく、既存ワールド・概略図・レシピはそのまま使用できます。

### 🆕 新機能 / 改善

- **EMC トグル復活** — EMC概略図砲の EMC ON/OFF ボタンが再表示されるようになりました。以前のリグレッションでは、`Menu#supportsEmc()` が `Screen#init()` 実行時(ContainerData がサーバーから届く前)に `false` を返していたためボタンが生成されていませんでした。チェックを BlockEntity に直接委譲することで、BE が EMC バリアントの場合は常にボタンが作られます。
- **不足アイテム名表示** — GUI の「不足ブロック」表示にアイコンの隣にローカライズされたアイテム名を併記。パネル幅を超える場合は自動で末尾「...」に切り詰めます。
- **強化型砲の専用ビジュアル** — 強化型に独自の GUI 背景(EMC 燃料スロットアイコン無し)と砲身前面テクスチャを追加。レンダラーがブロック種別で動的に切替。EMC 型は無変更。

### 🛡️ マルチプレイ堅牢化

- **オーナー UUID ロック** — GUI を開いただけでオーナーが奪われる挙動を修正。`ownerUUID` は `IDLE` / `FINISHED` / `ERROR` 時のみ更新され、`RUNNING` / `PAUSED` 中はジョブを開始したプレイヤーのまま固定されます。後続の EMC・燃料コストは正しくそのプレイヤーから引き落とされます。
- **パケット所有権チェック** — `CannonActionPacket`(start/stop/pause/resume)と `CannonSettingsPacket` がオーナー以外からの操作を拒否します。OP(permissions(2))はモデレーション目的で許可。
- **パケット入力検証** — `Action.values()[i]` のインデックスを範囲チェック、`RangeBoardEditPacket.editMode` を 0..2 に制限、`WandDistancePacket.distance` を `Mth.clamp` で杖の許容範囲に正規化。

### 🤖 AE2 自動クラフトの堅牢化

- **キャノン位置を含む追跡** — `PendingCraftKey` にキャノン位置を追加。同一 ME 網に2台のキャノンがあって同じアイテムを要求しても相互干渉しません。
- **計算ストーム対策** — Standalone リトライ間隔を **1秒 → 30秒** に延長。レシピが永続的に未登録のとき、毎 tick craft 計算を AE2 に投げ続ける問題を解消。
- **per-call シミュレーション Requester** — 呼び出しごとに新しい `StaticCraftingRequester` を生成。以前は `AE2GridNodeManager.simulationSource` を共有 field として上書きしていたため、複数の craft 要求が重なると race していました。当該 field は削除済み。
- **強制クリーンアップ** — Pending craft エントリは **5分** タイムアウトで強制 cancel & 削除。AE2 から `jobStateChange` 通知が来なかった場合のメモリ圧迫を防止。

### 🖱️ GUI / UX 改善

- **タブクリック貫通修正** — `SettingsTabWidget` / `SpeedTabWidget` / `InformationTabWidget` の `mouseClicked` がタブのデッドゾーンクリックでも `true` を返してしまい、下層のインベントリスロットに反応しない問題を修正。
- **スピードスライダーの spam 解消** — ドラッグ中はローカル値の更新のみ行い、毎フレームのパケット送信を停止。`mouseReleased` 時に1回だけ送信。クリックとドラッグの X 基点を統一(従来の 1 px ずれを解消)。
- **設定同期 cooldown 延長** — `syncCooldown` を **10 → 30 tick** に延長し、ドラッグ中はサーバー値で上書きしないように修正。「値が戻る UX バグ」を解消。
- **アイドル時の燃料消費停止** — `consumeFuelItems` を `state == RUNNING` でガード。アイドル中のキャノンが古いオーナー UUID から EMC を吸い続ける問題を解消。
- **クライアント側 Menu フォールバック修正** — クライアント BE が未到着のとき、ダミー BE に `level` をセット。ProjectE 不在環境では `ENHANCED_CANNON_BLOCK` を fallback に使用。`stillValid()` が false を返して GUI が瞬時閉じる UX バグを解消。

### 🔧 内部 / 将来互換

- **EnergyStorage リフレクション廃止** — 内部消費メソッド `consumeEnergy(int)` は以前 `EnergyStorage.energy` (private field) をリフレクションで操作していました。NeoForge の将来リリースでフィールドがリネームされると mod がクラスロードに失敗する状況でした。`InternalEnergyStorage` サブクラスを作って継承した `protected` フィールドを安全に更新する `consumeInternal(int)` に置換。
- **範囲体積上限** — フィラー撤去モードの `scanRemoveRange` に既存の 50,000 ブロック体積上限を適用。64×64×64 の範囲ボードを挿入してもサーバ tick がスパイクしません。
- **Create 依存条件の厳格化** — `neoforge.mods.toml` の Create 依存範囲を `[0.5,)` → `[6.0,)` に変更。本 mod は `com.simibubi.create.AllDataComponents`(Create 6.x API)を使うため、古い Create バージョンではクラスロード時にクラッシュしていました。

### 🧾 メタ情報

- **作者** — `hololocheck` → **BelugaLab**。
- **mod_version** — `gradle.properties` で **1.1.2** に更新。
- **ゲーム内バージョン表示の修正** — `neoforge.mods.toml` を `version = "${file.jarVersion}"` から `version = "1.1.2"` に直接指定に変更。`${file.jarVersion}` が moddev plugin と JAR マニフェストの相互作用で `0.0NONE` に解決されてしまう問題を修正(v1.0.1 で直していた箇所のリグレッション)。

### 📦 依存 Mod

| Mod | バージョン | 必須 |
|-----|-----------|------|
| Create | 6.0+ | ✅ 必須 |
| Applied Energistics 2 | 19.0+ | ⚙️ 任意(AE2機能利用に強く推奨) |
| ProjectE | 1.0+ | ⚙️ 任意(EMC バリアント有効化) |
| JEI | 19.27+ | ⚙️ 任意 |

### 🔄 アップグレード手順

- **v1.1.0 / v1.2.0 / それ以前のプレリリースから drop-in アップグレード可能**。ワールド・概略図・範囲指定ボード・レシピは保持されます。
- AE2 のジョブ完了通知が来ずにハングしていた古い craft エントリがあった場合、5 分後に自動クリアされます。手動操作は不要です。
- マルチプレイサーバーでは挙動変更にご注意: ジョブ稼働中、設定変更可能なのはジョブ開始者のみになります。OP は引き続き全権限でアクセスできます。

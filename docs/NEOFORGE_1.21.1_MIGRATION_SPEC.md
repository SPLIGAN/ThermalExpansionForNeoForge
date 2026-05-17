# Thermal Expansion — NeoForge 1.21.1 向け修正仕様書

本書は **Arclight** 上で **NeoForge 対応 Thermal Expansion** を動作させるための修正仕様である。  
実装は **Free 枠のみ**で進め、有償クレジット依存の手段は用いない。

---

## 1. 目的

- **Arclight**（`arclight-neoforge-1.21.1-1.0.2-SNAPSHOT-0769551`）環境で **NeoForge 対応 Thermal Expansion** が安定して動作すること。
- **Forge 1.20.1 → NeoForge 1.21.1** の API 変更（特に **ItemStack / FluidStack の Data Components 化**）に追随すること。
- 依存 **[CoFHCoreForNeoForge](https://github.com/CoFH/CoFHCore)** / **[ThermalCoreForNeoForge](https://github.com/CoFH/ThermalCore)** / **[ThermalFoundationForNeoForge](https://github.com/CoFH/ThermalFoundation)** の移行方針に整合すること。

---

## 2. 前提（ターゲット環境）

| 項目 | 値 |
|------|-----|
| Minecraft | **1.21.1** |
| サーバ | **Arclight 1.0.2-SNAPSHOT-0769551** |
| Mod ローダ | **NeoForge 21.1.219** |
| 依存 | **cofh_core** 11.0.2+、**thermal**（Thermal Core）11.0.7+ |
| 本リポジトリ | `gradle.properties` 上 **1.21.1 / NeoForge 21.1.219** |

---

## 3. 参照資料

| 資料 | URL |
|------|-----|
| NeoForge 公式 | https://docs.neoforged.net/ |
| Data Components | https://docs.neoforged.net/docs/1.21.1/items/datacomponents |
| Forge Wiki | https://forge.gemwire.uk/wiki/Main_Page |
| Forge / NeoForge 比較表 | https://docs.google.com/spreadsheets/d/1_DQELiPvCF0FmFfyU4opGDbWi7zv-bSl8ImZuh6645E/edit?gid=248444698 |
| 修正参考（Thermal Foundation） | `ThermalFoundationForNeoForge/docs/NEOFORGE_1.21.1_MIGRATION_SPEC.md` |
| 修正参考（Thermal Core） | `ThermalCoreForNeoForge`（composite build） |

---

## 4. スコープ

### 4.1 本リポジトリ（Thermal Expansion）

- Dynamos / Machines ブロック・BE・Menu・Screen
- JEI 連携（レシピカテゴリ、Potion 流体プラグイン）
- データ生成（ルート、レシピ、タグ、モデル）
- `META-INF/neoforge.mods.toml`

### 4.2 依存側

| 依存 | 状態（2026-05-17） |
|------|---------------------|
| cofh_core | CoFHCoreForNeoForge — **compile 成功** |
| thermal | ThermalCoreForNeoForge — **compileJava / publishToMavenLocal 成功** |
| thermal_foundation | ランタイム用（本 mod の compile 依存ではない） |

**Expansion 単体の `compileJava` は Thermal Core の composite build または `publishToMavenLocal` が前提。**

---

## 5. 全体方針

1. **ItemStack / FluidStack**: `getTag` / `setTag` / `hasTag` を置かない。ポーションは `PotionFluid.getItemFromPotionFluid` / `getPotionFluidFromItem`（CoFH Core）を使用。
2. **イベント**: `@Mod.EventBusSubscriber` 廃止 → `ThermalExpansion` コンストラクタで `modEventBus.register(TExpDataGen.class)`。
3. **ResourceLocation**: `ResourceLocation.parse` / `fromNamespaceAndPath` を使用。
4. **メタデータ**: `META-INF/neoforge.mods.toml` + `ProcessResources` の expand プロパティ。

---

## 6. 修正仕様 — ビルド・メタデータ

| ID | 内容 | 状態 |
|----|------|------|
| B-01 | `gradle.properties` → MC **1.21.1** / NeoForge **21.1.219** / Java **21** | **完了** |
| B-02 | NeoGradle userdev **7.1.25**、Gradle **8.14** | **完了** |
| B-03 | `settings.gradle` — `includeBuild('../ThermalCoreForNeoForge')` | **完了** |
| B-04 | `neoforge.mods.toml` 作成、`mods.toml` 削除 | **完了** |
| B-05 | Modrinth Maven、CoFH Maven、`build.gradle` を Foundation 準拠 | **完了** |
| B-06 | `./gradlew jar` 成功 | **完了** |
| B-07 | `runData` — Thermal Core の `--existing` を `data` 実行に追加 | **完了** |

---

## 7. 修正仕様 — ソース（本リポジトリ調査）

| ID | 区分 | 対応 |
|----|------|------|
| TE-S01 | `PotionFluidRecipeManagerPlugin` — `hasTag` / `setTag` | `PotionFluid.getItemFromPotionFluid` / `getPotionFluidFromItem` | **完了** |
| TE-S02 | `@Mod.EventBusSubscriber` — `TExpDataGen` | `ThermalExpansion` で `modEventBus.register` | **完了** |
| TE-S03 | `new ResourceLocation(String)` — Screen×22, JEI, Sounds | `ResourceLocation.parse` / `fromNamespaceAndPath` | **完了** |
| TE-S04 | `TExpLootTableProvider` | `HolderLookup.Provider` を渡す | **完了** |
| TE-S05 | `TExpBlockLootTables` | `BlockLootSubProviderCoFH(HolderLookup.Provider)` コンストラクタ | **完了** |
| TE-S06 | `TExpRecipeProvider` — 廃止 `Tags.Items.STONE/SAND/GLASS` | `Blocks.STONE` / `Blocks.SAND` / `Blocks.GLASS` | **完了** |
| TE-S07 | `MachineCrafter*` — クラフト API | `CraftingInput.of` + `getRecipeFor` | **完了** |
| TE-S08 | BE `cacheRenderFluid` — `new FluidStack(FluidStack, int)` | `copyWithAmount` | **完了** |
| TE-S09 | 触媒 `ItemStack.hurt` | `hurtAndBreak` + `ServerLevel` キャスト | **完了** |
| TE-S10 | Crafter 設定パケット `writeItem` | `ItemStack.STREAM_CODEC` + `RegistryFriendlyByteBuf` | **完了** |
| TE-C01 | `ThermalMachineConfig` — Crystallizer 設定が `push("Brewer")` に誤配置 | `push("Crystallizer")` | **完了** |
| TE-C02 | `javax.annotation` — BlockEntity 等 23 ファイル | `org.jetbrains.annotations`（`@Nonnull` → `@NotNull`） | **完了** |
| TE-S11 | `MachineCrafterMenu.slotChangedCraftingGrid` — `setRecipeUsed` が自己参照 | `possibleRecipe.get()` を設定 | **完了** |
| TE-D01 | `./gradlew runData` | `build.gradle` の `data` 実行に Thermal Core の `--existing` を追加（`slot_seal` 等の item テクスチャは Core 側） | **完了** |
| TE-D02 | JEI `addTooltipCallback` | JEI 19.x で非推奨。動作に支障なし。必要時のみ `IRecipeSlotTooltipCallback` 等へ移行 | **保留** |

### 7.1 CompoundTag（変更不要）

以下 3 ファイルの `CompoundTag` は **オーグメント設定用引数**（ItemStack ルート NBT ではない）。Thermal Core 親クラスに合わせ **変更不要**。

- `MachinePulverizerBlockEntity.java`
- `MachineSmelterBlockEntity.java`
- `MachineInsolatorBlockEntity.java`

---

## 8. 完了定義

1. **Thermal Core** 含め **`compileJava` / `jar` / `build` / `runData` / `publishToMavenLocal` 成功**（composite build または `publishToMavenLocal`）。
2. 本リポジトリに **ItemStack 廃止 NBT API ゼロ**、**`@Mod.EventBusSubscriber` ゼロ**、**`javax.annotation` ゼロ**。
3. Arclight 上で **A-01〜A-03** を最低限実施。

### 8.0 ビルド成果物（2026-05-17 再検証）

| コマンド | 結果 |
|----------|------|
| `./gradlew compileJava` | **成功** |
| `./gradlew build` | **成功** |
| `./gradlew runData` | **成功**（122 ファイルキャッシュ、新規 0） |
| `./gradlew publishToMavenLocal` | **成功** → `~/.m2/repository/com/teamcofh/thermal_expansion/` |
| 廃止 API スキャン（`getTag` / `setTag` / `hasTag` / `@Mod.EventBusSubscriber` / `new ResourceLocation(`） | **ヒット 0** |

### 8.1 Arclight 実機検証手順

1. `CoFHCoreForNeoForge` / `ThermalCoreForNeoForge` で `publishToMavenLocal`（または jar を `mods/` に配置）。
2. **cofh_core**、**thermal**（Core）、**thermal_expansion**（本 jar）を同一 `mods` に置く。
3. Arclight NeoForge 1.21.1 サーバ起動 — mod ロードエラーなし（A-01）。
4. Dynamo / Machine の設置・GUI 開閉（A-02）。
5. JEI で Bottler + ポーション流体レシピ表示（A-03）。

---

## 9. Arclight 向け検証

| ID | 検証項目 | 状態 |
|----|----------|------|
| A-01 | cofh_core + thermal + thermal_expansion 起動 | **未** |
| A-02 | Dynamo / Machine 設置・GUI 開閉 | **未** |
| A-03 | JEI でレシピ表示（特に Bottler + ポーション流体） | **未** |

---

## 10. 改訂履歴

| 日付 | 内容 |
|------|------|
| 2026-05-17 | 初版 — 1.20.4 から 1.21.1 移植開始。仕様書作成、ビルド基盤・ソース修正着手 |
| 2026-05-17 | **B-01〜B-06 / TE-S01〜TE-S10 完了** — `compileJava` / `jar` 成功。JEI `addTooltipCallback` は警告のみ |
| 2026-05-17 | **TE-C01 / TE-C02 完了** — Crystallizer 設定セクション名修正、`javax.annotation` → JetBrains |
| 2026-05-17 | **TE-D01** — `runData` は `slot_seal` テクスチャ未同梱で失敗（Arclight 実機検証 A-01〜A-03 は未実施） |
| 2026-05-17 | **TE-S11** — Crafter GUI でレシピ ID が更新されない不具合を修正（`setRecipeUsed` 自己参照） |
| 2026-05-17 | **TE-D01** — `runData` 用に Thermal Core `src/main/resources` を `--existing` に追加 |
| 2026-05-17 | **TE-D02** — JEI `addTooltipCallback` 非推奨を仕様書に記録（コード変更は保留） |
| 2026-05-17 | **再検証** — `build` / `runData` / `publishToMavenLocal` 成功。廃止 API スキャン 0 件。コード移植フェーズ完了、Arclight 実機（A-01〜A-03）のみ残 |

---

*本ドキュメントは Free 枠での作業前提で作成・更新する。*

# Codex CLI 交接文件 — Smart Wrong Notebook — 2026-05-17

## 项目定位

Smart Wrong Notebook 是 Flutter 移动应用，面向学生的 AI 错题本。一期重点是单机 Android。核心链路是：

> 拍照/框选图片 → AI 识别与解析 → 展示解析结果 → 生成举一反三 → 用户确认并保存到错题本

当前不要退回旧流程“拍照 → OCR 确认 → AI 分析”。

## 必守约束

- 不要 push 到 GitHub，除非用户明确要求。
- 不要 `git add .`。如需提交，先给用户精确文件清单确认。
- 不要把 API Key 写入代码、日志、Markdown 或 commit。
- 不要硬编码某张 fixture 图的答案。
- 图形题读图不确定时，App 应显示“可能解法/需核对”，不要显示确定绿色答案。
- 不要修改 LaTeX 渲染引擎：
  - `lib/src/shared/widgets/math_content_view.dart`
  - `lib/src/shared/widgets/katex_math_view.dart`
  - `assets/katex/`

## 本轮 Codex 已完成

- 接手项目并读取交接文件。
- 强化 AI generatedExercises 质量：
  - 默认 3 道题：简单 / 同级 / 提高。
  - 保留合格槽位，替换答案冲突或题型漂移槽位。
  - 几何题练习要求带 `diagramData`。
  - 加强平方方程、比例关系、函数求值、方程组、圆锥体积、等腰三角形、圆/半圆/组合面积、正方形垂直平分线等 profile/fallback。
- 优化多题拆分：
  - 普通独立编号题 `1. ... 2. ...` 拆成多题。
  - 共享题干小问 `（1）（2）` 保持一道综合题。
  - 语文/英语/文科材料默认不拆。
  - 化学/数学/物理共享题干 + 小问保持综合题；普通独立题仍拆。
- 修复多题偶发混成一道题的漏洞：
  - 症状：同一张 1-6 数学多题图偶发在 AI 解析结果页混在一起展示。
  - 根因：AI 识别文本正确，但某些入口状态下 `splitResult/candidateAnalyses` 丢失。
  - 修复：`AnalysisLoadingScreen` 在分析前，如果已有编号文本但缺少多题 `splitResult`，补跑拆题，再进入候选题并行分析。
  - 新增回归测试。
- 美化举一反三几何配图卡：
  - `GeometryDiagramWidget` 去掉内部左右 margin。
  - 背景/边框/圆角对齐题干和选项卡：`surface`、`outlineVariant`、14 圆角。
  - 检查确认正式入口中几何配图只通过该组件渲染。
- 工具脚本安全修正：
  - 测试/实验脚本改为从 `AI_TEST_API_KEY` / `AI_API_KEY` 读取 key。
- 迁移 Codex `park` skill：
  - `/Users/tangjun/.codex/skills/park/SKILL.md`
  - park 时会更新 `progress-current.md` 和 handoff md，并提醒是否安全 WIP commit。

## 关键文件

- `lib/src/data/remote/ai/ai_analysis_service.dart`
  - AI prompt、JSON repair、生成练习题解析、质量门、fallback/profile。
- `lib/src/data/services/question_split_service.dart`
  - 本地拆题规则，包含独立编号题与共享题干小问判断。
- `lib/src/features/analysis/presentation/analysis_loading_screen.dart`
  - AI 分析入口；新增 splitResult 缺失时的分析前兜底拆题。
- `lib/src/features/analysis/presentation/widgets/geometry_diagram_widget.dart`
  - 举一反三几何配图渲染；已统一卡片宽度/边框样式。
- `test/features/analysis/analysis_loading_screen_test.dart`
  - 多题兜底拆分回归测试。
- `test/features/analysis/exercise_practice_test.dart`
  - 举一反三练习、继续练习、diagramData 渲染回归。
- `test/data/remote/ai_analysis_service_test.dart`
  - generatedExercises 质量门和 fallback 回归。
- `test/app/providers_test.dart`
  - split session / provider 相关回归。

## 验证结果

- `flutter test test/data/remote/ai_analysis_service_test.dart test/data/local/drift_question_repository_test.dart test/features/analysis/exercise_practice_test.dart test/tool/analyze_image_fixture_test.dart test/app/providers_test.dart test/features/analysis/analysis_loading_screen_test.dart`
  - `EXIT_CODE=0`
  - `91 passed, 1 skipped`
  - skip：未设置 `AI_FIXTURE_IMAGE` 时图片 fixture 工具测试自动跳过。
- `flutter test test/features/analysis/exercise_practice_test.dart test/features/analysis/analysis_loading_screen_test.dart`
  - `EXIT_CODE=0`
  - `All tests passed`
- `flutter analyze --no-fatal-infos --no-fatal-warnings`
  - `EXIT_CODE=0`
  - 仍有 `100 issues found`，均 non-fatal。
  - 主要来自受保护 math 渲染测试和未跟踪 geometry demo。
- `git diff --check`
  - 通过。
- 受保护 LaTeX 渲染文件 diff 为空。

## 真实 fixture 观察

- `shuxue-jihe.png`
  - 识别正方形/垂直平分线，3 道练习带 diagramData。
- `duoti.png`
  - 模型可识别 6 道题。
  - 单题 fixture 入口会当综合输入；主 App 拆题链路可分别保存/练习。
- `wuli-dianzu.png`
  - 物理电学题通过，但因滑片/电路读图不确定进入 needsReview。
- `yuwen.png`、`yingyu.png`
  - 作为整篇材料类综合题处理，因小字/缺选项等进入 needsReview。

## APK

- 已构建 release APK：
  - `build/app/outputs/flutter-apk/ai-wrong-notebook-v62-20260517-0402.apk`
  - 约 `68M`
  - SHA256：`870bf14cb64007887062bd903ccf9d0802ea8cccc9f01e8e9350a06827c839a5`
- 用户下一步应真机安装该包验证：
  - 多题图是否稳定拆成 1-6 题展示。
  - 每个子题是否分别生成举一反三。
  - 几何配图卡是否与题干/选项同宽、边框一致。

## 当前 Git 状态摘要

Modified tracked files:

- `lib/src/data/remote/ai/ai_analysis_service.dart`
- `lib/src/data/services/question_split_service.dart`
- `lib/src/features/analysis/presentation/analysis_loading_screen.dart`
- `lib/src/features/analysis/presentation/widgets/geometry_diagram_widget.dart`
- `test/app/providers_test.dart`
- `test/data/remote/ai_analysis_service_test.dart`
- `test/features/analysis/analysis_loading_screen_test.dart`
- `test/tool/direct_image_experiment.dart`
- `test/tool/direct_image_test.dart`
- `test/tool/single_pass_experiment.dart`
- `progress-current.md`
- `progress-codex-cli-handoff.md`

Untracked files include:

- `CLAUDE.md`
- `progress-2026-04-29.md`
- `progress-ai-b1-handoff.md`
- `progress-ai-geometry-handoff.md`
- `test/fixtures/duoti.png`
- `test/fixtures/shuxue-jihe.png`
- `test/fixtures/wuli-dianzu.png`
- `test/fixtures/yingyu.png`
- `test/fixtures/yuwen.png`
- multiple `docs/*.html` drafts
- `test/tool/geometry_canvas_demo.dart`
- `test/tool/geometry_svg_auxiliary_demo.html`
- `test/tool/geometry_svg_samples.html`

Do not stage all files.

## 风险点

- 旧异常记录不会自动迁移；如果用户需要，应手动删除异常题或单独做数据清理。
- 用户此前提供过真实 API Key，后续应作废/重置。
- Analyzer 仍有 non-fatal info/warning；不要为了清它去碰 LaTeX 渲染引擎。
- 当前没有自动真机验证，仍以用户安装 APK 后反馈为准。

## 下一步

1. 等用户真机测试 `ai-wrong-notebook-v62-20260517-0402.apk`。
2. 如果真机 OK，准备精确文件清单，请用户确认是否做本地 WIP commit。
3. 如用户确认 commit：
   - 只 `git add <明确文件列表>`。
   - commit message 固定：`wip: end of day state`。
   - 不 push。

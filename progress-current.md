# 项目进展 / Park — 2026-05-17

## Done

- Codex 已接手 Smart Wrong Notebook 当前 Flutter 项目，并读取交接文件与项目约束。
- 已继续推进 AI 解析与举一反三质量优化：
  - generatedExercises 默认收敛为 3 道题：简单 / 同级 / 提高。
  - 强化练习题质量门：答案冲突、自我否定、题型漂移、几何题缺失 diagramData 等都会被拦截或替换。
  - 对平方方程、比例关系、函数求值、方程组、圆锥体积、等腰三角形、圆/半圆/组合面积、正方形垂直平分线等题型增加或强化 profile/fallback。
  - “继续练习”链路已按当前实现支持新一轮 3 题循环练习。
- 已优化多题拆分规则：
  - 普通独立编号题（如 `1. ... 2. ... 3. ...`）继续拆成多题。
  - 共享题干的小问（如 `（1）（2）`）保持为一道综合题。
  - 语文/英语/文科综合材料类默认不拆。
  - 化学/数学/物理遇到共享题干 + 小问时保持综合题；普通独立题仍拆。
- 已修复一个实际发现的多题链路漏洞：
  - 异常现象：同一张多题数学图偶发进入结果页时 1-6 题混成一道题展示。
  - 根因判断：AI 识别文本正确，但某些入口状态下 `splitResult/candidateAnalyses` 丢失，结果页只能按单题展示。
  - 修复：`AnalysisLoadingScreen` 在分析前若发现已有编号文本但缺少多题 `splitResult`，会用现有文本补跑拆题，再进入多题候选分析。
  - 已新增回归测试覆盖“已有 1/2/3 文本但 splitResult 缺失”的路径。
- 已按用户反馈美化举一反三详情页中的几何配图卡：
  - `GeometryDiagramWidget` 去掉内部左右 16px margin。
  - 配图卡边框、背景和圆角对齐题干/选项卡：`surface` + `outlineVariant` + 14 圆角。
  - 检查确认正式入口中几何配图只通过 `GeometryDiagramWidget` 渲染，因此该修改覆盖所有举一反三几何配图入口。
- 已清理测试/工具脚本中的真实 API Key 写入风险：
  - 相关脚本改为读取 `AI_TEST_API_KEY` / `AI_API_KEY` 环境变量。
  - 未把 API Key 写入文件、日志、Markdown 或 commit。
- 已用真实 fixture + `gpt-5.5` 跑过多张图片回归：
  - `shuxue-jihe.png`：正方形/垂直平分线识别和练习题质量通过，几何练习带 diagramData。
  - `duoti.png`：模型识别 6 题，单题 fixture 入口会当综合输入；主 App 拆题链路可分别保存/练习。
  - `wuli-dianzu.png`：物理电学题通过，因滑片/电路读图不确定进入 needsReview。
  - `yuwen.png`、`yingyu.png`：整篇材料类按综合题处理，因小字/缺选项等进入 needsReview。
- 已打包 release APK：
  - `build/app/outputs/flutter-apk/ai-wrong-notebook-v62-20260517-0402.apk`
  - 大小约 `68M`
  - SHA256：`870bf14cb64007887062bd903ccf9d0802ea8cccc9f01e8e9350a06827c839a5`
- 已把 Claude Code 的 `park` skill 迁移到 Codex：
  - `/Users/tangjun/.codex/skills/park/SKILL.md`
  - 已调整为 park 时主动展示 git 状态并提醒是否做安全本地 WIP commit。

## Verification

- `flutter test test/data/remote/ai_analysis_service_test.dart test/data/local/drift_question_repository_test.dart test/features/analysis/exercise_practice_test.dart test/tool/analyze_image_fixture_test.dart test/app/providers_test.dart test/features/analysis/analysis_loading_screen_test.dart`
  - `EXIT_CODE=0`
  - `91 passed, 1 skipped`
  - skip 原因：未设置 `AI_FIXTURE_IMAGE` 时图片 fixture 工具测试自动跳过。
- `flutter test test/features/analysis/exercise_practice_test.dart test/features/analysis/analysis_loading_screen_test.dart`
  - `EXIT_CODE=0`
  - `All tests passed`
  - 覆盖举一反三练习页、继续练习、候选题保存选择、diagramData 渲染、多题兜底拆分。
- `flutter analyze --no-fatal-infos --no-fatal-warnings`
  - `EXIT_CODE=0`
  - 当前仍有 `100 issues found`，为 non-fatal info/warning。
  - 主要来自受保护的 math 渲染测试和未跟踪 geometry demo；本轮没有为清 analyzer 去碰 LaTeX 渲染引擎。
- `git diff --check`
  - 通过。
- 受保护 LaTeX 渲染文件 diff 检查：
  - `lib/src/shared/widgets/math_content_view.dart`
  - `lib/src/shared/widgets/katex_math_view.dart`
  - `assets/katex/`
  - diff 为空，未修改。

## Blockers / 风险点

- 当前 working tree 有较多已修改和未跟踪文件，不要 `git add .`。
- 本轮尚未 commit、未 push。
- 用户已在真机安装过 `ai-wrong-notebook-v62-20260517-0335.apk` 并发现一次旧链路异常；新包为 `0402`，需要真机复测：
  - 多题数学图是否稳定显示 1-6 题。
  - 每个子题是否分别生成举一反三。
  - 举一反三几何配图卡是否与题干/选项同宽、同边框颜色。
- 旧的异常保存记录不会自动迁移；如需要，应手动删除异常记录或后续单独做数据清理工具。
- 用户此前提供过真实 API Key，后续应作废/重置；交接文件不能记录 key。
- 仍不要修改 LaTeX 渲染引擎：
  - `lib/src/shared/widgets/math_content_view.dart`
  - `lib/src/shared/widgets/katex_math_view.dart`
  - `assets/katex/`

## Current Git State

- Modified tracked files:
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
- Untracked files include handoff/history docs, fixture images, and geometry demo files:
  - `CLAUDE.md`
  - `progress-2026-04-29.md`
  - `progress-ai-b1-handoff.md`
  - `progress-ai-geometry-handoff.md`
  - `test/fixtures/duoti.png`
  - `test/fixtures/shuxue-jihe.png`
  - `test/fixtures/wuli-dianzu.png`
  - `test/fixtures/yingyu.png`
  - `test/fixtures/yuwen.png`
  - multiple `docs/*.html` drafts and `test/tool/geometry_*` demos.

## Next First Step

1. 真机安装并测试 `ai-wrong-notebook-v62-20260517-0402.apk`。
2. 重点验证：
   - `duoti.png` 类多题数学图是否稳定拆成多题展示。
   - 每个子题的举一反三是否分别围绕原题型生成。
   - 几何配图卡宽度/边框是否与题干和选项一致。
3. 若真机确认 OK，再决定是否做本地 WIP commit。
4. 如需 commit，必须先确认精确文件清单；不要 `git add .`，不要 push。

## Tomorrow first action

- 先从真机反馈开始；如果 `0402` 包通过，就准备一份精确提交清单供用户确认，再做安全本地 WIP commit。

Good night.

# Responsive Object Continuity (ROC)

An agent skill for interfaces where objects keep their identity, state and place as layouts change.

Cards expand into readers. Folders unfold without replacing their contents. Responsive views preserve selection and context. Motion explains where the object went and how to return.

**响应式对象连续性**：让用户始终认得、找得到并能继续操作同一个对象。覆盖卡片展开、文档阅读、文件夹收纳、拖拽、关联视图、响应式布局和物理动效。

## Install

Clone into your agent's skill directory (example for Codex):

```sh
git clone https://github.com/SonghaiFan/responsive-object-continuity.git ~/.codex/skills/responsive-object-continuity
```

If that directory already exists, update your existing checkout or choose a different destination; do not overwrite local changes.

You can also [download the ZIP](https://github.com/SonghaiFan/responsive-object-continuity/archive/refs/heads/main.zip), extract it, and place the folder containing `SKILL.md` in your agent's supported skill directory. Other agents can read `SKILL.md` and the linked cases directly; automatic discovery depends on the host.

## Use

Ask your agent:

> Use $responsive-object-continuity to make this card expand into its full view while preserving the editor, selection, layout and return position. Verify opening, closing after scrolling, and interruption on desktop and mobile.

> 使用 responsive-object-continuity 检查这个界面的对象连续性：保留同一个对象、内部排版和操作状态，检查展开、收起、滚动后返回及中途反转。

Start with [SKILL.md](SKILL.md), then read only the relevant cases. It includes the principles, implementation workflow, diagnostic method and case index.

## What is inside

- Retained objects, editors and media controls across layout and presentation changes.
- Viewport and host safe areas; source slots and return geometry.
- Document composition, attached image annotations and continuous reading surfaces.
- Folder intake, paper/cover motion, coordinated shading and synchronization boundaries.
- Linked projections, durable selection and focused lanes with context strips.
- [Tooling notes](references/tooling.md) for observing and measuring transitions.

This is a Markdown skill and case library, not a runtime UI package. Examples include React/Motion, browser DOM and native UI patterns; the underlying principles are framework-independent. Referenced project files and revisions document the origin of observations; those projects and their assets are not bundled and may not be publicly available. The cases remain readable without them.

Evidence levels are explicit: source review, historical tests and current visual acceptance are different claims. Project-specific values are examples, not universal design tokens.

## Contribute

Open an issue with the interaction, expected behavior, reproduction conditions and evidence. For a case contribution, use [the template](cases/_TEMPLATE.md), choose an unused number and update the index in `SKILL.md`. A source-backed positive example should state mechanisms and limitations rather than inventing a bug history. Exclude personal paths, credentials and private user content.

## License

[MIT](LICENSE). The license covers this repository's skill text and examples; it does not grant rights to referenced external projects, trademarks or assets.

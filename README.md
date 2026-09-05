# claude_code_skills

Personal [Claude Code](https://claude.com/claude-code) skills.

## Skills

| Skill | Use when |
| --- | --- |
| [accessibility](skills/accessibility/SKILL.md) | Auditing or improving web accessibility — a11y audit, WCAG 2.2 compliance, screen reader support, keyboard navigation |
| [clean-code](skills/clean-code/SKILL.md) | Writing, reviewing, or refactoring code — function size, nesting, conditionals, magic values, duplication, naming, over-abstraction, and behavior-preserving refactoring with evidence and safety gates |
| [freelance-proposal-writer](skills/freelance-proposal-writer/SKILL.md) | Writing an Upwork (EN) or Mostaql (AR) proposal from a pasted job post — research-first hook, 250-char opener, job-driven positioning |
| [performance-review](skills/performance-review/SKILL.md) | Reviewing or diagnosing performance — slow endpoints, bad p99, N+1 queries, memory growth, cache misses, Core Web Vitals, scaling questions |
| [seo](skills/seo/SKILL.md) | Creating or editing pages, routes, layouts, templates, sitemaps, or structured data in a website |
| [ui-review](skills/ui-review/SKILL.md) | Reviewing UI/frontend changes for content overflow, responsive layout, flex-vs-fixed sizing, z-index discipline, SCSS/BEM quality, and async loading/error/empty states |
| [web-design-guidelines](skills/web-design-guidelines/SKILL.md) | Building or reviewing any web UI against a generic WCAG 2.2, focus/keyboard, forms, motion, responsive, Core Web Vitals, and dark-mode checklist |

`accessibility` is vendored from [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills) (MIT).

## Install

Copy or symlink a skill directory into `~/.claude/skills/`:

```bash
ln -s "$PWD/skills/clean-code" ~/.claude/skills/clean-code
```

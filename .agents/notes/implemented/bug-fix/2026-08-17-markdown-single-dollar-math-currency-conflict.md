# Agent Note: Markdown single-dollar math disabled to keep currency literal

Status: implemented

English | [中文](2026-08-17-markdown-single-dollar-math-currency-conflict.zh.md)

## Problem

The settled markdown grammar registers `micromark-extension-math` with default options, which enables single-dollar inline math (`$…$`). Its tokenizer is permissive: any `$` opens, the next `$` closes, and spaces, line endings, CJK, and arbitrary prose are accepted as content. Assistant replies routinely contain paired dollars as currency (`$0.094/M` … `$1`), so a sentence such as "闲时输出约 $0.094/M，可以先花 $1 买套餐" is swallowed into a single `inlineMath` node. KaTeX then either renders the prose as math-italic garbage or falls through the three-arm error chain to a red `katex-error` span, making the affected sentence look like a different font.

## Decision

`parseGfmWithMath` registers `math({ singleDollarTextMath: false })`. A bare `$` is no longer a math delimiter; it renders literally, so currency amounts survive. TeX math remains fully available through the delimiters `mathCompatibility()` already provides — `\(…\)`, `\[…\]`, and same-line `$$…$$` display blocks — plus the upstream multi-dollar text construct (`$$…$$` inline, with `\tag{}`). Escaped `\$` keeps its upstream meaning. The streaming arm (`parseGfm`) never had math, so both arms agree that a lone dollar is literal; only the settled arm's delimiter set changed.

Unit and DOM-parity coverage now exercises currency and prose dollars as literal text, and the README delimiter list (English and Chinese) drops `$…$` accordingly.

## Alternatives considered

**Keep single-dollar math but require TeX-looking content.** Rejected because a content heuristic (balanced braces, no CJK, no whitespace runs) is brittle, surprises users asymmetrically, and still misclassifies plausible currency strings; the custom tokenizer work would duplicate upstream grammar surface for a marginal capability.

**Escape dollars adjacent to digits or CJK.** Rejected because escaping by context silently rewrites user-visible source and creates a second, undocumented dialect of dollar handling that differs between prose and code.

**Keep the default and document the escape hatch.** Rejected because the failure is visual and frequent — every currency pair in a reply mangles — while the documented workaround (`\$` or `\(…\)`) is undiscoverable for normal users. The two-dollar threshold (`$$…$$`) keeps real math ergonomic.

## Consequences

Currency and any other prose containing `$` render literally; `$…$` math written as such now shows the dollars instead of a formula. Math authors use `\(…\)`, `\[…\]`, or `$$…$$`, which the same settled grammar already supports — the README and the JSDoc on `parseGfmWithMath` state this. Tests pin literal dollars in prose and tables, the error arm via an unbalanced `\(` instead of an unbalanced `$`, and unchanged behavior for `$$…$$` inline/display, escaped dollars inside math, streaming deferral, and fenced `math` blocks.

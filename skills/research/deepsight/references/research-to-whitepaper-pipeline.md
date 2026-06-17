# Research → Whitepaper Pipeline (2026-06-12 session)

## Workflow: Deep Research → DeepSight Review → Whitepaper

This session established a repeatable pipeline for converting a theoretical research question ("what are the strategic management theories for tacit knowledge acquisition in AI transformation?") into a published DeepArchi whitepaper.

### Phase 1: Rapid Literature Scan (15 min)
1. SearXNG-first approach: restart if engines are suspended, then multi-engine search
2. DDG HTML fallback for 1-2 queries (expect CAPTCHA after second query)
3. **arxiv PDF full-text extraction**: Download PDF → `pdftotext -f N -l M paper.pdf` for targeted section extraction. More reliable than scraping arxiv HTML pages or JS-rendered journal sites (ScienceDirect, Emerald).
4. Google Scholar via SearXNG `engines=google+scholar` for journal article discovery
5. Parallel searches across 3-4 theory angles simultaneously

### Phase 2: DeepSight Adversarial Review (10 min)
1. Map all collected theories onto a single framework diagram
2. Identify 3 key controversies/conflicts between competing models
3. Apply adversarial templates: 
   - "Is GenAI an actor or tool?" (GRAI/AKI vs GenAI SECI)
   - "Can tacit knowledge actually be captured?" (three-layer model)
   - "Does AI cause deskilling?" (empirical evidence)
4. Evidence grading: [A] for mature theories, [B] for recent preprints, [C] for synthesis

### Phase 3: Whitepaper Production (15 min)
1. Follow `deeparchi-writing` conventions:署名, 词汇, 脱敏, 调性
2. Follow `consulting-report` 10-chapter structure
3. Write to local markdown first (`/mnt/d/ai workspace/deeparchi/docs/`)
4. **Humanizer self-review**: scan for 29 AI writing patterns, especially:
   - `deeparchi-writing` specific: 终局, 独特X在于, 邝谧原则, 诺致科技
   - Humanizer: signposting, hedging, persuasive authority tropes
5. Fix violations, then create Feishu doc

### Phase 4: Feishu Delivery
1. Prep content: `sed -i '1i<title>标题</title>' whitepaper.md` (v2 API requires `<title>` tag in content, `--title` flag is deprecated)
2. Create: `lark-cli docs +create --api-version v2 --doc-format markdown --as bot --content '@whitepaper.md'`
3. Verify table separators use `|---|---|` (Feishu rejects `|-|-|`)

### Pitfalls Encountered
- **SearXNG engine suspension**: All engines show "暂停服务: 超时" after previous session. Kill and restart SearXNG to clear suspended state.
- **DDG HTML CAPTCHA**: Only 1-2 queries per session. Plan queries carefully.
- **JS-rendered journal sites**: ScienceDirect, Emerald, MDPI — meta tags often missing, full text requires browser. Use Google Scholar search + arxiv fallback.
- **Feishu `--title` deprecation**: lark-cli v1.0.50 rejects `--title`. Must embed `<title>` in markdown content.
- **"邝谧原则" naming**: deeparchi-writing脱敏规则要求使用"交叉验证协议"或"双模型交叉验证"替代。

### Key Literature Sources for This Class of Research
- arxiv.org — preprints, best for bleeding-edge (2026 papers)
- Google Scholar via SearXNG — journal article discovery
- pdftotext — PDF text extraction (already installed on WSL)
- Avoid: direct browser navigation to ScienceDirect/Emerald (JS-rendered, requires heavyweight browser approach)

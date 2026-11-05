---
name: "ai-writing-self-check"
description: "Self-check a document for likely AI-written text before an examiner, lecturer or journal sees it, with a Turnitin-style report (overall AI percentage, AI-generated and AI-paraphrased breakdown, cyan and purple highlights, integrity checks) and ranked revision recommendations. Use whenever someone asks to check a thesis, essay, assignment, dissertation, manuscript or report for AI, asks for an AI score or AI percentage, mentions Turnitin's AI detector, GPTZero-style checks, or wants to lower a flagged AI score, even if they do not say \"self-check\"."
---

# AI writing self-check

Give the author the view an examiner would get, then show what to revise. The result is an independent estimate. It never reproduces Turnitin's score, because Turnitin's model is private.

Always show the actual percentage, however low or high it is. Write everything for the author in plain language: say what the score means and what to do, and avoid terms such as "qualifying text", "signals" and "register".

## Workflow

0. **Unpack the scripts.** Both scripts are stored at the end of this file. `<skill-dir>` is this skill's base directory.

   ```bash
   mkdir -p aicheck && python3 -c "import re,sys;[open('aicheck/'+n,'w').write(c) for n,c in re.findall(r'^~~~python (\S+)\n(.*?)^~~~$',open(sys.argv[1]).read(),re.M|re.S)]" <skill-dir>/SKILL.md
   ```

   If SKILL.md is not on disk, write each block out to `aicheck/` exactly as printed.

1. **Find the document.** Use the file the person attached or named. If several drafts exist, confirm which one. For a Google Doc, read it through the connector and rebuild it as a .docx that keeps heading styles, bullets and tables, so that the rules for what counts still work.

2. **Run the automatic check.** It needs `python-docx` and `pdfplumber`, plus LibreOffice for .rtf.

   ```bash
   python3 aicheck/analyze.py INPUT -o analysis.json
   ```

   If the state is `unavailable`, skip to step 4 and report why. Files under 300 words, over 30,000 words, scanned PDFs and non-English documents all land here. For a long thesis, offer to split it by chapter and check each part. For a scanned PDF, run OCR and try again.

3. **Do the reading review.** Follow **Reading review** below: read every paragraph in `analysis.json`, then write `review.json`. Do not skip this, including when the automatic check says 0%. The automatic check misses plainly written AI text and is pulled down by citations, so the reading review is the main cover for heavily cited academic work.

4. **Build the report.**

   ```bash
   python3 aicheck/build_report.py analysis.json -r review.json -o report.html --pdf report.pdf
   ```

   Leave out `-r` only if the state is `unavailable`. `--pdf` needs Playwright and can be dropped if it is missing.

5. **Deliver.** Send `report.html` and the PDF as files. Student work and unpublished manuscripts are private, so do not publish them to a hosted page unless the person asks.

6. **Reply briefly.** Give the percentage as a number, what it means, the score for each part, any hidden-tricks findings, and the top one or two things to fix. Say plainly that the score is an estimate, cannot predict Turnitin's, and may be too low or too high. Suggest comparing it with a real Turnitin report or another detector.

## Reading the result

| State | Shown | Meaning |
|---|---|---|
| `unavailable` | `No score` | Requirements not met |
| `zero` | `0%` | Nothing flagged |
| `asterisk` | `1–19%` | Low. Show the number. Turnitin itself would show `*%`, and the report says so in one line |
| `scored` | `20–100%` | Share of checked words flagged |

A score is not evidence of misconduct, and 0% is not proof of human writing. The check was built and tested with Claude, cannot identify which AI tool was used, and has not been tested on other tools, so it may miss some AI text, including Claude's. Your own reading review may share that blind spot. Never present a low score as safe.

## Recommendations

The goal is work that is honestly the author's and shows it. That is also what lowers a score in a lasting way. Good recommendations ask for what only the author can supply: their data, reasoning, decisions, sources they have read, and their own voice.

- Give prompts and questions, not replacement sentences. AI-written replacements would still be AI text, and the author could not defend them in a viva or review.
- Do not recommend paraphrasers, humanisers, invisible characters, swapped letters, or text hidden in images. Turnitin looks for these, and they turn an authorship question into a misconduct question.
- Do not promise a score. Say that a revised passage may stop being flagged, and that the percentage arithmetic assumes it does.
- If the author says the text is their own, treat a flag as a possible false positive. Formal, formulaic and non-native English writing are known risks. Help them gather drafts and version history instead of rewriting good work.
- If the author used AI, point them to the rule that applies and to disclosure. Many journals and universities allow declared use.
- If someone says they want to conceal AI use that their institution prohibits, say plainly that you will not help hide it, then offer the check and help with disclosure or real rewriting by the author.

## Scope limits to state when relevant

- There is no similarity or plagiarism matching.
- The numeric check supports English only. Turnitin also supports other languages.
- Scoring uses hand-set signals, so there is no measured accuracy rate. Never quote Turnitin's accuracy figures as this check's.
- Report titles and text must not suggest that the output comes from Turnitin.

## Reading review

The automatic check misses plain AI writing and can wrongly flag formal writing. Read every paragraph of `analysis.json` yourself. Your judgement is also fallible, so change a label only when you can give a reason from the text.

### Add a flag when several of these appear together

- Balanced and generic: it could sit in any paper on the topic.
- Summarises what a field says, but makes no argument of its own.
- Even paragraph shape: topic sentence, three supports, tidy close.
- Hedged in a uniform way, with no sign of a decision, surprise, or dead end.
- A sudden change in voice, register, spelling convention or quality from nearby text.
- Citations that look unfamiliar, or claims more specific than their source could support.

### Remove a flag when

- The passage is a method, standard definition, legal wording, or formulaic section where any careful author would write alike.
- It contains details from the author's own data, fieldwork or reasoning.
- Formal phrasing fits the writer's non-native English or the field's conventions, and the content is specific.

### Write review.json

Sentence ids come from `analysis.json`. `from` and `to` are inclusive. Write `note`, `why` and `actions` in plain language for the author.

```json
{
  "reviewed": true,
  "note": "One or two plain sentences on what the review found.",
  "changes": [
    {"from": 12, "to": 15, "label": "ai", "reason": "Generic summary with a voice shift"},
    {"from": 30, "to": 33, "label": "human", "reason": "Standard method wording"}
  ],
  "passages": [
    {"from": 12, "to": 15,
     "why": "Why this passage looks AI-written, in terms the author can act on.",
     "actions": ["Specific action for this passage", "Another specific action"]}
  ]
}
```

Labels: `human`, `ai`, `ai_paraphrased`. Use `ai_paraphrased` only for signs of a word spinner, such as odd synonyms for established terms.

### Writing the actions

Tie each action to the passage's content: which claim needs a source, which data belongs here, which decision the author should explain. Ask questions only the author can answer. Do not write replacement sentences.

## Method and limits

### What mirrors Turnitin (checked November 2026 against guides.turnitin.com)

| Feature | Rule used here |
|---|---|
| Categories | AI-generated only (cyan); AI-generated then AI-paraphrased (purple) |
| Checked text | Prose sentences in long-form writing. Lists, headings, short fragments, captions and bibliography are excluded. Long prose in table cells counts |
| File rules | Under 100 MB; 300–30,000 words of checked prose; .docx, .pdf, .txt, .rtf |
| Segmentation | Overlapping groups of sentences, scored and averaged per sentence |
| Start and end | Stricter threshold in the first and last two sentences |
| Guidance | Not to be the sole basis for adverse action; false positives are possible |

Difference by the author's choice: Turnitin shows 1–19% as `*%` with no highlights. This report always shows the number and highlights.

Turnitin also handles Spanish and Japanese, and one third-party guide reports an Arabic model released in August 2026. This skill's numeric check is English only. Other languages show as unavailable.

Turnitin's published claim is a document false positive rate under 1% for documents with over 20% AI writing, and about 4% at sentence level. Those are Turnitin's figures. Never quote them as this skill's accuracy.

### What differs

- Turnitin's model is private and trained on large datasets. This check is not a copy of it.
- Scores here will not match Turnitin's and cannot predict them.
- There is no similarity or plagiarism matching, because that needs a database of sources.
- Weights in `analyze.py` are hand-set. No labelled corpus was available, so no accuracy rate exists.

### Automatic check signals

Raises the score: overused AI vocabulary, stock phrases, formal transition openers, trailing “-ing” clauses, reflexive lists of three, “not only… but” patterns, uniform sentence length, heavy nominalisation, and tortured phrases from word spinners.

Lowers the score: citations, numbers, named details, first-person statements and contractions.

Sentence score = 40% overlapping five-sentence windows + 40% its paragraph + 20% the sentence with its neighbours. Sentences are flagged at 0.50 or above, in runs of at least two.

### Development tests

| Sample | Expected | Result |
|---|---|---|
| Stereotyped AI essay | High | 100% |
| Personal human-style essay | Low | 0% |
| Formal human-style research paper | Low | 0% |
| Newton's *Opticks*, real human prose (9,857 words) | Low | 0% |
| Human and AI-style paragraphs mixed | Boundary found | Exact paragraph boundary |
| Plain-style AI essay | High | 0% in the automatic check |

All but Newton were written for testing, so they show the check runs as designed, not how accurate it is on real work. The last row is why the reading review is required.

## Scripts

~~~python analyze.py
#!/usr/bin/env python3
"""Extract a document, keep qualifying prose, and score it for AI-writing signals.

Usage: analyze.py INPUT [-o analysis.json]

The scoring is a transparent heuristic, not Turnitin's model and not a trained
classifier. Weights are hand-set.
"""
import argparse, json, math, re, statistics, subprocess, sys, unicodedata
from datetime import datetime, timezone
from pathlib import Path

MIN_WORDS, MAX_WORDS, MAX_BYTES = 300, 30000, 100 * 1024 * 1024
ACCEPTED = {".docx", ".pdf", ".txt", ".rtf"}
THRESHOLD, LOW, HIGH = 0.50, 0.35, 0.65

WORD = re.compile(r"[^\W\d_]+(?:['’-][^\W\d_]+)*|\d+(?:[.,]\d+)*", re.U)

AI_VOCAB = set("""delve delves delving tapestry multifaceted pivotal crucial intricate intricacies
nuanced nuances underscore underscores underscoring underscored landscape landscapes realm realms
foster fosters fostering fostered leverage leverages leveraging leveraged robust comprehensive
holistic seamless seamlessly transformative testament navigate navigating navigates myriad plethora
paramount meticulous meticulously harness harnessing harnesses bolster bolstering garner garnered
showcase showcasing showcases elevate elevating commendable noteworthy vibrant dynamic ever-evolving
ever-changing groundbreaking cutting-edge synergy synergies streamline streamlining empower empowering
empowers unwavering profound resonate resonates pertinent encompass encompasses encompassing
facilitate facilitates facilitating optimize optimizing innovative stakeholders imperative
interplay paradigm overarching""".split())

STOCK = [r"it is (?:important|worth|crucial|essential) to (?:note|recogni[sz]e|consider|understand|acknowledge)",
 r"plays? an? (?:\w+ )?(?:crucial|vital|pivotal|key|significant|important|essential) role",
 r"in today'?s (?:\w+[- ]){0,3}(?:world|society|era|age|landscape)", r"in the (?:\w+ )?(?:realm|landscape|era|world) of",
 r"a (?:deeper|better|nuanced|comprehensive|holistic) understanding of", r"(?:wide|broad|diverse) (?:range|array|spectrum) of",
 r"as (?:we|society|the world) (?:continues?|moves?|navigates?)", r"has (?:emerged|become) as an?", r"cannot be (?:overstated|understated|ignored)",
 r"serves? as an?", r"shed(?:s|ding)? light on", r"paves? the way", r"pave the way", r"stands? as an?", r"at the (?:heart|forefront) of",
 r"in conclusion", r"overall,", r"ultimately,", r"by (?:understanding|addressing|fostering|leveraging|embracing) ", r"not without (?:its )?challenges",
 r"both .{3,40} and .{3,40} alike", r"offers? (?:valuable|unique|significant) insights?", r"valuable insights?", r"far-reaching (?:implications|consequences)",
 r"highlights? the (?:need|importance|significance)", r"remains? (?:a )?(?:critical|essential|vital|key)", r"a (?:complex|delicate) (?:interplay|balance)"]
STOCK_RE = [re.compile(p, re.I) for p in STOCK]

TRANSITION = re.compile(r"^(?:moreover|furthermore|additionally|however|consequently|therefore|thus|nevertheless|in addition|similarly|likewise|as a result|on the other hand|for instance|for example|in contrast|notably|importantly|ultimately|overall|in conclusion|in summary|to conclude|firstly|secondly|thirdly|finally)\b[, ]", re.I)
TAIL_ING = re.compile(r",\s+(?:thereby\s+|thus\s+|ultimately\s+)?(?:highlighting|ensuring|underscoring|emphasi[sz]ing|reflecting|enabling|allowing|fostering|leading|contributing|making|creating|providing|demonstrating|showcasing|reinforcing|illustrating|shaping|improving|enhancing|offering|paving)\b", re.I)
TRIAD = re.compile(r"\b[\w-]+(?: [\w-]+){0,2}, [\w-]+(?: [\w-]+){0,2},? and [\w-]+", re.I)
CONTRAST = re.compile(r"\bnot (?:only|just|merely|simply)\b.{2,80}\b(?:but|rather)\b|\bit(?:'s| is) not (?:about )?.{2,50}[,;—-] ?(?:it(?:'s| is)|but)\b", re.I)
CITE = re.compile(r"\([A-Z][\w'’-]+(?: (?:et al\.?|and|&) ?[A-Z]?[\w'’-]*)?,? (?:19|20)\d\d[a-z]?(?:, ?p+\. ?\d+)?\)|\[\d+(?:[,–-] ?\d+)*\]|\((?:19|20)\d\d\)")
FIRST = re.compile(r"\b(?:I|my|me|we|our|us)\b|\b\w+(?:n't|'ve|'ll|'d)\b|\b(?:I'm|it's|that's|there's)\b")
NOMINAL = re.compile(r"\b\w{4,}(?:tion|tions|ment|ments|ity|ities|ness|ance|ence)\b", re.I)

# Word-spinner output. From the "tortured phrases" literature (Cabanac et al., 2021).
TORTURED = ["counterfeit consciousness", "man-made consciousness", "fake consciousness", "profound learning",
 "profound neural organization", "profound neural network", "colossal information", "enormous information",
 "huge information", "flag commotion", "flag clamor", "arbitrary woodland", "irregular woodland", "irregular backwoods",
 "arbitrary backwoods", "haphazard backwoods", "bosom peril", "bosom malignant growth", "kidney disappointment",
 "sun based vitality", "sun oriented vitality", "wind vitality", "discourse acknowledgment", "facial acknowledgment",
 "picture acknowledgment", "straight relapse", "calculated relapse", "mean square mistake", "standard deviation mistake",
 "AI calculation", "information mining calculation", "organization security", "remote sensor organization",
 "cloud figuring", "lattice figuring", "blockchain innovation arrangement", "savvy city", "savvy home", "keen framework",
 "natural surroundings", "understudy learning results", "instructive establishment", "exploration paper examines"]
TORTURED_RE = re.compile("|".join(re.escape(t) for t in TORTURED), re.I)

STOPWORDS = {"en": "the of and to in is that for it with as was on are this by be from".split(),
 "ms": "yang dan di dengan untuk pada dari ini itu dalam adalah tidak akan kepada oleh juga".split(),
 "es": "el la de que y en los las del se por con para una es".split()}
ABBREV = r"(?:et al|e\.g|i\.e|cf|vs|Dr|Mr|Mrs|Ms|Prof|Fig|Figs|Eq|No|Vol|pp|p|Ed|eds|Inc|Ltd|St|etc|al)"
REF_HEAD = re.compile(r"^(?:\d+\.?\s*|chapter \d+:?\s*)?(?:references?(?: list)?|bibliography|works cited|literature cited|rujukan|daftar pustaka|bibliografi)\s*:?$", re.I)
APPX_HEAD = re.compile(r"^(?:appendix|appendices|lampiran)\b", re.I)
CAPTION = re.compile(r"^(?:figure|fig\.|table|rajah|jadual|chart|plate)\s+\d", re.I)
TOC = re.compile(r"\.{4,}\s*\d+\s*$|\s\d+\s*$")
LISTY = re.compile(r"^\s*(?:[-•*▪◦–]|\(?\d{1,2}[.)]|\(?[a-z][.)])\s+")


def words(t): return WORD.findall(t)


def sentences(text):
    t = re.sub(r"\b" + ABBREV + r"\.", lambda m: m.group(0).replace(".", chr(0x2024)), text)
    parts = re.split(r"(?<=[.!?])[\"'”’)\]]*\s+(?=[\"'“‘(\[]?[A-Z0-9])", t)
    return [p.replace(chr(0x2024), ".").strip() for p in parts if p.strip()]


def is_white(c):
    if c is None: return False
    if isinstance(c, (int, float)): return c >= .99
    c = tuple(c)
    if len(c) in (1, 3): return all(x >= .99 for x in c)
    if len(c) == 4: return all(x <= .01 for x in c)
    return False


def read_docx(path, flags):
    import docx
    from docx.table import Table
    from docx.text.paragraph import Paragraph
    d = docx.Document(path)
    blocks, hidden = [], 0

    def para(p):
        nonlocal hidden
        vis = []
        for r in p.runs:
            f = r.font
            col = None
            try: col = f.color.rgb if f.color is not None else None
            except Exception: pass
            hide = bool(f.hidden) or str(col).upper() == "FFFFFF" or (f.size is not None and f.size.pt <= 2)
            if hide: hidden += len(words(r.text))
            else: vis.append(r.text)
        return "".join(vis).strip() if p.runs else p.text.strip()

    for el in d.element.body.iterchildren():
        if el.tag.endswith("}p"):
            p = Paragraph(el, d)
            s = para(p)
            if not s: continue
            style = (p.style.name or "").lower() if p.style is not None else ""
            ppr = el.pPr
            kind = "heading" if style.startswith(("heading", "title", "subtitle")) else \
                   "list" if "list" in style or (ppr is not None and ppr.numPr is not None) else None
            blocks.append({"text": s, "kind": kind, "page": None})
        elif el.tag.endswith("}tbl"):
            for row in Table(el, d).rows:
                for c in row.cells:
                    s = c.text.strip()
                    if s: blocks.append({"text": s, "kind": "prose_cell" if len(words(s)) >= 40 else "table", "page": None})
    if hidden: flags.append({"type": "Hidden text", "severity": "high", "count": hidden,
        "detail": f"{hidden} words are white, hidden or 2pt-or-smaller. They were left out of scoring. Anyone inspecting the file can find them."})
    return blocks, None


def read_pdf(path, flags):
    import pdfplumber
    hidden, out = 0, []
    gone = lambda o: is_white(o.get("non_stroking_color")) or o.get("size", 12) <= 2
    with pdfplumber.open(path) as pdf:
        n = len(pdf.pages)
        for num, pg in enumerate(pdf.pages, 1):
            hidden += sum(1 for c in pg.chars if gone(c))
            keep = pg.filter(lambda o: o.get("object_type") != "char" or not gone(o))
            lines = [l for l in keep.extract_text_lines() if not re.fullmatch(r"(?:page\s+)?\d{1,4}(?:\s+of\s+\d+)?", l["text"].strip(), re.I)]
            if not lines: continue
            gaps = [b["top"] - a["top"] for a, b in zip(lines, lines[1:]) if b["top"] > a["top"]]
            step = statistics.median(gaps) if gaps else 14
            left = min(l["x0"] for l in lines); right = max(l["x1"] for l in lines)
            sizes = sorted(c.get("size", 12) for c in keep.chars) or [12]
            body = sizes[len(sizes) // 2]
            heady = lambda l: len(words(l["text"])) <= 12 and l["text"].rstrip()[-1:] not in ".,;" and bool(l.get("chars")) and (
                l["chars"][0].get("size", body) > body * 1.1 or "bold" in str(l["chars"][0].get("fontname", "")).lower())
            cur, prev = "", None
            for l in lines:
                t = l["text"].strip()
                brk = prev is None or l["top"] - prev["top"] > 1.35 * step or l["x0"] > left + 8 or LISTY.match(t) or heady(l) or heady(prev) \
                      or (prev["x1"] < right - .15 * (right - left) and prev["text"].rstrip()[-1:] in ".!?:")
                if brk and cur:
                    out.append({"text": cur, "kind": "heading" if heady(prev) else None, "page": num}); cur = ""
                if not cur: cur = t
                elif cur.endswith("-") and t[:1].islower(): cur = cur[:-1] + t
                else: cur += " " + t
                prev = l
            if cur:
                # A paragraph running over the page break continues the previous one.
                out.append({"text": cur, "kind": "heading" if heady(prev) else None, "page": num})
    merged = []
    for b in out:
        if merged and merged[-1]["page"] == b["page"] - 1 and merged[-1]["text"][-1:] not in ".!?:" and b["text"][:1].islower():
            merged[-1]["text"] += " " + b["text"]
        else: merged.append(b)
    if hidden: flags.append({"type": "Hidden text", "severity": "high", "count": hidden,
        "detail": f"{hidden} characters are white or 2pt-or-smaller. They were left out of scoring."})
    return merged, n


def read_text(path):
    raw = Path(path).read_bytes()
    for enc in ("utf-8-sig", "utf-16", "cp1252"):
        try:
            t = raw.decode(enc); break
        except UnicodeError: continue
    return [{"text": re.sub(r"\s*\n\s*", " ", b).strip(), "kind": None, "page": None} for b in re.split(r"\n\s*\n", t.replace("\r", "")) if b.strip()]


def read_rtf(path, wd):
    subprocess.run(["soffice", "--headless", "--convert-to", "txt:Text", "--outdir", str(wd), str(path)], check=True, capture_output=True, timeout=120)
    return read_text(Path(wd) / (Path(path).stem + ".txt"))


def classify(blocks):
    in_refs = False
    for b in blocks:
        t, w = b["text"], len(words(b["text"]))
        head = b["kind"] == "heading" or (w <= 12 and t[-1:] not in ".!?;:,")
        if head and REF_HEAD.match(t.strip()): in_refs, b["kind"] = True, "heading"; continue
        if head and in_refs and APPX_HEAD.match(t.strip()): in_refs, b["kind"] = False, "heading"; continue
        if in_refs: b["kind"] = "reference"; continue
        if b["kind"] == "prose_cell": b["kind"] = "prose"; continue
        if b["kind"] in ("heading", "table"): continue
        if CAPTION.match(t): b["kind"] = "caption"
        elif b["kind"] == "list" or LISTY.match(t): b["kind"] = "list"
        elif head: b["kind"] = "toc" if TOC.search(t) and w > 1 else "heading"
        elif TOC.search(t) and re.search(r"\.{4,}", t): b["kind"] = "toc"
        elif w < 20 or len(sentences(t)) < 1: b["kind"] = "short"
        else: b["kind"] = "prose"
    return blocks


def integrity(blocks, flags):
    text = "\n".join(b["text"] for b in blocks)
    zw = sum(text.count(c) for c in map(chr, (0x200b, 0x200c, 0x200d, 0x2060, 0xfeff)))
    if zw: flags.append({"type": "Invisible characters", "severity": "high", "count": zw, "detail": f"{zw} zero-width characters found. They can split words and change how software reads text."})
    mixed = 0
    for w in re.findall(r"[^\W\d_]{3,}", text):
        sc = set()
        for ch in w:
            n = unicodedata.name(ch, "")
            sc.add("L" if n.startswith("LATIN") else "C" if n.startswith("CYRILLIC") else "G" if n.startswith("GREEK") else "O")
        if "L" in sc and sc & {"C", "G"} and not unicodedata.name(w[0], "").startswith("GREEK"): mixed += 1
    if mixed: flags.append({"type": "Replaced characters", "severity": "high", "count": mixed, "detail": f"{mixed} words mix Latin letters with look-alike Cyrillic or Greek letters."})
    nb = text.count(chr(0xa0))
    if nb > 20 and nb > .05 * max(1, text.count(" ")): flags.append({"type": "Unusual spacing", "severity": "medium", "count": nb, "detail": f"{nb} non-breaking spaces. Check for accidental paste artefacts."})
    for pat, name in [(r"as an ai (?:language )?model|as a large language model|i cannot browse", "Chatbot wording left in the text"),
                      (r"\bregenerate response\b|\bcertainly[!,] here(?:'s| is)\b|\bhere(?:'s| is) (?:a|an|the) (?:revised|rewritten|improved) (?:version|draft)", "Chatbot wording left in the text"),
                      (r"\[(?:insert|your|add) [^\]]{2,40}\]|\bTODO\b", "Unfilled placeholder")]:
        c = len(re.findall(pat, text, re.I))
        if c: flags.append({"type": name, "severity": "high", "count": c, "detail": f"{c} instance(s) found."})


def language(text):
    if not text: return "unknown"
    letters = [c for c in text if c.isalpha()]
    if letters:
        kana = sum(1 for c in letters if 0x3040 <= ord(c) <= 0x30ff or 0x4e00 <= ord(c) <= 0x9fff)
        if kana / len(letters) > .3: return "ja"
        if sum(1 for c in letters if 0x600 <= ord(c) <= 0x6ff) / len(letters) > .3: return "ar"
    w = [x.lower() for x in words(text)][:6000]
    sc = {k: sum(w.count(s) for s in v) / max(1, len(w)) for k, v in STOPWORDS.items()}
    best = max(sc, key=sc.get)
    return best if sc[best] > .08 else "unknown"


def sent_features(s):
    w = words(s)
    low = [x.lower() for x in w]
    vocab = [x for x in low if x in AI_VOCAB]
    stock = [m.group(0) for r in STOCK_RE for m in r.finditer(s)]
    tort = [m.group(0) for m in TORTURED_RE.finditer(s)]
    caps = len(re.findall(r"(?<=[a-z,] )[A-Z][a-z]+", s))
    return {"n": len(w), "vocab": vocab, "stock": stock, "tort": tort,
        "trans": bool(TRANSITION.match(s)), "ing": bool(TAIL_ING.search(s)), "triad": bool(TRIAD.search(s)),
        "contrast": bool(CONTRAST.search(s)), "spec": bool(re.search(r"\d", CITE.sub("", s)) or CITE.search(s) or caps >= 1),
        "first": bool(FIRST.search(s)), "nom": len(NOMINAL.findall(s))}


def window_logit(fs):
    n = max(1, sum(f["n"] for f in fs)); k = len(fs); per = 100 / n
    v = sum(len(f["vocab"]) for f in fs) * per
    p = sum(len(f["stock"]) for f in fs) * per
    tort = sum(len(f["tort"]) for f in fs) * per
    share = lambda key: sum(1 for f in fs if f[key]) / k
    nom = sum(f["nom"] for f in fs) * per
    L = [f["n"] for f in fs]
    cv = statistics.pstdev(L) / (sum(L) / k) if k > 1 else .6
    uniform = min(1, max(0, (.55 - cv) / .35))
    return (-2.9 + .6 * min(v, 6) + .95 * min(p, 4) + 2.0 * share("trans") + 2.4 * share("ing") + 1.2 * share("triad")
            + 1.6 * share("contrast") + 1.4 * uniform + .13 * max(0, nom - 5) - 1.9 * share("spec") - 2.4 * share("first") + 1.3 * min(tort, 5))


def score(sents):
    """Blend three views so one AI-like paragraph does not smear into its neighbours:
    overlapping windows, the sentence's own paragraph, and the sentence with its neighbours."""
    fs = [s["f"] for s in sents]
    n = len(fs); size = 5
    if n == 0: return
    sig = lambda x: 1 / (1 + math.exp(-x))
    tot, cnt = [0.0] * n, [0] * n
    for i in range(max(1, n - size + 1)):
        win = list(range(i, min(n, i + size)))
        p = sig(window_logit([fs[j] for j in win]))
        for j in win: tot[j] += p; cnt[j] += 1
    blocks = {}
    for i, s in enumerate(sents): blocks.setdefault(s["block"], []).append(i)
    bp = {b: sig(window_logit([fs[j] for j in ids])) for b, ids in blocks.items()}
    for i, s in enumerate(sents):
        own = sig(window_logit([fs[j] for j in range(max(0, i - 1), min(n, i + 2))]))
        s["score"] = round(.4 * tot[i] / cnt[i] + .4 * bp[s["block"]] + .2 * own, 3)


def label(sents, t):
    """Flag runs of at least two sentences. Be stricter at the very start and end,
    where generic introductions and conclusions cause false positives."""
    n = len(sents); on = []
    for i, s in enumerate(sents):
        lim = t + .1 if i < 2 or i >= n - 2 else t
        on.append(s["score"] >= lim)
    i = 0
    while i < n:
        if on[i]:
            j = i
            while j + 1 < n and on[j + 1]: j += 1
            if j == i: on[i] = False
            i = j + 1
        else: i += 1
    return on


def signals(f):
    out = []
    if f["tort"]: out.append(("spinner", ", ".join(f["tort"])))
    if f["stock"]: out.append(("stock", "; ".join(dict.fromkeys(x.lower() for x in f["stock"]))))
    if f["vocab"]: out.append(("vocab", ", ".join(dict.fromkeys(f["vocab"]))))
    if f["ing"]: out.append(("tail", "trailing -ing clause"))
    if f["trans"]: out.append(("trans", "formal transition opener"))
    if f["contrast"]: out.append(("contrast", "“not only… but” pattern"))
    if f["triad"]: out.append(("triad", "list of three"))
    if not f["spec"]: out.append(("nospec", "no data, source or named detail"))
    return out


def summarise(sents):
    q = sum(s["words"] for s in sents)
    ai = sum(s["words"] for s in sents if s["label"] == "ai")
    para = sum(s["words"] for s in sents if s["label"] == "ai_paraphrased")
    return q, ai, para


def state_of(pct):
    return "zero" if pct == 0 else "asterisk" if pct < 20 else "scored"


def main():
    ap = argparse.ArgumentParser(); ap.add_argument("input"); ap.add_argument("-o", "--output", default="analysis.json")
    a = ap.parse_args(); path = Path(a.input); flags, reason = [], None
    size = path.stat().st_size; ext = path.suffix.lower(); pages, blocks = None, []
    if ext not in ACCEPTED: reason = f"{ext or 'This file type'} is not an accepted type (.docx, .pdf, .txt, .rtf)."
    elif size >= MAX_BYTES: reason = "The file is 100 MB or larger."
    else:
        if ext == ".docx": blocks, pages = read_docx(path, flags)
        elif ext == ".pdf": blocks, pages = read_pdf(path, flags)
        elif ext == ".txt": blocks = read_text(path)
        else: blocks = read_rtf(path, Path(a.output).resolve().parent)
    classify(blocks); integrity(blocks, flags)
    prose = " ".join(b["text"] for b in blocks if b["kind"] == "prose")
    lang = language(prose or " ".join(b["text"] for b in blocks))
    sents = []
    for bi, b in enumerate(blocks):
        if b["kind"] != "prose": continue
        for s in sentences(b["text"]):
            sents.append({"id": len(sents), "block": bi, "text": s, "words": len(words(s)), "f": sent_features(s)})
    q = sum(s["words"] for s in sents)
    if not reason:
        if not blocks or not "".join(b["text"] for b in blocks).strip(): reason = "No readable text was found. A scanned PDF needs OCR first."
        elif q < MIN_WORDS: reason = f"Only {q} words of qualifying prose were found. The minimum is {MIN_WORDS}."
        elif q > MAX_WORDS: reason = f"{q} words of qualifying prose exceeds the {MAX_WORDS:,}-word limit. Split the document and check each part."
        elif lang != "en": reason = f"The prose appears to be in '{lang}'. The numeric check is built for English only."
    if reason:
        res_state, pct = "unavailable", None
        for s in sents: s["score"], s["label"] = 0, "human"
    else:
        score(sents)
        on = label(sents, THRESHOLD)
        for s, o in zip(sents, on):
            s["label"] = ("ai_paraphrased" if s["f"]["tort"] else "ai") if o else "human"
        # Spinner hits nearby turn the whole flagged run purple.
        for i, s in enumerate(sents):
            if s["label"] == "ai" and any(sents[j]["f"]["tort"] for j in range(max(0, i - 2), min(len(sents), i + 3))): s["label"] = "ai_paraphrased"
        lo = sum(x["words"] for x, o in zip(sents, label(sents, HIGH)) if o) * 100 / q
        hi = sum(x["words"] for x, o in zip(sents, label(sents, LOW)) if o) * 100 / q
        _, ai, para = summarise(sents)
        pct = round((ai + para) * 100 / q); res_state = state_of(pct)
    for s in sents:
        f = s.pop("f"); s["signals"] = [{"kind": k, "detail": d} for k, d in signals(f)]
    body = {"file": path.name, "generated": datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M UTC"), "pages": pages, "bytes": size,
        "language": lang, "state": res_state, "percent": pct, "unavailable_reason": reason,
        "band": None if reason else [round(lo), round(hi)], "total_words": sum(len(words(b["text"])) for b in blocks), "qualifying_words": q,
        "excluded": {k: sum(len(words(b["text"])) for b in blocks if b["kind"] == k) for k in ("heading", "list", "table", "reference", "caption", "toc", "short")},
        "flags": flags, "blocks": blocks, "sentences": sents}
    Path(a.output).write_text(json.dumps(body, ensure_ascii=False, indent=1))
    print(json.dumps({k: body[k] for k in ("file", "state", "percent", "band", "language", "qualifying_words", "total_words", "unavailable_reason")}, ensure_ascii=False))
    print("flags:", [f["type"] for f in flags])
    print("flagged sentences:", [s["id"] for s in sents if s["label"] != "human"])


if __name__ == "__main__":
    main()
~~~

~~~python build_report.py
#!/usr/bin/env python3
"""Build the self-check report from analysis.json and an optional review.json.

Usage: build_report.py analysis.json [-r review.json] [-o report.html] [--pdf report.pdf]
"""
import argparse, html, json, re
from pathlib import Path

ADVICE = {
 "spinner": "Replace the odd phrase with the standard term in your field, then rewrite the sentence from the source. Do not put it through a paraphrasing tool again.",
 "stock": "Cut the framing phrase and state the claim itself. Ask what you actually want the reader to accept here.",
 "vocab": "Say it the way you would explain it to your supervisor. Keep a term only if your field uses it precisely.",
 "tail": "End the sentence at the main point. If the “-ing” clause makes a real claim, give it its own sentence and support it.",
 "trans": "Remove the connector if the logic is already clear. If it is not, show how the two ideas relate.",
 "contrast": "Say what you found directly. Keep the contrast only if someone actually holds the view you are rejecting.",
 "triad": "Keep only the items you can support. Two specific points are stronger than three general ones.",
 "nospec": "Add what only you could add: your data, a cited source you have read, a real example, or a decision you made and why.",
}
SIGNAL_NAMES = {"spinner": "Word-spinner phrasing", "stock": "Stock phrasing", "vocab": "Overused vocabulary", "tail": "Trailing “-ing” clauses",
 "trans": "Repeated formal transitions", "contrast": "“Not only… but” patterns", "triad": "Reflexive lists of three", "nospec": "No specific evidence"}
KIND_NAMES = {"heading": "Headings", "list": "Lists", "table": "Tables", "reference": "References", "caption": "Captions", "toc": "Contents lines", "short": "Short fragments"}
LABELS = {"ai": "Looks written by AI", "ai_paraphrased": "Looks written by AI, then reworded by a tool"}
E = lambda s: html.escape(str(s), quote=True)
pc = lambda n, q: round(n * 100 / q) if q else 0


def apply_review(a, r):
    by = {s["id"]: s for s in a["sentences"]}
    for c in r.get("changes", []):
        for i in range(c["from"], c["to"] + 1):
            if i in by:
                by[i]["label"] = c["label"]; by[i]["review_reason"] = c.get("reason", "")


def passages(a, r):
    out, cur = [], None
    for s in a["sentences"]:
        flagged = s["label"] != "human"
        if flagged and cur and cur["block"] == s["block"] and cur["label"] == s["label"]:
            cur["sents"].append(s)
        elif flagged:
            cur = {"block": s["block"], "label": s["label"], "sents": [s]}; out.append(cur)
        else: cur = None
    notes = r.get("passages", [])
    for p in out:
        ids = {s["id"] for s in p["sents"]}
        p["words"] = sum(s["words"] for s in p["sents"])
        p["page"] = a["blocks"][p["block"]].get("page")
        p["section"] = next((b["text"].replace("\t", " ") for b in reversed(a["blocks"][:p["block"]]) if b["kind"] == "heading"), f"Paragraph {p['block'] + 1}")
        p["note"] = next((n for n in notes if ids & set(range(n["from"], n["to"] + 1))), None)
        c = {}
        for s in p["sents"]:
            for g in s["signals"]: c.setdefault(g["kind"], []).append(g["detail"])
        p["signals"] = sorted(c.items(), key=lambda kv: -len(kv[1]))
    return sorted(out, key=lambda p: -p["words"])


def passage_html(i, p, q, cum):
    first = p["sents"][0]["text"]
    excerpt = first[:150] + ("…" if len(first) > 150 else "")
    where = f"Page {p['page']}" if p["page"] else p["section"][:70]
    n = p["note"] or {}
    why = n.get("why") or "It contains: " + ", ".join(SIGNAL_NAMES[k].lower() for k, _ in p["signals"][:4]) + "."
    acts = n.get("actions") or [ADVICE[k] for k, _ in p["signals"][:3]] or ["Reread this passage against your notes and rewrite anything you cannot explain unaided."]
    chips = "".join(f'<li><b>{E(SIGNAL_NAMES[k])}</b><span>{E("; ".join(dict.fromkeys(v))[:150])}</span></li>' for k, v in p["signals"][:4])
    cls = "para" if p["label"] == "ai_paraphrased" else "ai"
    return f'''<article class="rec"><header><span class="dot {cls}"></span><a href="#s{p['sents'][0]['id']}">{i}. {E(where)}</a>
<span class="meta">{p['words']} words · your score after fixing this and the passages above: {cum}%</span></header>
<blockquote>{E(excerpt)}</blockquote><p><b>Why it was flagged:</b> {E(why)}</p><p><b>What to do:</b></p><ul class="acts">{"".join(f"<li>{E(x)}</li>" for x in acts)}</ul>
<details><summary>Wording that triggered the automatic check</summary><ul class="sig">{chips or "<li>The automatic check found nothing here. This came from Claude reading the text.</li>"}</ul></details></article>'''


def doc_html(a):
    by_block = {}
    for s in a["sentences"]: by_block.setdefault(s["block"], []).append(s)
    parts = []
    for i, b in enumerate(a["blocks"]):
        if b["kind"] == "prose":
            spans = []
            for s in by_block.get(i, []):
                c = {"ai": ' class="hl ai"', "ai_paraphrased": ' class="hl para"'}.get(s["label"], "")
                spans.append(f'<span id="s{s["id"]}"{c} title="{"sentence " + str(s["id"])}">{E(s["text"])}</span>')
            parts.append(f"<p>{' '.join(spans)}</p>")
        elif b["kind"] == "heading": parts.append(f"<h4>{E(b['text'])}</h4>")
        else: parts.append(f'<p class="ex"><span>{E(KIND_NAMES.get(b["kind"], b["kind"]))}, not counted</span>{E(b["text"][:400])}</p>')
    return "\n".join(parts)


CSS = """:root{--bg:#fbfaf7;--card:#fff;--ink:#1d2329;--mute:#5e6873;--line:#e2e0d9;--accent:#0b6e76;--cy:#b4eef2;--cyd:#0a97a3;--pu:#dcc8f8;--pud:#7443c9;--warn:#fdf0d5;--warnink:#7a4b00;--bad:#fbe3e0;--badink:#9b2415}
@media (prefers-color-scheme:dark){:root:not([data-theme=light]){--bg:#14181c;--card:#1c2228;--ink:#eceef0;--mute:#a2acb6;--line:#333b43;--accent:#6fd3db;--cy:#0f5a61;--cyd:#46cdd9;--pu:#4b3280;--pud:#b796ee;--warn:#46350f;--warnink:#f4d089;--bad:#512018;--badink:#f6b3a8}}
*{box-sizing:border-box}body{margin:0;background:var(--bg);color:var(--ink);font:16px/1.6 Georgia,'Times New Roman',serif}
.sans,h1,h2,h3,h4,header,.meta,button,table,summary,.tile,.note{font-family:-apple-system,'Segoe UI',Roboto,Helvetica,Arial,sans-serif}
.wrap{max-width:900px;margin:auto;padding:32px 16px 64px}h1{font-size:26px;margin:0 0 4px}h2{font-size:19px;margin:36px 0 10px;padding-top:12px;border-top:1px solid var(--line)}
.sub{color:var(--mute);font-size:14px;margin:0}.score{display:grid;grid-template-columns:minmax(170px,230px) 1fr;gap:16px;margin:22px 0}@media(max-width:640px){.score{grid-template-columns:1fr}}
.tile{background:var(--card);border:1px solid var(--line);border-radius:10px;padding:18px}.big{font-size:60px;font-weight:700;line-height:1.05;color:var(--accent)}.big.small{font-size:36px}.tile small{display:block;color:var(--mute);line-height:1.4;margin-top:6px}
button.cat{all:unset;box-sizing:border-box;cursor:pointer;display:flex;gap:10px;align-items:center;width:100%;padding:10px 4px;border-bottom:1px solid var(--line);font-size:15px}button.cat:last-child{border:0}button.cat:hover,button.cat:focus-visible{background:var(--bg)}button.cat b{margin-left:auto}
.dot{width:13px;height:13px;border-radius:3px;display:inline-block;flex:none}.dot.ai,.hl.ai{background:var(--cy)}.dot.para,.hl.para{background:var(--pu)}.hl{padding:1px 0}
.note{border-radius:8px;padding:12px 14px;font-size:14px;margin:12px 0;background:var(--warn);color:var(--warnink)}.note.bad{background:var(--bad);color:var(--badink)}
.facts{display:grid;grid-template-columns:repeat(auto-fit,minmax(135px,1fr));gap:8px;margin:0;font-size:14px}.facts div{background:var(--card);border:1px solid var(--line);border-radius:8px;padding:8px 10px}.facts dt{color:var(--mute);font-size:12px}.facts dd{margin:0;font-weight:600;word-break:break-word}
.rec{background:var(--card);border:1px solid var(--line);border-radius:10px;padding:14px 16px;margin:10px 0}.rec header{display:flex;flex-wrap:wrap;gap:8px;align-items:center;font-weight:600}.rec a{color:var(--accent)}.meta{color:var(--mute);font-weight:400;font-size:13px}
blockquote{margin:10px 0;padding-left:12px;border-left:3px solid var(--line);color:var(--mute);font-size:15px}.acts{padding-left:20px;margin:6px 0}.sig{list-style:none;padding:0;font-size:13px}.sig li{margin:5px 0}.sig b{margin-right:6px}summary{cursor:pointer;color:var(--mute);font-size:13px}
.doc{background:var(--card);border:1px solid var(--line);border-radius:10px;padding:22px}.doc p{margin:0 0 14px}.ex{color:var(--mute);font-size:13px;border-left:3px solid var(--line);padding-left:10px}.ex span{display:block;font:11px -apple-system,'Segoe UI',sans-serif;text-transform:uppercase;letter-spacing:.04em}
.focus-ai .hl.para,.focus-para .hl.ai{background:none}table{width:100%;border-collapse:collapse;font-size:14px}th,td{text-align:left;padding:7px 6px;border-bottom:1px solid var(--line);vertical-align:top}.tools{display:flex;gap:8px;margin:0 0 10px}.tools button{font-size:13px;padding:6px 10px;border:1px solid var(--line);background:var(--card);color:var(--ink);border-radius:6px;cursor:pointer}
@media print{body{background:#fff}.tools,button.cat{break-inside:avoid}details{display:block}.rec,.tile{break-inside:avoid}}"""
JS = """(function(){var d=document.getElementById('doc');document.querySelectorAll('[data-cat]').forEach(function(b){b.addEventListener('click',function(){var c=b.dataset.cat;d.classList.remove('focus-ai','focus-para');d.classList.add('focus-'+c);var f=d.querySelector('.hl.'+c);if(f)f.scrollIntoView({behavior:'smooth',block:'center'})})});var r=document.getElementById('all');if(r)r.addEventListener('click',function(){d.classList.remove('focus-ai','focus-para')});var x=document.getElementById('ex');if(x)x.addEventListener('click',function(){d.querySelectorAll('.ex').forEach(function(e){e.hidden=!e.hidden})})})();"""


def by_section(a):
    cur, sec = None, {}
    for i, b in enumerate(a["blocks"]):
        if b["kind"] == "heading":
            m = re.match(r"(\d+)\.\d", b["text"])
            if m: cur = f"Chapter {m.group(1)}"
            elif b["text"].upper().startswith(("APPENDIX", "LAMPIRAN")): cur = "Appendices"
        sec[i] = cur
    rows = {}
    for s in a["sentences"]:
        c = sec[s["block"]]
        if c:
            r = rows.setdefault(c, [0, 0]); r[1] += s["words"]
            if s["label"] != "human": r[0] += s["words"]
    if len(rows) < 2: return ""
    body = "".join(f"<tr><td>{k}</td><td>{pc(f, t)}%</td><td>{f:,} of {t:,} words</td></tr>" for k, (f, t) in rows.items())
    return f"<h2>Score for each part</h2><table><tr><th>Part</th><th>Score</th><th>Flagged</th></tr>{body}</table>"


def build(a, r):
    q = a["qualifying_words"]; st = a["state"]; auto = a.get("percent")
    reviewed = bool(r.get("reviewed"))
    if st != "unavailable": apply_review(a, r)
    ai = sum(s["words"] for s in a["sentences"] if s["label"] == "ai")
    pa = sum(s["words"] for s in a["sentences"] if s["label"] == "ai_paraphrased")
    if st == "unavailable": pct = None
    else:
        pct = round((ai + pa) * 100 / q); st = "zero" if pct == 0 else "asterisk" if pct < 20 else "scored"
    shown = "No score" if pct is None else f"{pct}%"
    big = "big small" if pct is None else "big"
    cap = {"unavailable": "This file could not be scored.",
           "zero": "of your text looks AI-written. Nothing was flagged.",
           "asterisk": "of your text looks AI-written. This is low, but read the flagged passages.",
           "scored": "of your text looks AI-written."}[st]
    how = f"<small>Automatic check: {auto}%. Claude reading the full text: {pct}%.</small>" if reviewed and pct is not None else ""
    ps = passages(a, r) if st != "unavailable" else []
    flagged = ai + pa; recs = []; rem = flagged; steps = None
    for i, p in enumerate(ps, 1):
        rem -= p["words"]; cum = pc(rem, q); recs.append(passage_html(i, p, q, cum))
        if steps is None and cum < 20: steps = i
    breakdown = f'''<button class="cat" data-cat="ai"><span class="dot ai"></span>{LABELS["ai"]}<b>{pc(ai, q)}%</b></button>
<button class="cat" data-cat="para"><span class="dot para"></span>{LABELS["ai_paraphrased"]}<b>{pc(pa, q)}%</b></button>'''
    note = ""
    if st == "unavailable": note += f'<div class="note bad"><b>Why there is no score:</b> {E(a["unavailable_reason"])}</div>'
    elif st == "asterisk": note += '<div class="note">On Turnitin itself, any score from 1% to 19% appears as *% with no number. This report shows the number so you can see it.</div>'
    if not reviewed and st != "unavailable": note += '<div class="note">Only the automatic check was used, so AI text written in a plain style may have been missed.</div>'
    flags = "".join(f'<div class="note{" bad" if f["severity"] == "high" else ""}"><b>{E(f["type"])}.</b> {E(f["detail"])}</div>' for f in a["flags"]) or '<p>Nothing found. There is no hidden text, no swapped letters and no leftover chatbot wording.</p>'
    excl = [(k, v) for k, v in a["excluded"].items() if v]
    excl_rows = "".join(f"<tr><td>{KIND_NAMES[k]}</td><td>{v:,} words</td></tr>" for k, v in excl) or "<tr><td colspan=2>Everything was counted</td></tr>"
    if st == "scored" and steps: lead = f"<p>The biggest passages come first. If the first {steps} stop being flagged, your score falls below 20%. If all {len(ps)} do, it falls to 0%.</p>"
    elif st in ("scored", "asterisk"): lead = f"<p>The biggest passages come first. If all {len(ps)} stop being flagged, your score falls to 0%.</p>"
    elif st == "zero": lead = "<p>Nothing needs fixing. Keep your drafts and notes in case anyone asks how you wrote the work.</p>"
    else: lead = "<p>Fix the problem above, then run the check again.</p>"
    if ps: lead += "<p>Fix a passage by rewriting it yourself from your notes and sources, not by swapping words. A fix may not change your score on Turnitin or any other detector.</p>"
    recs_block = "".join(recs)
    disclosure = '''<h2>If you used AI in this work</h2><ul class="acts"><li>Check the rules from your university, supervisor or journal.</li><li>Declare the AI use the way those rules ask, including the tool and what you used it for.</li><li>Do not use a paraphrasing or “humaniser” tool. Turnitin looks for these, and using one can look worse than the original text.</li></ul><h2>If you did not use AI, but text was flagged</h2><ul class="acts"><li>Honest writing can be flagged, especially formal writing and writing by non-native English speakers.</li><li>Keep your drafts, version history, notes and data files as proof of how you worked.</li><li>Turnitin says its score should not be the only basis for action against a student, so you can ask to explain your process.</li></ul>'''
    reader = f"<p>{E(r['note'])}</p>" if r.get("note") else ""
    pages = f"<div><dt>Pages</dt><dd>{a['pages']}</dd></div>" if a.get("pages") else ""
    left = a["total_words"] - q
    return f'''<!doctype html><html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1"><title>AI Writing Self-Check</title><style>{CSS}</style></head><body><main class="wrap">
<h1>AI Writing Self-Check</h1><p class="sub sans">{E(a["file"])} · {E(a["generated"])}</p>
<p class="note">This is an estimate, not a Turnitin result, and your real Turnitin score may be different. A flag does not prove that AI was used, and a low score does not prove that a person wrote the text. This check was built and tested with Claude and cannot tell which AI tool was used, so it may miss some AI text, including text written by Claude.</p>
<section class="score"><div class="tile"><div class="{big}">{shown}</div><small>{cap}</small>{how}</div><div class="tile"><strong>What was found</strong>{breakdown}<small>Click a line to jump to that text in your document.</small></div></section>
{note}<dl class="facts"><div><dt>Words in the file</dt><dd>{a["total_words"]:,}</dd></div><div><dt>Words checked</dt><dd>{q:,}</dd></div><div><dt>Words not counted</dt><dd>{left:,}</dd></div>{pages}</dl>
{by_section(a)}<h2>What to fix first</h2>{reader}{lead}{recs_block}
<h2>Hidden tricks check</h2><p class="sub">This looks for things that are often used to fool detectors.</p>{flags}{disclosure}
<h2>Text not counted in the score</h2><table>{excl_rows}</table>
<h2>Your document, with flagged text highlighted</h2><div class="tools"><button id="all">Show all highlights</button><button id="ex">Show or hide text that was not counted</button></div><div id="doc" class="doc">{doc_html(a)}</div>
<h2>How this score works</h2><p class="sub">The score is the share of your checked words that were flagged. Like Turnitin, it counts full paragraphs and leaves out references, headings, lists, short table text and captions. Each document needs 300 to 30,000 checked words. An automatic check looks for common AI wording, then Claude reads the text for sudden changes in style. Neither has a measured accuracy rate, and neither compares your text with other sources, so this is not a plagiarism check.</p>
</main><script>{JS}</script></body></html>''', pct, st


def main():
    ap = argparse.ArgumentParser(); ap.add_argument("analysis"); ap.add_argument("-r", "--review"); ap.add_argument("-o", "--output", default="report.html"); ap.add_argument("--pdf")
    x = ap.parse_args()
    a = json.loads(Path(x.analysis).read_text()); r = json.loads(Path(x.review).read_text()) if x.review else {}
    out, pct, st = build(a, r); Path(x.output).write_text(out)
    print(json.dumps({"report": x.output, "state": st, "percent": pct}))
    if x.pdf:
        from playwright.sync_api import sync_playwright
        with sync_playwright() as p:
            b = p.chromium.launch(); g = b.new_page(); g.goto(Path(x.output).resolve().as_uri()); g.emulate_media(media="print")
            g.pdf(path=x.pdf, format="A4", print_background=True, margin=dict(top="14mm", bottom="14mm", left="12mm", right="12mm")); b.close()
        print("pdf:", x.pdf)


if __name__ == "__main__":
    main()
~~~
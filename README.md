# AI Writing Self-Check

![AI Writing Self-Check poster](assets/poster.png)

A Claude skill that estimates how much of an English document looks AI-written, then shows what to fix. Use it to review your own thesis, essay or manuscript before an examiner, lecturer or journal sees it.

This is an independent project. It is not affiliated with or endorsed by Turnitin, and it cannot predict your Turnitin score.

## What you get

- A percentage score, including a score for each chapter
- Highlighted passages, ranked with the biggest first
- A reason for each flag and steps you can take
- A check for hidden text, swapped letters and leftover chatbot wording
- A report in HTML and PDF

See the [sample report](examples/sample-report.pdf), made from a test document, and the [one-page manual](docs/AI-Writing-Self-Check-Quick-Manual.pdf).

## Install

1. Download [`skill/ai-writing-self-check.zip`](skill/ai-writing-self-check.zip). Do not unzip it.
2. In Claude, go to **Customize > Skills** and click **+**.
3. Choose **+ Create skill**, then **Upload a skill**, and upload the ZIP.

Your Claude plan must allow custom skills, and code execution must be available.

## Use

Attach a .docx, .pdf, .txt or .rtf file to a new chat, then type:

```text
/ai-writing-self-check Check this document for AI.
```

## How the score works

The score is the share of checked words that were flagged. References, headings, lists, captions and short table text are not counted. Documents need 300 to 30,000 checked words.

An automatic check looks for common AI wording. Claude then reads the text for sudden changes in style. The report shows both results.

| Score | Meaning |
| --- | --- |
| 0% | Nothing was flagged. This does not prove the text is human-written. |
| 1–19% | Low. Turnitin itself shows this range as *% with no number. |
| 20–100% | More text was flagged. Start with the biggest passages. |

## Limits

- The score is an estimate, and its accuracy has not been measured.
- A flag is not proof that AI was used. Formal writing and writing by non-native English speakers can be flagged wrongly.
- A low score is not proof that a person wrote the text. The check was built and tested with Claude and cannot tell which AI tool was used, so it may miss some AI text, including text written by Claude. Compare it with a real Turnitin report or another detector.
- It scores English only.
- It does not check plagiarism or similarity.
- It does not rewrite your text or help you hide AI use. Follow your institution's or journal's rules on declaring AI.

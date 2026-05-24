<h1 align="center">PDF to Infographic</h1>

<p align="center">
  <b>Four steps, one pipeline.</b> Transform any academic PDF into a hand-drawn cartoon infographic.
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> &middot;
  <a href="#how-it-works">How It Works</a> &middot;
  <a href="#configuration">Configuration</a> &middot;
  <a href="#examples">Examples</a> &middot;
  <a href="#advanced-usage">Advanced</a>
</p>

---

## Features

- **One command** to go from PDF to infographic (`run_pipeline.py`)
- **CJK-robust** text extraction using PyMuPDF `rawdict` mode (fixes garbled Chinese/Japanese/Korean output)
- **Heuristic content analysis** -- no LLM required for steps 1-3
- **Prompt-only mode** -- generate & edit the image prompt before calling any API
- **Flexible image backend** -- OpenAI, Azure, or any OpenAI-compatible endpoint
- **WorkBuddy native** -- can use built-in `ImageGen` tool instead of Step 4

## Quick Start

```bash
# 1. Clone
git clone https://github.com/jiaqihriver/pdf-to-infographic.git
cd pdf-to-infographic

# 2. Install dependencies
pip install -r requirements.txt

# 3. Run (prompt only -- no API key needed)
python run_pipeline.py paper.pdf ./output --prompt-only

# 4. Review & edit the prompt
cat ./output/prompt.txt

# 5. Generate image (requires OPENAI_API_KEY)
export OPENAI_API_KEY="sk-..."
python scripts/generate_infographic.py ./output/prompt.txt ./output
```

Or do everything in one shot:

```bash
python run_pipeline.py paper.pdf ./output
```

## How It Works

```
PDF ──[Step 1]──> extracted_text.txt
                          │
                   [Step 2]
                          │
                   content_summary.json
                          │
                   [Step 3]
                          │
                   prompt.txt
                          │
                   [Step 4]
                          │
                   output/infographic.png
```

| Step | Script | Description |
|:----:|--------|-------------|
| 1 | `scripts/extract_pdf.py` | Extract text via PyMuPDF `rawdict` mode. Decodes CJK characters that `get_text()` garbles. |
| 2 | `scripts/analyze_content.py` | Heuristic analysis: detect language, extract key numbers, split sections, build structured JSON. |
| 3 | `scripts/build_prompt.py` | Convert JSON into a natural-language prompt optimized for hand-drawn infographic style. |
| 4 | `scripts/generate_infographic.py` | Call OpenAI-compatible image API with the prompt and save the result as PNG. |

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `OPENAI_API_KEY` | *(none)* | Your OpenAI (or compatible) API key |
| `OPENAI_BASE_URL` | OpenAI default | Override for Azure / local endpoints |
| `IMAGE_MODEL` | `gpt-image-1` | Model identifier |
| `IMAGE_SIZE` | `1536x1024` | Canvas size (`1536x1024` = 3:2 landscape) |
| `IMAGE_QUALITY` | `high` | `low` / `medium` / `high` |

### Customizing the Output Style

Edit the `STYLE_SUFFIX` constant at the top of `scripts/build_prompt.py` to change colors, line style, or layout. The default prompt requests:

- Hand-drawn sketch with thick black outlines
- Soft pastel palette (yellow, blue, green, pink)
- 2x3 grid layout for 6 content sections
- Key numbers as large "data hero" figures
- All text in the paper's detected language (Chinese or English)

### Limiting Page Count

In `scripts/extract_pdf.py`, change `MAX_PAGES` at line 25:

```python
MAX_PAGES = 8  # Only extract the first 8 pages
```

## Examples

The `examples/` directory contains a real output generated from this paper:

> **《中国英语能力等级量表》研究综述** -- Li & Gu (2019), *Foreign Languages and Translation*

| File | Description |
|------|-------------|
| `examples/sample_summary.json` | Content analysis result (Step 2 output) |
| `examples/sample_prompt.txt` | Generated image prompt (Step 3 output) |
| `examples/sample_output.png` | Final infographic (Step 4 output) |

## Advanced Usage

### Run Steps Individually

```bash
python scripts/extract_pdf.py         paper.pdf         ./output/text.txt
python scripts/analyze_content.py     ./output/text.txt ./output/summary.json
python scripts/build_prompt.py        ./output/summary.json ./output/prompt.txt
python scripts/generate_infographic.py ./output/prompt.txt ./output
```

This gives you full control at every stage -- edit the JSON or prompt between steps.

### Use with WorkBuddy / CodeBuddy

If you're running inside WorkBuddy, skip Step 4 and use the built-in `ImageGen` tool:

```python
# In a WorkBuddy conversation, after Step 3:
DeferExecuteTool(
    toolName="ImageGen",
    params={
        "prompt": open("./output/prompt.txt").read(),
        "size": "1536x1024",
        "quality": "high",
        "style": "natural",
        "output_dir": "./output"
    }
)
```

See `docs/WORKBUDDY_SKILL.md` for the full WorkBuddy skill integration guide.

### Use a Different Image Provider

The pipeline generates a plain-text prompt (Step 3). You can feed it into any image generation service:

```bash
# Copy the prompt and paste into your tool of choice
cat ./output/prompt.txt | pbcopy  # macOS: copy to clipboard
```

Compatible with DALL-E 3, Stable Diffusion, Midjourney (via API), Hunyuan, etc.

## Project Structure

```
pdf-to-infographic/
├── run_pipeline.py                  # One-shot convenience wrapper
├── requirements.txt                 # pymupdf, openai, Pillow
├── README.md
├── LICENSE                          # MIT
├── .gitignore
├── scripts/
│   ├── extract_pdf.py               # Step 1: PDF -> text (rawdict CJK fix)
│   ├── analyze_content.py           # Step 2: text -> JSON summary
│   ├── build_prompt.py              # Step 3: JSON -> image prompt
│   └── generate_infographic.py      # Step 4: prompt -> PNG (OpenAI API)
├── examples/
│   ├── sample_summary.json          # Example: content analysis output
│   ├── sample_prompt.txt            # Example: generated prompt
│   └── sample_output.png            # Example: final infographic
└── docs/
    └── WORKBUDDY_SKILL.md           # WorkBuddy integration notes
```

## Contributing

PRs are welcome! Some ideas:

- [ ] LLM-powered content analysis (replace heuristic Step 2 for better accuracy)
- [ ] Streamlit / Gradio web UI
- [ ] Support for multi-page infographic layouts
- [ ] Additional image providers (Stability AI, Replicate, etc.)
- [ ] CLI options for style customization (colors, layout, font style)

## License

MIT -- see [LICENSE](LICENSE).

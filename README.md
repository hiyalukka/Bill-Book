# 🧾 BillBook: bills in, books out

**Turn a photo or PDF of an Indian GST bill into a clean, verified row ready for Excel or Tally. An open-weight AI model reads the bill inside your browser, so the bill itself is never uploaded anywhere.**

[**Live demo →**](https://hiyalukka.github.io/Bill-Book/) · 



Built for the DEV × Hacktoberfest Week 3 challenge, *Side Hustle*: open-source AI that someone would actually pay for.

---

## The problem

A small accountant or shopkeeper receives bills as WhatsApp photos, crumpled paper, and inconsistent PDFs. Someone then retypes vendor, GSTIN, date, taxable amount, CGST/SGST/IGST, and total into a sheet by hand. It is slow, boring, and one wrong digit can cause a GST mismatch.

Sending client financial documents to a cloud AI is often not an option, and per-bill API fees eat into a small tool's margin.

## What BillBook does

1. **Cleans the image** (grayscale, contrast, sensible resizing).
2. **Reads the text** with Tesseract OCR (English, Hindi, or Gujarati data).
3. **Extracts the fields** with an open-weight model (Gemma or Qwen2.5) running on your own GPU through WebGPU.
4. **Verifies every row with plain code**, not AI.
5. **Shows an editable table** with OK or REVIEW per row, and exports a CSV.

> **The model extracts, the code verifies.** A small model can misread or invent numbers. A wrong total that looks right is the worst outcome for an accountant, so BillBook never trusts the model silently.

## The checks (no AI involved)

| Check | What it catches |
|---|---|
| GSTIN format + state code | Wrong length, bad structure, impossible state |
| GSTIN checksum | A single misread character (the 15th character is computed from the first 14) |
| Taxable + taxes = total | Misread or invented amounts |
| CGST = SGST, and not IGST alongside them | Invalid tax combinations |
| **Grounding check** | Every amount the model returns must literally appear in the scanned text, so the model cannot invent a number |
| Date and vendor sanity | Impossible dates, empty fields |

Rows that fail any check are marked **REVIEW** with the reasons. Edit a cell and the checks re-run.

## Why open models matter here

- **Privacy.** The model runs on the user's device. Bills are never sent to an AI provider.
- **Cost.** There are no per-bill fees. Processing 10 bills or 10,000 costs the same.
- **Swappable.** Pick any compatible open model from the dropdown. No lock-in to one vendor's pricing or API.
- **Tunable.** Open weights can be fine-tuned later on a specific kind of bill, which a closed API cannot offer.

> **Honest note on "private":** your *bills* are never uploaded. The page itself still downloads its libraries and the model weights from the web the first time, after which the browser caches them.

## Run it

**Online:** open the live demo link : https://hiyalukka.github.io/Bill-Book/ , click **Load model** (first load takes a few minutes), then drop in bills.

**Locally:**
```bash
git clone https://github.com/YOUR-USERNAME/billbook.git
cd billbook
python -m http.server 8000
# open http://localhost:8000/ in Chrome or Edge
```

Requires a browser with **WebGPU** (recent Chrome or Edge). Opening the file by double-click will not work; serve it as above.

## Test it without real client data

Open `demo-bills.html`. It generates synthetic GST invoices with **valid GSTINs**, some clean and some made to look like tilted, blurry phone photos, plus an `answer_key.csv`.

Run the bills through BillBook, download its CSV, and use the scorer on the same page. It reports:

- rows fully correct before any checks
- wrong rows the checks caught (marked REVIEW)
- **wrong rows that slipped through as OK** (the number that matters most)
- per-field accuracy

### Results

> 

| Model | Bills | Fully correct | Wrong rows caught | Slipped through as OK |
|---|---|---|---|---|
| Gemma 2B | 5 rows(3 unique bills) | 3 of 5(2 of 3 unique) | 2 of 2 | 0 |
| Qwen2.5 1.5B | 3 | 0 of 3 | 3 of 3 | 0 |

## Limitations

- Test bills are **synthetic** and cleaner than real ones. A good score here does not prove real-world accuracy.
- Small models make mistakes. OCR errors upstream cannot be recovered by the model.
- The grounding check confirms a number exists *somewhere* in the text, not that it is the right one. A misread value can still be matched by a lucky copy from another line.
- Hindi and Gujarati OCR is available but not yet tested.
- PDFs use page 1 only. Line items and HSN codes are not extracted.
- Speed depends on your GPU. Weak or integrated GPUs can take a while per bill.

## Roadmap

- [ ] Tally-ready export
- [ ] Optional local Ollama backend for faster inference
- [ ] Line items and HSN extraction
- [ ] Fine-tuning on real, consented Indian bills
- [ ] Measured Hindi and Gujarati accuracy

## Built with

[WebLLM](https://github.com/mlc-ai/web-llm) (in-browser inference) · [Gemma](https://ai.google.dev/gemma) and [Qwen2.5](https://github.com/QwenLM/Qwen2.5) (open-weight models; see each model's license) · [Tesseract.js](https://github.com/naptha/tesseract.js) · [pdf.js](https://github.com/mozilla/pdf.js) · [JSZip](https://github.com/Stuk/jszip)

## License

MIT. See `LICENSE`. Model weights are licensed separately by their publishers.

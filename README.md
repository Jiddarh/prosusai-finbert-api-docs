
ProsusAI FinBERT API

The ProsusAI FinBERT API is a financial sentiment analysis API powered by FinBERT — a BERT-based model fine-tuned on financial text. It analyzes financial statements, news headlines, and market commentary, and returns structured sentiment predictions with confidence scores.

This document is intended for developers integrating financial sentiment analysis into applications such as trading dashboards, investment tools, and financial news platforms.

---

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Quick Start](#quick-start)
- [API Endpoint](#api-endpoint)
- [Request Body](#request-body)
- [Response Body](#response-body)
- [Example Request & Response](#example-request--response)
- [Error Reference](#error-reference)
- [Troubleshooting](#troubleshooting)
- [Understanding the Model](#understanding-the-model)
- [Final Notes](#final-notes)

---

## Overview

FinBERT is trained specifically on financial language, making it significantly more accurate on financial text than general-purpose sentiment models. It classifies text into three sentiment categories: **positive**, **negative**, and **neutral**.

### Typical Use Cases

- Analyzing earnings call transcripts
- Classifying financial news headlines by sentiment
- Monitoring market commentary in real time
- Flagging negative sentiment in investor reports

---

## Authentication

All requests require a Hugging Face API token.

### Getting Your API Token

1. Sign up or log in at [huggingface.co](https://huggingface.co)
2. Go to **Settings → Access Tokens**
3. Click **New token**, select **Read** scope
4. Copy your token — it starts with `hf_`

### Authorization Header Format

```
Authorization: Bearer YOUR_API_TOKEN
```

> ⚠️ Never expose your API token in client-side code or commit it to version control. Always use environment variables.

---

## Quick Start

Make your first API call in minutes.

### cURL

```bash
curl -X POST https://router.huggingface.co/hf-inference/models/ProsusAI/finbert \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": "The stock market performed strongly this quarter."
  }'
```

### Python

```python
import requests
import os

API_TOKEN = os.environ["HF_TOKEN"]
API_URL = "https://router.huggingface.co/hf-inference/models/ProsusAI/finbert"

headers = {
    "Authorization": f"Bearer {API_TOKEN}",
    "Content-Type": "application/json"
}

def analyze_sentiment(text: str) -> dict:
    payload = {"inputs": text}
    response = requests.post(API_URL, headers=headers, json=payload)
    response.raise_for_status()
    return response.json()

result = analyze_sentiment("The stock market performed strongly this quarter.")
print(result)
# Output: [[{"label": "positive", "score": 0.9532}, {"label": "negative", "score": 0.0253}, {"label": "neutral", "score": 0.0232}]]
```

🎉 You're now successfully using the ProsusAI FinBERT API!

---

## API Endpoint

### `POST /hf-inference/models/ProsusAI/finbert`

Analyzes financial text and returns sentiment predictions with confidence scores.

**Base URL:**
```
https://router.huggingface.co
```

**Full endpoint:**
```
https://router.huggingface.co/hf-inference/models/ProsusAI/finbert
```

---

## Request Body

| Field | Type | Required | Description |
|---|---|---|---|
| `inputs` | string or array | Yes | The financial text to analyze (max 512 tokens) |

### Single Input

```json
{
  "inputs": "The stock market performed strongly this quarter."
}
```

### Batch Input

You can pass multiple texts in a single request:

```json
{
  "inputs": [
    "The stock market performed strongly this quarter.",
    "Inflation concerns are weighing heavily on investor confidence."
  ]
}
```

> **Note:** Batch inputs return one result array per input text.

---

## Response Body

The API returns an array of sentiment predictions, each with a label and confidence score.

| Field | Type | Description |
|---|---|---|
| `label` | string | Sentiment label: `positive`, `negative`, or `neutral` |
| `score` | number | Confidence score between 0 and 1 |

> All three labels are always returned, ranked by confidence score from highest to lowest.

---

## Example Request & Response

### Request

```json
{
  "inputs": "The stock market performed strongly this quarter."
}
```

### Response

```json
[
  [
    { "label": "positive", "score": 0.9532 },
    { "label": "negative", "score": 0.0253 },
    { "label": "neutral", "score": 0.0232 }
  ]
]
```

### Interpreting the Response

The first label in the array is always the predicted sentiment — the one with the highest confidence score. In this example:

- The model predicts **positive** sentiment with **95.3%** confidence
- This is a strong, reliable prediction
- Scores below `0.60` should be treated with caution and may warrant human review

---

## Error Reference

| HTTP Status | Error | Cause | Resolution |
|---|---|---|---|
| `400` | Bad Request | Malformed request body | Check your JSON syntax and required fields |
| `401` | Unauthorized | Missing or invalid API token | Verify your `Authorization` header format |
| `403` | Forbidden | Token lacks required permissions | Generate a new token with correct scope |
| `404` | Not Found | Incorrect endpoint URL | Double-check the full endpoint URL |
| `429` | Too Many Requests | Rate limit exceeded | Slow down requests or upgrade your plan |
| `503` | Model Loading | Model is initializing (cold start) | Retry after `estimated_time` seconds |

### Error Response Format

```json
{
  "error": "Authorization error: invalid credentials"
}
```

---

## Troubleshooting

| Issue | Possible Cause | What to Do |
|---|---|---|
| `401 Unauthorized` | Missing `Bearer` prefix | Ensure header is `Bearer hf_xxxx` not just `hf_xxxx` |
| `403 Forbidden` | Insufficient token permissions | Regenerate token with Read scope |
| `404 Not Found` | Wrong URL | Copy the full endpoint URL exactly as shown |
| Low confidence scores | Text is ambiguous or non-financial | Use clear, specific financial language |
| All scores close to 0.33 | Model is uncertain | Provide more context in the input text |

---

## Understanding the Model

### About FinBERT

FinBERT is built on Google's BERT architecture and fine-tuned on a large corpus of financial text including earnings reports, analyst notes, and financial news articles. This makes it significantly more accurate on financial language than general-purpose sentiment models.

### Confidence Scores

Each prediction returns a confidence score between `0` and `1`. Use these thresholds as a guide:

| Score Range | Interpretation | Recommended Action |
|---|---|---|
| `0.85 – 1.00` | Very high confidence | Safe to act on programmatically |
| `0.60 – 0.84` | Moderate confidence | Use with caution |
| `0.33 – 0.59` | Low confidence | Flag for human review |

### Model Limitations

- Performs best on formal financial English
- May struggle with slang, abbreviations, or social media language
- Sarcasm and irony are not reliably detected
- Very short inputs (under 5 words) often return low confidence scores

> **Pro tip:** For critical financial decisions, always combine model predictions with human judgment. Never rely solely on a single confidence score.

---

## Final Notes

- Always store your API token in environment variables — never hardcode it
- Handle `503` errors with retry logic and exponential backoff
- Monitor confidence scores and set thresholds appropriate for your use case
- For production workloads requiring dedicated resources, consider [Hugging Face Inference Endpoints](https://huggingface.co/docs/inference-endpoints)
- View the model card at [huggingface.co/ProsusAI/finbert](https://huggingface.co/ProsusAI/finbert)

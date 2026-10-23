# Phishing Link Analyzer

Paste in a suspicious message, and this tool pulls out every link and checks each one against VirusTotal. You get a short report with a verdict per URL and an overall risk level for the whole message.

It uses Gemini (through LangChain) to find the links, and VirusTotal to judge them. The LLM never decides what's safe or malicious, it only extracts. The security call comes from VirusTotal's data.

## How it works

1. **Extract**: Gemini 2.5 Flash reads the message and returns structured output: a short title, a list of URLs, and a list of direct image URLs (validated with Pydantic).
2. **Check**: each URL is encoded the way the VirusTotal v3 API expects and looked up for an existing report.
3. **Score**: each URL gets a risk level based on its detection counts:
   - `HIGH`: 3 or more engines flag it as malicious
   - `MEDIUM`: fewer than 3 malicious detections, but at least one malicious or suspicious detection
   - `LOW`: no malicious or suspicious detections
   - `UNKNOWN`: no report found, or the lookup failed
4. **Report**: prints a summary with per-URL stats and an overall risk, which is the worst level found across all the URLs.

## Setup

The notebook is written for Google Colab and reads keys from Colab Secrets.

You need two API keys:

- **Google AI Studio** key for Gemini: https://aistudio.google.com/apikey
- **VirusTotal** key (a free account works): https://www.virustotal.com/gui/my-apikey

In Colab, open the key icon in the left sidebar and add them as:

| Secret name      | Value                |
| ---------------- | -------------------- |
| `GOOGLE_API_KEY` | your Gemini key      |
| `VT_API_KEY`     | your VirusTotal key  |

Then turn on notebook access for both.

Running it locally instead? Swap the `google.colab.userdata` lines for environment variables:

```python
import os
GOOGLE_API_KEY = os.environ["GOOGLE_API_KEY"]
VT_API_KEY = os.environ["VT_API_KEY"]
```

## Usage

1. Open `phishing_link_analyzer.ipynb` in Colab.
2. Replace `user_message` with the message you want to check.
3. Run all cells (Runtime > Run all).

## Example output

```
============================================================
              PHISHING ANALYSIS REPORT
============================================================

Message Title: Important Documents and Photos

URLs Detected:    6
Images Detected:  2

1. http://secure-update-account.net

   VirusTotal Report: FOUND
   Malicious:   1
   Suspicious:  0
   Harmless:    55
   ...
```

The sample message in the notebook has a mix of lookalike domains (like `appleid.apple.com.verify-security.info`), a couple of image links, and a legit `google.com` link, so you can see how each one gets handled.

## Limitations

- **Only looks up existing reports.** It doesn't submit new URLs for scanning, so a brand new phishing link with no VirusTotal history shows up as `UNKNOWN`, not safe.
- **Image URLs aren't scanned.** They're extracted and counted but not sent to VirusTotal.
- **"LOW" doesn't mean safe.** It just means no engine has flagged it yet. New phishing pages often go undetected for a while.
- **Free VirusTotal keys are rate limited** (around 4 requests per minute), so long messages with lots of links may hit 429 errors.
- **The LLM can miss links**, especially obfuscated ones (`hxxp://`, spaces in the URL, etc.).


## Ideas for later

- Submit unknown URLs to VirusTotal for a fresh scan
- Add rule-based checks for lookalike domains and suspicious subdomain patterns
- Handle defanged URLs
- Retry/backoff for rate limits
- Wrap it in a small CLI or web UI

## Built with

- [LangChain](https://www.langchain.com/) + `langchain-google-genai`
- Gemini 2.5 Flash
- [VirusTotal API v3](https://docs.virustotal.com/reference/overview)
- Pydantic, Requests


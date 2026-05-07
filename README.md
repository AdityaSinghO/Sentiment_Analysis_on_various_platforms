# E-Commerce & Social Media Sentiment Analysis

A multi-source sentiment analysis project that extracts and analyzes customer reviews and comments from **Flipkart**, **Amazon**, and **YouTube**, classifying them as Positive, Negative, or Neutral.

---

> ⚠️ **Legal Disclaimer**
>
> The web scraping components of this project (Amazon and Flipkart scrapers) are intended **for educational and research purposes only**.
>
> Scraping Amazon and Flipkart may violate their **Terms of Service**. Both platforms explicitly prohibit automated access to their content without prior written permission. The scrapers in this project also use techniques to disguise automated traffic (spoofed user-agents, disabled automation flags), which may carry additional legal risk.
>
> **Do not use the scraping modules for commercial purposes, large-scale data collection, or in any way that could harm the platforms or their users.** The author takes no responsibility for any misuse of this code. Always review a website's `robots.txt` and Terms of Service before scraping.

---

## Project Structure

| Notebook | Description |
|---|---|
| `online_commerce_sentiment_analysis.ipynb` | Sentiment analysis on a static Flipkart CSV dataset |
| `sentiment_analysis_using_links.ipynb` | **Original** live scraper — paste an Amazon or Flipkart URL and get sentiment analysis |
| `sentiment_analysis_optimized.ipynb` | **Optimized rewrite** of the live scraper — see improvements below |
| `sentiment_analysis.ipynb` | YouTube comment sentiment analysis via the YouTube Data API |

---

## Notebook Breakdown

### 1. `online_commerce_sentiment_analysis.ipynb` — Static Analysis
Reads from a local `flipkart_product.csv` file and performs full sentiment analysis on it. Also includes a built-in 20-review sample dataset (Smartphone, Laptop, Headphones, Smartwatch) so it runs without needing any file at all. Good starting point for understanding the sentiment pipeline.

**Sentiment engine:** TextBlob

---

### 2. `sentiment_analysis_using_links.ipynb` — Original Live Scraper
The original version of the live scraper. Accepts an Amazon or Flipkart product URL, scrapes reviews across multiple pages using Selenium + BeautifulSoup, and runs sentiment analysis on the results.

**Sentiment engine:** TextBlob  
**Known issues:** See [Known Limitations](#known-limitations)

---

### 3. `sentiment_analysis_optimized.ipynb` — Optimized Live Scraper ⭐
A complete rewrite of Notebook 2 with the following improvements:

| Area | Original | Optimized |
|---|---|---|
| Sentiment engine | TextBlob (movie reviews) | **VADER** (built for informal/short text) |
| Storage | In-memory dict → DataFrame at end | **SQLite** — saves per review, resume after crash |
| Deduplication | None — duplicates possible | **MD5 fingerprinting** per review |
| Error handling | Bare `except: continue` everywhere | **Structured logging** to `scraper.log` |
| `clean_text` | Stripped numbers (`5 star` → `star`) | **Numbers preserved** |
| Blocked by CAPTCHA | Silent failure | **Exponential backoff** + `headless=False` option |
| Chrome setup | Duplicated across scrapers | **Single `build_driver()`** with rotating user-agents |
| Flipkart selectors | Single class name (breaks on UI update) | **Multiple fallback class fragments** |
| Entry point | Platform-specific function calls | **One cell** — `PRODUCT_URL`, `MAX_PAGES`, `HEADLESS` |
| VADER scores | N/A | `compound`, `pos`, `neu`, `neg` all saved to CSV |

**Sentiment engine:** VADER  
**Recommended:** Use this notebook over the original for all new runs.

---

### 4. `sentiment_analysis.ipynb` — YouTube Comments
Fetches up to 100 top-level comments from any YouTube video via the YouTube Data API v3 and runs sentiment analysis on them. Produces a bar chart, pie chart, polarity histogram, and boxplot.

**Sentiment engine:** TextBlob  
**Requires:** YouTube Data API v3 key (see setup below)

---

## Setup

### Install Dependencies

**For Notebooks 1, 2, and 3 (optimized scraper):**
```bash
pip install pandas numpy matplotlib seaborn vaderSentiment wordcloud \
            selenium webdriver-manager beautifulsoup4 lxml
```

**For Notebook 4 (YouTube):**
```bash
pip install pandas matplotlib seaborn textblob google-api-python-client nltk
python -m textblob.download_corpora
```

```python
import nltk
nltk.download('punkt')
nltk.download('averaged_perceptron_tagger')
```

**Chrome + ChromeDriver** (for scraping notebooks):
- Install [Google Chrome](https://www.google.com/chrome/)
- `webdriver-manager` installs the correct ChromeDriver automatically — no manual setup needed

---

## Usage

### Notebook 1 — Static CSV Analysis
1. Place your `flipkart_product.csv` at the path in cell 3, or skip that cell to use the built-in sample data
2. Run all cells

### Notebook 3 — Optimized Live Scraper (recommended)

> ⚠️ See the legal disclaimer above before running.

1. Open `sentiment_analysis_optimized.ipynb`
2. In the final cell, set your config:
   ```python
   PRODUCT_URL = "https://www.amazon.in/dp/YOUR_PRODUCT_ID"
   MAX_PAGES   = 5
   HEADLESS    = True   # set False if you get blocked by a CAPTCHA
   ```
3. Run all cells — Chrome opens, scrapes, and closes automatically
4. Results are saved to `reviews.db` (SQLite) and exported to `sentiment_results.csv`

**If you get blocked (CAPTCHA):** Re-run with `HEADLESS = False`. A visible browser window will open — solve the CAPTCHA manually and the scraper will continue from where it left off.

**Supported URL formats:**
```
Amazon product page  : https://www.amazon.in/dp/B0XXXXXXXX
Amazon reviews page  : https://www.amazon.in/product-reviews/B0XXXXXXXX
Flipkart product page: https://www.flipkart.com/product-name/p/itemXXXXXXXX
```

### Notebook 2 — Original Live Scraper
Same usage as Notebook 3, but uses the older TextBlob-based pipeline without SQLite persistence.

```python
product_url = "https://www.amazon.in/dp/YOUR_PRODUCT_ID"
max_pages = 5
```

If the auto-scraper fails, use the manual fallback:
```python
scrape_amazon_reviews_manual(
    reviews_page_url="https://www.amazon.in/product-reviews/ASIN/...",
    max_pages=5
)
```

### Notebook 4 — YouTube Comments

> ⚠️ **Never commit your API key to a public repository.** Use an environment variable or `.env` file.

1. Get a YouTube Data API v3 key from [Google Cloud Console](https://console.cloud.google.com/)
2. In cell 3, set your key:
   ```python
   YOUTUBE_API_KEY = "YOUR_API_KEY_HERE"
   ```
3. In the final cell, set your video URL:
   ```python
   video_url = "https://www.youtube.com/watch?v=YOUR_VIDEO_ID"
   ```
4. Run all cells

---

## Output

| Notebook | CSV output | Other output |
|---|---|---|
| `online_commerce_sentiment_analysis.ipynb` | `sentiment_analysis_results.csv` | 4 charts + 3 word clouds |
| `sentiment_analysis_using_links.ipynb` | `sentiment_analysis_results.csv` | 4 charts + 3 word clouds |
| `sentiment_analysis_optimized.ipynb` | `sentiment_results.csv` | 4 charts + 3 word clouds + `reviews.db` + `scraper.log` |
| `sentiment_analysis.ipynb` | `youtube_sentiment_analysis.csv` | 4 charts |

---

## Requirements

| Package | Used in | Purpose |
|---|---|---|
| `pandas`, `numpy` | All | Data handling |
| `vaderSentiment` | Notebooks 1, 3 | Sentiment analysis (product reviews) |
| `textblob` | Notebooks 2, 4 | Sentiment analysis (original/YouTube) |
| `matplotlib`, `seaborn` | All | Visualizations |
| `wordcloud` | Notebooks 1–3 | Word cloud generation |
| `selenium`, `webdriver-manager` | Notebooks 2, 3 | Browser automation |
| `beautifulsoup4`, `lxml` | Notebooks 2, 3 | HTML parsing |
| `google-api-python-client` | Notebook 4 | YouTube Data API |
| `nltk` | Notebook 4 | NLP tokenization |

---

## Known Limitations

- **Flipkart selectors** use obfuscated class names that change on every UI deploy. Notebook 3 uses multiple fallback fragments to reduce breakage, but may still need updating after a major Flipkart redesign.
- **Amazon CAPTCHAs** will block scraping depending on your IP and request frequency. Use `HEADLESS=False` and solve manually, or add a proxy/scraper API for reliable large-scale runs.
- **VADER** (Notebook 3) handles informal text well but still struggles with sarcasm and highly technical reviews. For production-grade accuracy, consider a fine-tuned HuggingFace model like `nlptown/bert-base-multilingual-uncased-sentiment`.
- **TextBlob** (Notebooks 2 & 4) is less accurate on product/comment text — it was trained on movie reviews.
- The **YouTube notebook** has an exposed API key in the original file. Revoke it and replace with an environment variable before pushing to any repository.

# 链家成交数据爬虫（以广州为例）
# Lianjia Transaction Data Scraper (Guangzhou Example)

A Selenium-based tool for collecting **sold transaction data** from Lianjia (链家), implemented as a Jupyter Notebook. It supports batch scraping by district and sub-area, with checkpoint resume, CAPTCHA handling, and keep-awake features—designed for long-running, large-scale data collection.

> **Languages:** [中文](README.md) | English

**This repo uses Guangzhou as an example** (`gz.lianjia.com`). Lianjia city sites share the same page structure and scraping logic—swap the domain and starting URLs to adapt it for Beijing, Shanghai, Shenzhen, and other cities.

## Features

- **Sub-area list collection**: Automatically traverses administrative districts and their sub-areas to build a URL list (Guangzhou in this example)
- **Transaction data parsing**: Extracts community name, layout, area, orientation, decoration, deal price, unit price, listing price, deal date, time on market, floor, build year, five-year ownership, subway proximity, and more
- **Checkpoint resume**: Saves progress via `checkpoint.json` and `partial_data.json` so scraping can continue after interruption
- **CAPTCHA handling**: Pauses when a verification page is detected and waits for manual completion before resuming
- **Keep-awake**: A background thread moves the mouse periodically to prevent the computer from sleeping during long runs
- **Excel export**: Outputs data as `.xlsx` files

## Project Structure

```
.
├── 广州.ipynb                    # Main program (Jupyter Notebook)
├── checkpoint.json               # Scraping progress checkpoint (auto-generated)
├── partial_data.json             # Partial scraped data (auto-generated)
├── 广州链家板块列表.xlsx           # Sub-area list (generate via step 3)
├── 临时成交数据.xlsx               # Temporary Excel backup at each checkpoint
└── 广州链家成交数据.xlsx           # Final transaction data output
```

## Requirements

- Python 3.8+
- Google Chrome browser
- ChromeDriver (must match your Chrome version; Selenium 4.6+ usually manages this automatically)

### Python Dependencies

```bash
pip install pandas selenium pyautogui openpyxl
```

| Package | Purpose |
|---------|---------|
| `pandas` | Data processing and Excel I/O |
| `selenium` | Browser automation |
| `pyautogui` | Keep-awake (subtle mouse movement) |
| `openpyxl` | Excel write engine |

## Usage

### 1. Clone the Repository

```bash
git clone https://github.com/MinjieDING/Lianjia.git
cd Lianjia
```

### 2. Start Jupyter

```bash
jupyter notebook
```

Open `广州.ipynb` and run the cells in order.

> **Other cities**: Replace the Lianjia domain in the notebook (e.g. `gz.lianjia.com`) and the sub-area starting URL with your target city—e.g. Shanghai `sh.lianjia.com`, Beijing `bj.lianjia.com`.

### 3. Collect Sub-area List (First Run)

In the notebook, uncomment and run the cells under **「获取所有板块 URL」** (Get all sub-area URLs). This generates `广州链家板块列表.xlsx` with district and sub-area names and links.

You can skip this step if you already have a sub-area list file.

### 4. Run the Main Scraper

Run the **「执行主爬」** (Execute main crawl) cell. The program will:

1. Read `广州链家板块列表.xlsx`
2. Resume from a checkpoint if one exists
3. Scrape transaction data page by page, sub-area by sub-area
4. Save the final output to `广州链家成交数据.xlsx`

### 5. Close the Browser

When scraping finishes or you need to stop, run the last cell to close the browser.

## Checkpoint Resume

During scraping, the program periodically writes:

| File | Description |
|------|-------------|
| `checkpoint.json` | Current sub-area index, page number, total records, save time |
| `partial_data.json` | Raw scraped data in JSON format |
| `临时成交数据.xlsx` | Excel backup synced with the checkpoint |

After an interruption, re-run the **「执行主爬」** cell to resume automatically. When all sub-areas are complete, `checkpoint.json` is removed.

**Example checkpoint:**

```json
{
  "district_index": 151,
  "page_index": 10,
  "total_houses": 71020,
  "save_time": "2025-11-05 01:45:31"
}
```

## Output Fields

| Field | Description |
|-------|-------------|
| `district` | Sub-area name |
| `community` | Community / compound name |
| `layout` | Unit layout (e.g. 3BR/2BA) |
| `area` | Floor area |
| `orientation` | Facing direction |
| `decoration` | Decoration status |
| `total_price` | Total deal price |
| `unit_price` | Price per square meter |
| `list_price` | Original listing price |
| `deal_date` | Transaction date |
| `deal_cycle_days` | Days on market until sale |
| `floor_info` | Floor information |
| `built_year` | Year built |
| `position` | Location details |
| `five_years` | Five-year ownership exemption eligible |
| `near_subway` | Near subway |
| `agent` | Agent information |
| `district_url` | Sub-area page URL |

## Account Limits & Cost

In practice, **each account is typically blocked after scraping roughly 15,000 records**. For large-scale collection, the main cost is therefore **registering new phone numbers** to rotate accounts—not compute or bandwidth.

One feasible option: buy prepaid SIM cards at **7-Eleven (711) stores in Hong Kong** for about **HKD 30 each**.

## Important Notes

1. **Anti-bot & verification**: Lianjia uses anti-scraping measures. Frequent requests may trigger CAPTCHA. Complete verification manually in the browser; the script will detect completion and continue.
2. **Account bans**: A single account can only collect a limited amount (~15,000 records). Prepare multiple phone numbers to register new accounts and continue scraping.
3. **Request rate**: Random delays (~1.5–3.5 seconds) are built in. Do not shorten them aggressively to avoid IP bans.
4. **Compliance**: This project is for learning and research only. Follow [Lianjia/Beike](https://gz.lianjia.com) terms of service and robots rules. Do not use for commercial purposes or large-scale redistribution of data.
5. **Data freshness**: Transaction data reflects what the website shows at scrape time. Historical results may change as the site updates.
6. **Sensitive info**: The notebook includes third-party SMS platform links at the end, unrelated to the scraper. Handle accounts and privacy with care.

## FAQ

**Q: ChromeDriver errors?**  
A: Ensure Chrome is installed and the driver version matches, or upgrade Selenium to the latest version for automatic driver management.

**Q: Notebook crashed mid-scrape?**  
A: Reopen the notebook, re-run the setup cells (imports, browser init, etc.), then run **「执行主爬」** again—it will resume from the checkpoint.

**Q: A sub-area has no data?**  
A: The script skips sub-areas when it detects “没有找到房源” (no listings found). This is expected.

## License

This project is for personal learning and sharing only. Use at your own risk and responsibility.

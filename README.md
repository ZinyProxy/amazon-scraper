# Amazon Search & Product Scraper (Python/Scrapy) 

A complete, ready-to-use Python web scraper built with Scrapy to extract Amazon search results and detailed product metadata at scale. 

Most open-source Amazon scrapers fail immediately because Amazon strictly blocks datacenter IPs and local connections. To solve this, this scraper features a custom middleware with a **built-in residential proxy pool** powered by [Ziny.io](https://ziny.io), ensuring your requests bypass CAPTCHAs and anti-bot systems right out of the box.

## Key Features

* **Search Results Crawler (`amazon_search`)**: Scrapes Amazon search pages for specific keywords and extracts Titles, Prices, Ratings, ASINs, and URLs.
* **Product Detail Crawler (`amazon_product`)**: Navigates directly to product pages to extract comprehensive data including Descriptions, Feature Bullets, Review Counts, and Image URLs.
* **Built-in Proxy Rotation**: Pre-configured with a free 2GB residential proxy pool so you can test and run the scraper immediately without buying 3rd-party APIs.
* **Automatic Pagination**: Crawls through multiple pages of search results seamlessly.
* **Structured CSV Export**: Automatically cleans and exports scraped data into organized `.csv` files.

## Prerequisites and Installation

You will need Python 3.7+ installed on your system.

```bash
git clone [https://github.com/YOUR_GITHUB_USERNAME/amazon-scrapy-scraper.git](https://github.com/YOUR_GITHUB_USERNAME/amazon-scrapy-scraper.git)
cd amazon-scrapy-scraper

# Create and activate a virtual environment
python -m venv .venv

# Windows
.venv\Scripts\Activate.ps1
# macOS/Linux
source .venv/bin/activate

# Install the required Python packages
pip install -r requirements.txt               
```

---

## Quickstart

Because the proxy rotation is natively handled in the middleware, no complex configuration is required.

```bash
cd amazon_scraper

# Run the search spider to scrape keyword results
scrapy crawl amazon_search

# Run the product spider to scrape individual item details
scrapy crawl amazon_product
```
By default, the spiders save the extracted data into the amazon_scraper/data/ directory with an automatic timestamp.

---

## How It Works

Open amazon_scraper/spiders/amazon_search.py (or amazon_product.py) and modify the keyword_list variable to target your desired niche:

```python
def start_requests(self):
    keyword_list = ['gaming laptops', 'mechanical keyboards']
    for keyword in keyword_list:
        amazon_search_url = f'[https://www.amazon.com/s?k=](https://www.amazon.com/s?k=){keyword}&page=1'
        yield scrapy.Request(url=amazon_search_url, callback=self.parse_search_results, meta={'keyword': keyword, 'page': 1})
```
##Scaling for Production Use

This tool includes a 200MB free trial pool of residential proxies to get you started. If you plan to scrape Amazon at scale, you will eventually hit the bandwidth limit and experience 403 Forbidden errors.
To scale up, simply replace the community credentials with your own private keys:
1.  Create an account at Ziny.io (residential proxies are $0.40/GB).
2.  Open amazon_scraper/middlewares.py.
3.  Update the proxy_user and proxy_pass variables with your specific Ziny credentials.
---

##Repository Architecture

```bash
amazon-scrapy-scraper/
├── amazon_scraper/
│   ├── spiders/
│   │   ├── amazon_search.py      # Search page logic
│   │   └── amazon_product.py     # Product detail logic
│   ├── middlewares.py            # Custom Proxy Routing logic
│   ├── items.py                 
│   ├── pipelines.py                       
│   └── settings.py               # Scrapy configuration      
├── scrapy.cfg                    
├── requirements.txt              
├── .gitignore                    
├── LICENSE                      
└── README.md
```

## Example Data Output

### Search Results (CSV)
```csv
keyword,asin,url,ad,title,price,rating,rating_count,thumbnail_url
ipad,B09G9FPHY6,[https://www.amazon.com/dp/B09G9FPHY6,False,iPad](https://www.amazon.com/dp/B09G9FPHY6,False,iPad) (10th generation),$449.00,4.7 out of 5 stars,12,345,[https://m.media-amazon.com/images/I/71](https://m.media-amazon.com/images/I/71)...
```

## Config & Scaling

### Customizing Keywords
Edit the spider files to change search terms:

```python
# In amazon_scraper/spiders/amazon_search.py or amazon_product.py
def start_requests(self):
    keyword_list = ['your', 'keywords', 'here']
    for keyword in keyword_list:
        amazon_search_url = f'[https://www.amazon.com/s?k=](https://www.amazon.com/s?k=){keyword}&page=1'
        yield scrapy.Request(url=amazon_search_url, callback=self.parse_search_results, meta={'keyword': keyword, 'page': 1})
```

### Output Configuration
The spiders automatically save to CSV files in the `amazon_scraper/data/` directory:

```python
custom_settings = {
    'FEEDS': { 'data/%(name)s_%(time)s.csv': { 'format': 'csv',}}
}
```

### Geolocation
If you need to scrape Amazon UK or Amazon DE, you can target specific country IPs by modifying your Ziny.io username in `middlewares.py`:. Add the country code to your proxy username format based on your Ziny dashboard instructions (e.g., appending -country-uk to the user).

Concurrency & Rate Limiting
Because you are using residential proxies, you can scrape significantly faster without getting blocked. Adjust your concurrency in `amazon_scraper/settings.py`:

```python
SCRAPEOPS_PROXY_SETTINGS = {'country': 'us'}  # us, uk, ca, de, etc.
```

### Concurrency & Rate Limiting
Because you are using residential proxies, you can scrape significantly faster without getting blocked. Adjust your concurrency in `amazon_scraper/settings.py`:

```python
CONCURRENT_REQUESTS = 10  # Optimized for Ziny residential proxies
DOWNLOAD_DELAY = 0.5      # Add a slight delay to be safe
```

---

## Legal

⚠️ Scraping Amazon may violate their TOS.  
This repo is for educational purposes only. Use responsibly.

---

## Contributing

Want to add:
- More fields (variants, seller ratings)?
- Price history tracking?
- Anti-bot layer (Playwright, CAPTCHA solver)?
- Output to RAG pipelines?

→ PRs welcome!

---

## 🔗 Related Templates

In Pipeline

---

## Built With

- Scrapy (Python)
- [Ziny Proxy](https://ziny.io)
---

## Dependencies

- **scrapy:** Web scraping framework
- **base64:** For proxy authentication encoding (Built-in Python library)

## Troubleshooting

### No Products Found / 403 Errors
1.  Bandwidth Exceeded: The 200MB community proxy pool may be exhausted. You will need to replace the trial credentials in amazon_scraper/middlewares.py with your own Ziny.io keys.
2.  Debug Mode: Run with debug logging to see what is failing: scrapy crawl amazon_search -L DEBUG
3.  Invalid Selectors: Amazon occasionally updates their HTML structure. You may need to update the CSS selectors in the spider files.

### Virtual Environment Issues
```bash
# Ensure virtual environment is activated
.venv\Scripts\Activate.ps1  # Windows
# source .venv/bin/activate  # macOS/Linux

# Reinstall dependencies if needed
pip install -r requirements.txt
```

### Permission Errors (Windows PowerShell)
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---


## Migration Benefits

This project evolved from a simple single-file spider to a professional Scrapy structure:

- **90% Code Reduction**: From 600+ lines to ~150 lines total
- **Clean Architecture**: Each spider has a single, focused purpose
- **Easy Maintenance**: Simple, readable code structure
- **Professional Quality**: Industry-standard Scrapy project layout
- **Automatic Data Export**: CSV files with timestamps

---

## 🚀 Next Steps

1.  Get Unlimited Proxies: Sign up at Ziny.io to get ultra-cheap residential proxies ($0.40/GB).
2.  Test Spiders: Run each spider individually to see the proxy rotation in action.
3.  Customize Data: Modify extraction logic as needed to pull specific metrics.
4.  Scale Up: Increase the CONCURRENT_REQUESTS in your Scrapy settings for maximum speed.
5.  Add Features: Implement additional spiders or data processing pipelines.

The implementation provides a solid foundation for professional Amazon scraping with clean, maintainable code and robust infrastructure powered by Ziny Proxy

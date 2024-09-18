# WebScraping
Kelvins web scraping 
Certainly! Below is the full code for the Scrapy spider that you can use to crawl any website, extract all text data from each page, and save each page's content into separate folders that mirror the website's structure. The code is commented to make it easy for you to adjust the domain name and customize output folders.

---

## **1. Project Setup**

First, ensure you have Scrapy installed. If not, you can install it using:

```bash
pip install scrapy
```

Create a new Scrapy project:

```bash
scrapy startproject site_crawler
cd site_crawler
```

---

## **2. Spider Code**

Navigate to the `spiders` directory:

```bash
cd site_crawler/spiders
```

Create a new file named `site_spider.py` and add the following code:

### **`site_spider.py`**

```python
import os
import scrapy
from scrapy.spiders import CrawlSpider, Rule
from scrapy.linkextractors import LinkExtractor

class SiteSpider(CrawlSpider):
    name = 'site_spider'

    # Allowed domains and start_urls will be set in the __init__ method
    allowed_domains = []
    start_urls = []

    # Define crawling rules
    rules = (
        Rule(LinkExtractor(allow=()), callback='parse_page', follow=True),
    )

    def __init__(self, domain='', output_dir='', *args, **kwargs):
        super(SiteSpider, self).__init__(*args, **kwargs)
        if not domain:
            raise ValueError("Please provide a domain name using the -a domain=<domain> argument")
        self.domain = domain
        self.allowed_domains = [domain]
        self.start_urls = [f'https://{domain}/']
        # Set the output folder; if output_dir is provided, use it
        if output_dir:
            self.output_folder = os.path.join(output_dir, domain.replace('.', '_'))
        else:
            self.output_folder = domain.replace('.', '_')

    def parse_page(self, response):
        # Determine the relative URL path
        url_path = response.url.replace(f'https://{self.domain}', '')
        url_path = url_path.replace(f'http://{self.domain}', '')
        if url_path.endswith('/') or url_path == '':
            url_path += 'index.html'
        elif not url_path.endswith('.html'):
            url_path += '.html'

        # Construct the file path
        file_path = os.path.join(self.output_folder, url_path.lstrip('/'))

        # Create the directory structure if it doesn't exist
        os.makedirs(os.path.dirname(file_path), exist_ok=True)

        # Extract all text from the page's body, excluding scripts and styles
        page_text = '\n'.join(response.xpath(
            '//body//text()[normalize-space() and not(ancestor::script) and not(ancestor::style)]'
        ).getall()).strip()

        # Save the text to the file
        with open(file_path, 'w', encoding='utf-8') as f:
            f.write(page_text)
```

---

## **3. Adjusting the Domain Name**

You can adjust the domain name and output directory without modifying the code by passing them as arguments when running the spider.

**Example:**

```bash
scrapy crawl site_spider -a domain=london-electricians-247.co.uk -a output_dir=scraped_sites
```

- **domain**: The domain you wish to crawl.
- **output_dir**: (Optional) The base directory where the scraped data will be saved.

---

## **4. Running the Spider**

Navigate back to the project root directory:

```bash
cd ..
```

Run the spider with the desired domain:

```bash
scrapy crawl site_spider -a domain=london-electricians-247.co.uk -a output_dir=scraped_sites
```

This command will:

- Crawl the website `https://london-electricians-247.co.uk/`
- Extract all text from each page
- Save each page's content into separate folders, maintaining the website's structure
- Place the scraped data into a folder named `scraped_sites/london_electricians_247_co_uk`

---

## **5. Settings Adjustment (Optional)**

If you want to adjust the Scrapy settings, open `settings.py` in the `site_crawler` directory.

### **`settings.py`**

```python
# Disable obeying robots.txt
ROBOTSTXT_OBEY = False

# Set a custom user agent
USER_AGENT = 'site_spider/1.0 (+https://yourdomain.com)'

# Set a download delay (in seconds)
DOWNLOAD_DELAY = 0.5

# Enable logging
LOG_LEVEL = 'INFO'
```

---

## **6. Directory Structure**

After running the spider, your project directory will look like this:

```
site_crawler/
├── scraped_sites/
│   └── london_electricians_247_co_uk/
│       ├── index.html
│       ├── services/
│       │   ├── index.html
│       │   ├── emergency-electrician.html
│       │   └── electrical-installations.html
│       ├── contact/
│       │   └── index.html
│       └── about/
│           └── index.html
├── site_crawler/
│   ├── __init__.py
│   ├── items.py
│   ├── middlewares.py
│   ├── pipelines.py
│   ├── settings.py
│   └── spiders/
│       ├── __init__.py
│       └── site_spider.py
└── scrapy.cfg
```

---

## **7. Code Explanation**

### **Imports**

```python
import os
import scrapy
from scrapy.spiders import CrawlSpider, Rule
from scrapy.linkextractors import LinkExtractor
```

- **os**: For handling file paths and directories.
- **scrapy**: Scrapy framework for web crawling.
- **CrawlSpider**: A spider for crawling regular websites, following links.
- **Rule** and **LinkExtractor**: To define how to follow links.

### **Spider Class Definition**

```python
class SiteSpider(CrawlSpider):
    name = 'site_spider'
```

- **name**: The name used to call the spider from the command line.

### **Allowed Domains and Start URLs**

```python
    allowed_domains = []
    start_urls = []
```

- These are initialized as empty lists and set in the `__init__` method.

### **Crawling Rules**

```python
    rules = (
        Rule(LinkExtractor(allow=()), callback='parse_page', follow=True),
    )
```

- **LinkExtractor(allow=())**: Extracts all links.
- **callback='parse_page'**: Calls the `parse_page` method for each response.
- **follow=True**: Continues to follow links from the extracted pages.

### **Initialization Method**

```python
    def __init__(self, domain='', output_dir='', *args, **kwargs):
        super(SiteSpider, self).__init__(*args, **kwargs)
        if not domain:
            raise ValueError("Please provide a domain name using the -a domain=<domain> argument")
        self.domain = domain
        self.allowed_domains = [domain]
        self.start_urls = [f'https://{domain}/']
        # Set the output folder
        if output_dir:
            self.output_folder = os.path.join(output_dir, domain.replace('.', '_'))
        else:
            self.output_folder = domain.replace('.', '_')
```

- **domain**: The domain to crawl, provided via command-line argument.
- **allowed_domains**: Restricts the crawl to this domain.
- **start_urls**: The starting point of the crawl.
- **output_folder**: The directory where the scraped data will be saved.

### **Parsing Pages**

```python
    def parse_page(self, response):
        # Determine the relative URL path
        url_path = response.url.replace(f'https://{self.domain}', '')
        url_path = url_path.replace(f'http://{self.domain}', '')
        if url_path.endswith('/') or url_path == '':
            url_path += 'index.html'
        elif not url_path.endswith('.html'):
            url_path += '.html'

        # Construct the file path
        file_path = os.path.join(self.output_folder, url_path.lstrip('/'))

        # Create the directory structure if it doesn't exist
        os.makedirs(os.path.dirname(file_path), exist_ok=True)

        # Extract all text from the page's body, excluding scripts and styles
        page_text = '\n'.join(response.xpath(
            '//body//text()[normalize-space() and not(ancestor::script) and not(ancestor::style)]'
        ).getall()).strip()

        # Save the text to the file
        with open(file_path, 'w', encoding='utf-8') as f:
            f.write(page_text)
```

- **URL Path Determination**: Calculates the relative path for saving files.
- **File Path Construction**: Determines where to save the content, maintaining the directory structure.
- **Directory Creation**: Creates necessary directories if they don't exist.
- **Text Extraction**: Retrieves all visible text from the page's body, excluding scripts and styles.
- **File Saving**: Writes the extracted text to a file.

---

## **8. Customization Options**

### **Adjusting the Text Extraction**

If you want to exclude certain elements (like navigation menus or footers), you can modify the XPath expression in the `page_text` variable.

**Example: Excluding `<nav>` and `<footer>` elements**

```python
page_text = '\n'.join(response.xpath(
    '//body//*[not(ancestor::nav or ancestor::footer)]//text()[normalize-space() and not(ancestor::script) and not(ancestor::style)]'
).getall()).strip()
```

### **Adjusting File Extensions**

If your website uses different file extensions, adjust the file naming logic accordingly.

**Example:**

```python
if url_path.endswith('/') or url_path == '':
    url_path += 'index.html'
elif not any(url_path.endswith(ext) for ext in ['.html', '.php', '.asp']):
    url_path += '.html'
```

---

## **9. Handling HTTPS and HTTP**

The code accounts for both `https` and `http` protocols when determining the URL path:

```python
url_path = response.url.replace(f'https://{self.domain}', '')
url_path = url_path.replace(f'http://{self.domain}', '')
```

---

## **10. Excluding Query Parameters**

If you want to exclude URLs with query parameters (e.g., `?session=123`), you can modify the URL path extraction:

```python
from urllib.parse import urlparse

def parse_page(self, response):
    # Parse the URL to exclude query parameters
    parsed_url = urlparse(response.url)
    url_path = parsed_url.path
    # Rest of the code remains the same
```

---

## **11. Testing and Troubleshooting**

### **Testing the Spider**

Use Scrapy's shell to test your selectors:

```bash
scrapy shell 'https://london-electricians-247.co.uk/'
```

In the shell:

```python
response.xpath('//body//text()[normalize-space() and not(ancestor::script) and not(ancestor::style)]').getall()
```

### **Common Issues**

- **No Output Files**: Ensure you're running the spider from the project root and that the domain is correctly specified.
- **Spider Not Found**: Make sure the `site_spider.py` file is in the `spiders` directory.
- **Permission Errors**: Check your file system permissions for writing files.
- **Encoding Issues**: Ensure files are saved with `encoding='utf-8'`.

---

## **12. Conclusion**

You've now set up a Scrapy spider that:

- **Crawls any specified website**.
- **Extracts all text content from each page**.
- **Saves each page's content into separate folders**, maintaining the website's structure.
- **Allows easy adjustment of the domain name** without modifying the code.
- **Customizes output folders** based on your requirements.

Feel free to adjust the code to suit your specific needs.

---

**Note**: Always ensure you have the right to crawl and extract data from the websites you target. Since you mentioned owning the data, you should be good to go.

If you have any further questions or need additional assistance, please don't hesitate to ask!

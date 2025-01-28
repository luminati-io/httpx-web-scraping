# Web Scraping With HTTPX in Python

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

This guide explains how to use HTTPX, a powerful Python HTTP client, for web scraping:

- [What Is HTTPX?](#what-is-httpx)
- [Scraping with HTTPX: Step-By-Step Guide](#scraping-with-httpx-step-by-step-guide)
  - [Step #1: Project Setup](#step-1-project-setup)
  - [Step #2: Install the Scraping Libraries](#step-2-install-the-scraping-libraries)
  - [Step #3: Retrieve the HTML of the Target Page](#step-3-retrieve-the-html-of-the-target-page)
  - [Step #4: Parse the HTML](#step-4-parse-the-html)
  - [Step #5: Scrape Data From It](#step-5-scrape-data-from-it)
  - [Step #6: Export the Scraped Data](#step-6-export-the-scraped-data)
  - [Step #7: Put It All Together](#step-7-put-it-all-together)
- [HTTPX Web Scraping Advanced Features and Techniques](#httpx-web-scraping-advanced-features-and-techniques)
  - [Set Custom Headers](#set-custom-headers)
  - [Set a Custom User Agent](#set-a-custom-user-agent)
  - [Set Cookies](#set-cookies)
  - [Proxy Integration](#proxy-integration)
  - [Error Handling](#error-handling)
  - [Session Handling](#session-handling)
  - [Async API](#async-api)
  - [Retry Failed Requests](#retry-failed-requests)
- [HTTPX vs Requests for Web Scraping](#httpx-vs-requests-for-web-scraping)
- [Conclusion](#conclusion)


## What Is HTTPX?

[HTTPX](https://github.com/projectdiscovery/httpx) is a fully featured HTTP client for Python 3, built on top of the [`retryablehttp`](https://github.com/projectdiscovery/retryablehttp-go) library. It is built to deliver reliable performance even under heavy multithreading. HTTPX offers both synchronous and asynchronous APIs, supporting HTTP/1.1 and HTTP/2 protocols.

**Features**

- **Simple and Modular Codebase**: Designed for ease of contribution and extension.
- **Fast and Configurable**: Offers fully customizable flags for probing multiple elements.
- **Versatile Probing Methods**: Supports various HTTP-based probing techniques.
- **Automatic Protocol Fallback**: Defaults to smart fallback from HTTPS to HTTP.
- **Flexible Input Options**: Accepts hosts, URLs, and CIDR as input.
- **Advanced Features**: Includes support for proxies, custom HTTP headers, configurable timeouts, basic authentication, and more.

**Pros**

- **Command-Line Availability**: Accessible via [`httpx[cli]`](https://pypi.org/project/httpx/).
- **Feature-Rich**: Includes support for HTTP/2 and an asynchronous API.
- **Actively Developed**: Continuously improved with regular updates.

**Cons**

- **Frequent Updates**: New releases may introduce breaking changes.
- **Less Popular**: Not as widely used as the [`requests`](https://requests.readthedocs.io/en/latest/) library.

## Scraping with HTTPX: Step-By-Step Guide

HTTPX is an HTTP client, so parse and extract data from the HTML it retrieved, you will need an HTML parser like [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/).

> **Warning**:\
> While HTTPX is only used in the early stages of the process, we will walk you through a complete workflow. If you are interested in more advanced HTTPX scraping techniques, you can skip ahead to the next chapter after Step 3.

### Step #1: Project Setup

Install Python 3+ on your machine and create a directory for your HTTPX scraping project:

```bash
mkdir httpx-scraper
```

Navigate into it and initialize a [virtual environment](https://docs.python.org/3/library/venv.html):

```bash
cd httpx-scraper
python -m venv env
```

Open the project folder in your Python IDE, create a `scraper.py` file inside the folder, then activate the virtual environment. On Linux or macOS, run:

```bash
./env/bin/activate
```

On Windows:

```
env/Scripts/activate
```

### Step #2: Install the Scraping Libraries

Install HTTPX and BeautifulSoup:

```bash
pip install httpx beautifulsoup4
```

Import the added dependencies into your `scraper.py` script:

```python
import httpx
from bs4 import BeautifulSoup
```

### Step #3: Retrieve the HTML of the Target Page

In this example, the target page will be the “[Quotes to Scrape](https://quotes.toscrape.com/)” site:

![The Quotes To Scrape homepage](https://github.com/luminati-io/httpx-web-scraping/blob/main/Images/image-70.png)

Use HTTPX to retrieve the HTML of the homepage with the `get()` method:

```python
# Make an HTTP GET request to the target page
response = httpx.get("http://quotes.toscrape.com")
```

Behind the scenes, HTTPX will perform an HTTP GET request to the server, which will respond with the page's HTML. You can access the HTML content using the `response.text` attribute:

```python
html = response.text
print(html)
```

This will print the raw HTML content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Quotes to Scrape</title>
    <link rel="stylesheet" href="/static/bootstrap.min.css">
    <link rel="stylesheet" href="/static/main.css">
</head>
<body>
    <!-- omitted for brevity... -->
</body>
</html>
```

### Step #4: Parse the HTML

Feed the HTML content to the BeautifulSoup constructor to parse it:

```python
# Parse the HTML content using
BeautifulSoup soup = BeautifulSoup(html, "html.parser")
```

The `soup` variable now holds the parsed HTML and exposes the methods to extract the data.

### Step #5: Scrape Data From It

Scrape quotes data from the page:

```python
# Where to store the scraped data
quotes = []

# Extract all quotes from the page
quote_elements = soup.find_all("div", class_="quote")

# Loop through quotes and extract text, author, and tags
for quote_element in quote_elements:
    text = quote_element.find("span", class_="text").get_text().get_text().replace("“", "").replace("”", "")
    author = quote_element.find("small", class_="author")
    tags = [tag.get_text() for tag in quote_element.find_all("a", class_="tag")]

    # Store the scraped data
    quotes.append({
        "text": text,
        "author": author,
        "tags": tags
    })
```

This snippet defines a list named `quotes` to store the scraped data. It then selects all quote HTML elements and loops through them to extract the quote text, author, and tags. Each extracted quote is stored as a dictionary within the `quotes` list, organizing the data for further use or export.

### Step #6: Export the Scraped Data

Export the scraped data to a CSV file:

```python
# Specify the file name for export
with open("quotes.csv", mode="w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=["text", "author", "tags"])

    # Write the header row
    writer.writeheader()

    # Write the scraped quotes data
    writer.writerows(quotes)
```

This snippet opens a file named `quotes.csv` in write mode, defines column headers (`text`, `author`, `tags`), writes the headers to the file, and then writes each dictionary from the `quotes` list to the CSV file. The `csv.DictWriter` handles the formatting, making it easy to store structured data.

Import `csv` from the Python Standard Library:

```python
import csv
```

### Step #7: Put It All Together

The final HTTPX web scraping script will contain the following code:

```python
import httpx
from bs4 import BeautifulSoup
import csv

# Make an HTTP GET request to the target page
response = httpx.get("http://quotes.toscrape.com")

# Access the HTML of the target page
html = response.text

# Parse the HTML content using BeautifulSoup
soup = BeautifulSoup(html, "html.parser")

# Where to store the scraped data
quotes = []

# Extract all quotes from the page
quote_elements = soup.find_all("div", class_="quote")

# Loop through quotes and extract text, author, and tags
for quote_element in quote_elements:
    text = quote_element.find("span", class_="text").get_text().replace("“", "").replace("”", "")
    author = quote_element.find("small", class_="author").get_text()
    tags = [tag.get_text() for tag in quote_element.find_all("a", class_="tag")]

    # Store the scraped data
    quotes.append({
        "text": text,
        "author": author,
        "tags": tags
    })

# Specify the file name for export
with open("quotes.csv", mode="w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=["text", "author", "tags"])

    # Write the header row
    writer.writeheader()

    # Write the scraped quotes data
    writer.writerows(quotes)
```

Execute it with:

```bash
python scraper.py
```

Or, on Linux/macOS:

```bash
python3 scraper.py
```

A `quotes.csv` file will appear in the root folder of your project withthe following contents:

![The CSV containing the scraped data](https://github.com/luminati-io/httpx-web-scraping/blob/main/Images/image-71.png)

## HTTPX Web Scraping Advanced Features and Techniques

Let's use a more complex example. The target site will be the [HTTPBin.io `/anything` endpoint](https://httpbin.io/anything). This is a special API that returns the IP address, headers, and other information sent by the caller.

### Set Custom Headers

HTTPX allows you to [specify custom headers](https://www.python-httpx.org/quickstart/#custom-headers) using the `headers` argument:

```python
import httpx

# Custom headers for the request
headers = {
    "accept": "application/json",
    "accept-language": "en-US,en;q=0.9,fr-FR;q=0.8,fr;q=0.7,es-US;q=0.6,es;q=0.5,it-IT;q=0.4,it;q=0.3"
}

# Make a GET request with custom headers
response = httpx.get("https://httpbin.io/anything", headers=headers)
# Handle the response...
```

### Set a Custom User Agent

[`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) is one of the most important [HTTP headers for web scraping](/blog/web-data/http-headers-for-web-scraping). By default, HTTPX uses the following `User-Agent`:

```
python-httpx/<VERSION>
```

This value can easily indicate that your requests are automated, potentially resulting in the target site blocking them.

To avoid that, you can set a custom `User-Agent` to mimic a real browser, like so:

```python
import httpx

# Define a custom User-Agent
headers = {
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/132.0.0.0 Safari/537.36"
}

# Make a GET request with the custom User-Agent
response = httpx.get("https://httpbin.io/anything", headers=headers)
# Handle the response...
```

### Set Cookies

Set cookies in HTTPX using the [`cookies` argument](https://www.python-httpx.org/quickstart/#cookies):

```python
import httpx

# Define cookies as a dictionary
cookies = {
    "session_id": "3126hdsab161hdabg47adgb",
    "user_preferences": "dark_mode=true"
}

# Make a GET request with custom cookies
response = httpx.get("https://httpbin.io/anything", cookies=cookies)
# Handle the response...
```

This gives you the ability to include session data required for your web scraping requests.

### Proxy Integration

Now [route your HTTPX requests through a proxy](https://www.python-httpx.org/advanced/proxies/) to protect your identity and avoid IP bans while performing web scraping. To doo that, use the `proxies` argument:

```python
import httpx

# Replace with the URL of your proxy server
proxy = "<YOUR_PROXY_URL>"

# Make a GET request through a proxy server
response = httpx.get("https://httpbin.io/anything", proxy=proxy)
# Handle the response...
```

### Error Handling

By default, HTTPX raises errors only for [connection or network issues](https://www.python-httpx.org/exceptions/). To raise exceptions also for HTTP responses with [`4xx`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#client_error_responses) and [`5xx`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#server_error_responses) status codes,use the `raise_for_status()` method as below:

```python
import httpx

try:
    response = httpx.get("https://httpbin.io/anything")
    # Raise an exception for 4xx and 5xx responses
    response.raise_for_status()
    # Handle the response...
except httpx.HTTPStatusError as e:
    # Handle HTTP status errors
    print(f"HTTP error occurred: {e}")
except httpx.RequestError as e:
    # Handle connection or network errors
    print(f"Request error occurred: {e}")
```

### Session Handling


When using the top-level API in HTTPX, a new connection is created for each request, meaning TCP connections are not reused. This approach becomes inefficient as the number of requests to a host increases.

Meanwhile, using a `httpx.Client` instance enables [HTTP connection pooling](https://devblogs.microsoft.com/premier-developer/the-art-of-http-connection-pooling-how-to-optimize-your-connections-for-peak-performance/). This means that multiple requests to the same host can reuse an existing TCP connection instead of creating a new one for each request.

Here are some of the benefits of using a `Client` over the top-level API:

- Reduced latency across requests, because there is no repeated handshaking
- Lower CPU usage and fewer round-trips
- Decreased network traffic

Additionally, `Client` instances support session handling with features unavailable in the top-level API, including:

- Cookie persistence across requests
- Applying configuration across all outgoing requests
- Sending requests through HTTP proxies

It is typically recommended to use a `Client` in HTTPX with a context manager (`with` statement):

```python
import httpx

with httpx.Client() as client:
    # Make an HTTP request using the client
    response = client.get("https://httpbin.io/anything")

    # Extract the JSON response data and print it
    response_data = response.json()
    print(response_data)
```

Alternatively, you can manually manage the client and close the connection pool explicitly with `client.close()`:

```python
import httpx

client = httpx.Client()
try:
    # Make an HTTP request using the client
    response = client.get("https://httpbin.io/anything")

    # Extract the JSON response data and print it
    response_data = response.json()
    print(response_data)
except:
  # Handle the error...
  pass
finally:
  # Close the client connections and release resources
  client.close()
```

> **Note**:\
> If you are familiar with the `requests` library, `httpx.Client()` serves a similar purpose to [`requests.Session()`](https://requests.readthedocs.io/en/latest/user/advanced/#session-objects).

### Async API

By default, HTTPX exposes a standard synchronous API. At the same time, it also offers an [asynchronous client](https://www.python-httpx.org/async/) for when it's required. If you are working with [`asyncio`](https://docs.python.org/3/library/asyncio.html), using an async client is essential for sending outgoing HTTP requests efficiently.

To make asynchronous requests in HTTPX, initialize `AsyncClient` and use it to make a GET request as shown below:

```python
import httpx
import asyncio

async def fetch_data():
    async with httpx.AsyncClient() as client:
        # Make an async HTTP request
        response = await client.get("https://httpbin.io/anything")

        # Extract the JSON response data and print it
        response_data = response.json()
        print(response_data)

# Run the async function
asyncio.run(fetch_data())
```

The [`with`](https://docs.python.org/3/reference/compound_stmts.html#with) statement ensures the client is automatically closed when the block ends. Alternatively, if you manage the client manually, you can close it explicitly with `await client.close()`.

All HTTPX request methods (`get()`, `post()`, etc.) are asynchronous when using an `AsyncClient`. Therefore, you must add `await` before calling them to get a response.

### Retry Failed Requests

Network instability during web scraping may result in connection failures or timeouts. HTTPX simplifies handling such issues via its [`HTTPTransport`](https://www.python-httpx.org/advanced/transports/) interface. This mechanism retries requests when an `httpx.ConnectError` or `httpx.ConnectTimeout` occurs.

The following code demonstrates how to configure a transport to retry requests up to 3 times:

```python
import httpx

# Configure transport with retry capability on connection errors or timeouts
transport = httpx.HTTPTransport(retries=3)

# Use the transport with an HTTPX client
with httpx.Client(transport=transport) as client:
    # Make a GET request
    response = client.get("https://httpbin.io/anything")
    # Handle the response...
```

Only connection-related errors trigger a retry. To handle read/write errors or specific HTTP status codes, you need to implement custom retry logic with libraries like [`tenacity`](https://tenacity.readthedocs.io/en/latest/).

## HTTPX vs Requests for Web Scraping

The following table compares HTTPX and [Requests for web scraping](/blog/web-data/python-requests-guide):

| **Feature** | **HTTPX** | **Requests** |
| --- | --- | --- |
| **GitHub stars** | 8k  | 52.4k |
| **Async support** | ✔️  | ❌   |
| **Connection pooling** | ✔️  | ✔️  |
| **HTTP/2 support** | ✔️  | ❌   |
| **User-agent customization** | ✔️  | ✔️  |
| **Proxy support** | ✔️  | ✔️  |
| **Cookie handling** | ✔️  | ✔️  |
| **Timeouts** | Customizable for connection and read | Customizable for connection and read |
| **Retry mechanism** | Available via transports | Available via `HTTPAdapter`s |
| **Performance** | High | Medium |
| **Community support and popularity** | Growing | Large |

## Conclusion

Automated HTTP requests expose your public IP address, potentially revealing your identity and location, which compromises your privacy. To enhance your security and privacy, use a proxy server to hide your IP address.

Bright Data controls the best proxy servers in the world, serving Fortune 500 companies and more than 20,000 customers. Its offer includes a wide range of proxy types:

- [Datacenter proxies](https://brightdata.com/proxy-types/datacenter-proxies) – Over 770,000 datacenter IPs.
- [Residential proxies](https://brightdata.com/proxy-types/residential-proxies) – Over 72M residential IPs in more than 195 countries.
- [ISP proxies](https://brightdata.com/proxy-types/isp-proxies) – Over 700,000 ISP IPs.
- [Mobile proxies](https://brightdata.com/proxy-types/mobile-proxies) – Over 7M mobile IPs.

Create a free Bright Data account today to test our scraping solutions and proxies!

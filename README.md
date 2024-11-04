# PriceAlert
This Python script tracks the price of a product on Amazon and sends an email alert when the price falls below a target value. Here's a detailed breakdown of each part:
### 1. Import Statements

```python
import requests
from bs4 import BeautifulSoup
import smtplib
from email.mime.text import MIMEText
import time
import re
```

These libraries perform the following tasks:
- **`requests`**: Fetches the webpage HTML content.
- **`BeautifulSoup`**: Parses and locates elements in the HTML content (e.g., the product price).
- **`smtplib`** and **`MIMEText`**: Handle email sending.
- **`time`**: Pauses the script for a set interval to avoid constant checking.
- **`re`**: Provides regular expressions for text cleaning, like removing commas from price values.

---

### 2. `get_price` Function

```python
def get_price(url):
```

This function fetches and parses the price from the given Amazon URL.

- **Request Configuration**: The script sends an HTTP GET request with custom headers (like `User-Agent`, `Accept`, `DNT`) to simulate a legitimate browser request, preventing blocks by Amazon's security.
- **HTML Parsing**:
  ```python
  soup = BeautifulSoup(response.content, 'html.parser')
  ```
  It loads the HTML content into `BeautifulSoup` for parsing.

- **Price Extraction**:
  ```python
  price_element = soup.select_one('.a-price-whole')
  ```
  The `soup.select_one()` function locates the price on the page. The CSS selector `.a-price-whole` targets the specific element holding the price on Amazon's page.

- **Data Cleaning**:
  ```python
  price_text = price_element.text.strip()
  price_number = re.findall(r'[\d,]+', price_text)
  ```
  If `price_element` is found, it extracts text and uses regex (`re.findall`) to locate numbers, stripping out commas.

- **Conversion**:
  ```python
  price = float(price_number[0].replace(',', ''))
  ```
  This converts the cleaned text to a floating-point number to use in comparisons.

---

### 3. `send_email` Function

```python
def send_email(subject, body, to_email, from_email, from_password):
```

This function sends an email notification when the product's price drops below the target.

- **Email Body**:
  ```python
  msg = MIMEText(body)
  ```
  MIMEText formats the email content with a subject line and recipient information.

- **SMTP Connection**:
  ```python
  server = smtplib.SMTP_SSL('smtp.gmail.com', 465)
  server.login(from_email, from_password)
  ```
  Establishes a secure SSL connection to Gmail's SMTP server (port 465) and logs in using `from_email` and `from_password`.

- **Sending the Email**:
  ```python
  server.send_message(msg)
  server.quit()
  ```
  The message is sent, and the connection is closed.

---

### 4. `track_price` Function

```python
def track_price(url, target_price, check_interval, to_email, from_email, from_password):
```

This function continuously monitors the product price, sending an alert if the price falls below a specified target.

- **Loop and Check**:
  ```python
  while True:
      current_price = get_price(url)
  ```
  The `while True` loop ensures continuous monitoring. The script calls `get_price` to retrieve the latest price.

- **Price Comparison**:
  ```python
  if current_price <= target_price:
  ```
  If the `current_price` is equal to or less than `target_price`, it triggers `send_email` to alert the user.

- **Rechecking**:
  ```python
  time.sleep(check_interval * 3600)
  ```
  The script pauses for `check_interval` hours (converted to seconds) before rechecking the price.

---

### 5. Main Execution Block

```python
if __name__ == "__main__":
```

The final block initializes key parameters like the product URL, target price, email information, and check interval, and calls `track_price`.

### Example Execution

- **URL**: Amazon product URL.
- **Target Price**: ₹38000, the threshold for sending an alert.
- **Check Interval**: 1 hour.
- **Emails**:
  - **`from_email`**: Email to send alerts.
  - **`from_password`**: App password (for secure login without needing the actual Gmail password).
  - **`to_email`**: Email to receive the alert.

---

This script is useful for tracking price drops on Amazon and automating notifications based on price changes. However, note that scraping e-commerce sites may violate their terms of service, so verify Amazon’s policy before running the script continuously.

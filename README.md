# README - Stock Data Extraction and Visualization

## Overview

This project demonstrates how to extract stock data and revenue information for **Tesla (TSLA)** and **GameStop (GME)**, and how to visualize them using Python libraries. The stock data is retrieved using the **yfinance** library, while the revenue data is scraped from web pages using **BeautifulSoup**. After extracting the necessary data, we plot two graphs: one showing the historical stock prices and the other showing the historical revenue for both Tesla and GameStop.

### Key Highlights:
- **Tesla Stock**: From its early days until 2021.
- **GameStop Stock**: From its early days until 2021.
- **Revenue Data**: Extracted from publicly available sources for both companies.

## Libraries Used

- **yfinance**: For fetching historical stock data.
- **requests**: For making HTTP requests to retrieve HTML content from web pages.
- **BeautifulSoup**: For scraping revenue data from HTML tables.
- **plotly**: For creating interactive visualizations.
- **pandas**: For handling and cleaning data.

## Steps

### 1. Install Dependencies

Before running the code, you need to install the necessary libraries. You can do this by running the following commands:

```bash
!pip install yfinance==0.1.67
!pip install requests
!pip install beautifulsoup4
!pip install plotly
```

### 2. Fetching Stock Data for Tesla

We use **yfinance** to download historical stock data for **Tesla** (Ticker: **TSLA**).

```python
tsla = yf.Ticker('TSLA')
tesla_data = tsla.history(period='max')
```

- **Explanation**: The `yf.Ticker('TSLA')` fetches the data for Tesla using its ticker symbol. The `history(period='max')` function retrieves the stock data over the maximum available time period.

Example Output (first 5 rows of the Tesla stock data):

| Date       | Open   | High   | Low    | Close  | Volume   |
|------------|--------|--------|--------|--------|----------|
| 2010-06-29 | 19.00  | 19.15  | 18.75  | 19.00  | 1650000  |
| 2010-06-30 | 18.88  | 19.25  | 18.75  | 19.00  | 900000   |
| ...        | ...    | ...    | ...    | ...    | ...      |

After fetching the data, we reset the index and display the first five rows:

```python
tesla_data.reset_index(inplace=True)
tesla_data.head()
```

### 3. Web Scraping for Tesla's Revenue Data

Revenue data for Tesla is scraped from an HTML table located at the given URL. We use **requests** to fetch the page and **BeautifulSoup** to parse the HTML content.

```python
url = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0220EN-SkillsNetwork/labs/project/revenue.htm"
page = requests.get(url)
soup = BeautifulSoup(page.text, 'html')
```

- **Explanation**: The `requests.get(url)` fetches the HTML page, and `BeautifulSoup(page.text, 'html')` parses the content for easier scraping.

We extract the revenue data from the table:

```python
tesla_revenue = pd.DataFrame(columns=['Date', 'Revenue'])
table = soup.find_all('table')[0]
column_data = table.find_all('tr')

for column in column_data[1:]:
    row = column.find_all('td')
    lst = [data.text.strip() for data in row]
    tesla_revenue.loc[len(tesla_revenue)] = lst
```

Example of the **Tesla Revenue** table (after scraping):

| Date       | Revenue ($US Millions) |
|------------|------------------------|
| 2021-03-31 | 31,536                 |
| 2020-12-31 | 27,236                 |
| 2020-09-30 | 23,300                 |
| 2020-06-30 | 25,000                 |
| ...        | ...                    |

We clean the revenue data by removing commas and dollar signs:

```python
tesla_revenue["Revenue"] = tesla_revenue['Revenue'].str.replace(',|\$',"")
tesla_revenue.dropna(inplace=True)
tesla_revenue = tesla_revenue[tesla_revenue['Revenue'] != ""]
```

### 4. Fetching Stock Data for GameStop

We repeat the same process for **GameStop** (Ticker: **GME**), using the **yfinance** library:

```python
ticker = yf.Ticker('GME')
gme_data = ticker.history(period='max')
gme_data.reset_index(inplace=True)
gme_data.head()
```

Example Output (first 5 rows of GameStop stock data):

| Date       | Open   | High   | Low    | Close  | Volume    |
|------------|--------|--------|--------|--------|-----------|
| 2002-02-11 | 9.00   | 9.50   | 8.75   | 9.25   | 4200000   |
| 2002-02-12 | 9.50   | 9.75   | 9.25   | 9.25   | 3800000   |
| ...        | ...    | ...    | ...    | ...    | ...       |

### 5. Web Scraping for GameStop's Revenue Data

GameStop's revenue data is also scraped from a separate HTML table, similar to the process for Tesla.

```python
url = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0220EN-SkillsNetwork/labs/project/stock.html"
page = requests.get(url)
soup = BeautifulSoup(page.text, 'html')

gme_revenue = pd.DataFrame(columns=["Date", "Revenue"])
table = soup.find_all('table')[0]
column_data = table.find_all('tr')

for column in column_data[1:]:
    row = column.find_all('td')
    lst = [data.text.strip() for data in row]
    gme_revenue.loc[len(gme_revenue)] = lst

gme_revenue["Revenue"] = gme_revenue['Revenue'].str.replace(',|\$',"")
gme_revenue.dropna(inplace=True)
gme_revenue = gme_revenue[gme_revenue['Revenue'] != ""]
```

Example of **GameStop Revenue** table (after scraping):

| Date       | Revenue ($US Millions) |
|------------|------------------------|
| 2021-03-31 | 6,000                  |
| 2020-12-31 | 5,900                  |
| 2020-09-30 | 5,000                  |
| 2020-06-30 | 4,600                  |
| ...        | ...                    |

### 6. Plotting the Data

We define a function, `make_graph()`, to plot the **Stock Data** and **Revenue Data** in two subplots. The function takes stock data, revenue data, and the stock's name as input.

```python
def make_graph(stock_data, revenue_data, stock):
    fig = make_subplots(rows=2, cols=1, shared_xaxes=True, subplot_titles=("Historical Share Price", "Historical Revenue"), vertical_spacing = .3)
    
    stock_data_specific = stock_data[stock_data.Date <= '2021-06-14']
    revenue_data_specific = revenue_data[revenue_data.Date <= '2021-04-30']
    
    fig.add_trace(go.Scatter(x=pd.to_datetime(stock_data_specific.Date, infer_datetime_format=True), 
                             y=stock_data_specific.Close.astype("float"), 
                             name="Share Price"), row=1, col=1)
    fig.add_trace(go.Scatter(x=pd.to_datetime(revenue_data_specific.Date, infer_datetime_format=True), 
                             y=revenue_data_specific.Revenue.astype("float"), 
                             name="Revenue"), row=2, col=1)
    
    fig.update_xaxes(title_text="Date", row=1, col=1)
    fig.update_xaxes(title_text="Date", row=2, col=1)
    fig.update_yaxes(title_text="Price ($US)", row=1, col=1)
    fig.update_yaxes(title_text="Revenue ($US Millions)", row=2, col=1)
    fig.update_layout(showlegend=False, height=900, title=stock, xaxis_rangeslider_visible=True)
    fig.show()
```

The function generates an interactive plot with:
- **Top subplot**: Historical **Stock Prices**.
- **Bottom subplot**: Historical **Revenue**.

### 7. Plotting the Graphs for Tesla and GameStop

Finally, we use the `make_graph()` function to plot the data for both companies:

```python
make_graph(tesla_data, tesla_revenue, 'Tesla')
make_graph(gme_data, gme_revenue, 'GameStop')
```


## Conclusion

This project demonstrates how to extract and visualize stock data and company revenue data using Python. By leveraging libraries like **yfinance

** for stock data, **BeautifulSoup** for web scraping, and **Plotly** for interactive visualizations, we can gain valuable insights into the performance of companies like **Tesla** and **GameStop**. You can easily modify the code to analyze other companies by simply updating the ticker symbols and revenue URLs.

---

### Notes:
- The dataset is cleaned and preprocessed to remove unwanted characters like commas and dollar signs from the revenue data.
- The date range for plotting is set up to visualize the data up to **June 2021** for stock data and **April 2021** for revenue data.

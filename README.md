# FWD---S&P-500-Cortex-Analyst

## 📰 S&P 500 News Article Content Scrapping 

This project fetches articles from the NewsAPI, extracts full article content using the `newspaper` library, and stores the data in a Snowflake database. The project also saves the articles in a CSV file for local storage.

## Features

- Fetches articles from multiple sources using the NewsAPI.
- Extracts full article content from URLs.
- Saves articles to a CSV file.
- Inserts articles into a Snowflake database.

## Prerequisites

- Python 3.8 or higher
- A Snowflake account
- A NewsAPI key

## Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd FWDDemo
   ```

2. Install the required Python packages:
   ```bash
   pip install -r requirements.txt
   ```

3. Create a `.env` file in the project directory with the following variables:
   ```env
   SF_ACCOUNT=<your_snowflake_account>
   SF_USER=<your_snowflake_username>
   SF_PASSWORD=<your_snowflake_password>
   SF_WAREHOUSE=<your_snowflake_warehouse>
   SF_DATABASE=<your_snowflake_database>
   SF_SCHEMA=<your_snowflake_schema>
   SF_TABLE=<your_snowflake_table>
   API_KEY=<your_newsapi_key>
   ```

## Usage

1. Run the script:
   ```bash
   python main2.py
   ```

2. The script will:
   - Fetch articles from the NewsAPI for the specified date range.
   - Extract full article content.
   - Save the articles to `articles.csv`.
   - Insert the articles into the specified Snowflake table.

## Requirements

The following Python packages are required and listed in `requirements.txt`:

- `snowflake-connector-python`
- `snowflake-connector-python[pandas]`
- `newspaper`
- `lxml_html_clean`
- `openpyxl`

## File Structure

- `main2.py`: The main script for fetching, processing, and storing articles.
- `articles.csv`: The CSV file where articles are saved locally.
- `.env`: Environment variables for API keys and Snowflake credentials.
- `requirements.txt`: List of required Python packages.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

## Acknowledgments

- [NewsAPI](https://newsapi.org/) for providing the article data.
- [Snowflake](https://www.snowflake.com/) for cloud data storage.
- [Newspaper](https://newspaper.readthedocs.io/) for article content extraction.


## ❄️ S&P 500 News Article Cortex Analysis on Streamlit

This repository provides a framework for analyzing S&P 500 companies using Snowflake for data storage and processing, and Streamlit for interactive data visualization. The project includes fuzzy matching of company names in articles and sentiment analysis of article content.

## Features

- Snowflake environment setup for S&P 500 analysis
- S&P 500 company data integration from Snowflake Marketplace
- Article content processing with fuzzy matching to identify company mentions
- Sentiment analysis on news articles content
- S&P 500 article data consolidation

## Prerequisites

- Snowflake account with admin privileges
- Access to Snowflake Marketplace
- Python 3.7+
- Required Python packages:
  - streamlit
  - pandas
  - snowflake-snowpark-python
  - rapidfuzz

## Implementation Guide

### 1. Snowflake Environment Setup

First, set up your Snowflake environment by running the SQL setup commands in the `FWD_SP500_CORTEX_ANALYST_ENVIRONMENT.ipynb` file. These commands will:

- Enable Cortex on unsupported regions
- Create the FWDDEMO database and DEMO schema
- Create a stage for data loading
- Create a warehouse for analysis operations

Run the following commands in a Snowflake Worksheet to set up your environment:
```sql
-- Enable Cortex on unsupported regions
ALTER ACCOUNT SET CORTEX_ENABLED_CROSS_REGION = 'AWS_US'; 

-- Create database, schema, and stage
CREATE OR REPLACE DATABASE FWDDEMO;
CREATE OR REPLACE SCHEMA DEMO;
CREATE STAGE STG_DEMO;

-- Create warehouse for analysis
CREATE OR REPLACE WAREHOUSE CORTEX_ANALYST_WH 
WAREHOUSE_SIZE = 'LARGE'  
AUTO_SUSPEND = 60;
```

### 2. Data Import and View Creation

Follow these steps to import the necessary data:

#### Import S&P 500 Reference Data
1. Upload the `SP_500.csv` file into `FWDDEMO.PUBLIC` schema
   - Rename columns name: 
     - `SYMBOL`, `SECURITY`, `GICS_SECTOR`, `GICS_SUB_INDUSTRY`, `HEADQUARTERS_LOCATION`, `DATE_ADDED`, `CIK`, `FOUNDED`

2. Upload the `S&P_500_Finance_Info.csv` file into `FWDDEMO.PUBLIC` schema
   - Rename columns name: 
     - `SYMBOL`, `NAME`, `SECTOR`, `PRICE`, `PRICE_EARNINGS`, `DIVIDEND_YIELD`, `EARNINGS_SHARE`, `"52_Week_Low"`, `"52_Week_High"`, `MARKET_CAP`, `EBITDA`, `PRICE_SALES`, `PRICE_BOOK`, `SEC_FILINGS`

3. Upload the `vader_lexicon.txt` file into `FWDDEMO.PUBLIC` schema

4. Upload the `fwd_demo.yaml` file into `FWDDEMO.DEMO.STG_DEMO` stage


#### Import S&P 500 Domain Visit Data
1. Go to Snowflake Marketplace
2. Search for "S & P 500 by domain and aggregated by tickers (Sample)"
3. Click "Get Data" and create a new dataset named `SP_500_BY_DOMAIN`

#### Create View Tables
Run the SQL commands from the `FWD_SP500_CORTEX_ANALYST_ENVIRONMENT.ipynb` file to create:
- `SP_500` view table - combines company basic information with financial metrics
- `SP_500_VISITS` view table - contains website analytics for S&P 500 companies

### 3. Fuzzy Matching Implementation

The repository contains Python code in `FWD_SP500_CORTEX_ANALYST_ENVIRONMENT.ipynb` file for fuzzy matching article content with S&P 500 company names:

1. Review the fuzzy matching code in the repository
2. The application uses RapidFuzz with a threshold of 85% for matching
3. The code fetches articles from `FWDDEMO.PUBLIC.ARTICLE_CONTENT_SCRAPED`
4. Results are saved to `FWDDEMO.PUBLIC.ArticleContentScraped_Matched`

### 4. Sentiment Analysis Implementation

Another Python code provided in `FWD_SP500_CORTEX_ANALYST_ENVIRONMENT.ipynb` file performs sentiment analysis on article content:

1. Review the sentiment analysis code in the repository
2. The analysis uses referential data of word and score from a VADER lexicon stored in `FWDDEMO.PUBLIC.VADER_LEXICON`
3. Sentiment scores are calculated for each article using custom function
4. Results are saved to `FWDDEMO.PUBLIC.ArticleContentScraped_Sentiment`


### 5. Alter column SP_500_Symbol from varchar to varant 

Copy the code below into Snowflake worksheet

```SQL
-- Alter column SP_500_Symbol from varchar to varant 
ALTER TABLE FWDDEMO.PUBLIC.ARTICLECONTENTSCRAPED_SENTIMENT
ADD COLUMN SP_500_Symbol_variant VARIANT;

UPDATE FWDDEMO.PUBLIC.ARTICLECONTENTSCRAPED_SENTIMENT
SET SP_500_Symbol_variant = TO_VARIANT("SP_500_Symbol");

UPDATE FWDDEMO.PUBLIC.ARTICLECONTENTSCRAPED_SENTIMENT
SET SP_500_Symbol_variant = PARSE_JSON("SP_500_Symbol");

ALTER TABLE FWDDEMO.PUBLIC.ARTICLECONTENTSCRAPED_SENTIMENT
DROP COLUMN "SP_500_Symbol";

ALTER TABLE FWDDEMO.PUBLIC.ARTICLECONTENTSCRAPED_SENTIMENT
RENAME COLUMN SP_500_Symbol_variant TO SP_500_Symbol;
```

### 6. Creating the Final S&P 500 Articles View

After running both the fuzzy matching and sentiment analysis applications, create the final view:

1. Use the SQL command from the `FWD_SP500_CORTEX_ANALYST_ENVIRONMENT.ipynb` file to create `SP_500_ARTICLES` view
2. This view combines article content with matched ticker symbols and sentiment scores
3. The view flattens the array of matched tickers for easier analysis

## Usage

1. Run the SQL setup commands to configure your Snowflake environment
2. Import the required data sets
3. Execute the fuzzy matching and sentiment analysis
4. Create the final view for unified data access
5. Use the resulting views for your S&P 500 analysis

### 6. Loading Cortex Analyst Streamlit Demo

To launch the final Streamlit application:

1. Create a new Streamlit app in Snowflake
2. Select the `FWDDEMO.PUBLIC` schema
3. Choose the warehouse you created (`CORTEX_ANALYST_WH`)
4. Upload the Streamlit demo code from this repository: `DEPLOY_CORTEX_ANALYST_STREAMLIT.ipynb`
5. Run the application to test the cortex analyst and analyze the S&P 500 data with the sentiment analysis results

## Additional Resources

- Refer to the code files in this repository for detailed implementation
- Check the comments in the code for specific functionality explanations
- Snowflake documentation: [Snowflake Documentation](https://docs.snowflake.com/)
- Streamlit documentation: [Streamlit Documentation](https://docs.streamlit.io/)

# Web-Scrape-Medicare-Item-Statistics
This R script scrapes Medicare services data from the Medicare Statistics website <https://medicarestatistics.humanservices.gov.au>. 
It retrieves the number of services provided for specified Medicare Benefits Schedule (MBS) or Pharmaceutical Benefits Schedule (PBS) items, broken down by month and state.

## Prerequisites
Before running the script, ensure you have the following R packages installed:
```R
-   `rvest`
-   `dplyr`
-   `tidyr`
-   `httr`
```

You can install these packages using the following command in your R console:
```R
install.packages(c("rvest", "dplyr", "tidyr",  "httr"))
```

## Input Data - MBS
    2025-03-01 MBS-XML-20250301.csv (or your  file): This csv file should contain a column named ItemNum listing the MBS item numbers for which you want to retrieve data.
    Replace this with the latest version of the MBS XML from the MBS website <https://www.mbsonline.gov.au/internet/mbsonline/publishing.nsf/Content/downloads>, converted to a csv. 
## Input Data - pBS
    PBSSchedule.csv: This CSV file should contain a column named pbs_code listing the PBS item numbers for which you want to retrieve data.
    For the PBS you will need to download the latest version of the PBS Schedule using Postman <https://data.pbs.gov.au/document/91345.html>

## Script Overview
    1. Read Input Data: It reads the MBS item numbers from the specified Excel file.
    2. Create Batches: The script divides the list of item numbers into batches of 30 to avoid overwhelming the server with requests.
    3. Iteratively Scrape html via SAS query <https://medicarestatistics.humanservices.gov.au/VEA0032/SAS.Web/statistics/mbs_item.html>:
         For each batch, it sends an HTTP GET request to the Medicare Statistics website.
         It parses the HTML response to extract the data table.
         It cleans and formats the extracted data.
        It handles errors, creating creates empty dataframes when no data is returned.
        It adds a user agent to mimic a browser.
        It adds a delay between batches to be nice to the server.
    4. Combine Results: It combines the data from all batches into a single data frame.
    5. Write to Excel: It writes the final data frame to an Excel file named medicare_services_data.xlsx.

## Usage
    1. Place the latest MBS/PBS file in the same directory as the R script and modify the file path within the script.
      available here <https://www.mbsonline.gov.au/internet/mbsonline/publishing.nsf/Content/downloads>
                        If you only want certain items, or are only looking at services provided in a given time period, update this code:
```R
                            # Make the request
                            response <- GET(
                              "https://medicarestatistics.humanservices.gov.au/SASStoredProcess/guest",
                              query = list(
                                "_PROGRAM" = "SBIP://METASERVER/Shared Data/sasdata/prod/VEA0032/SAS.StoredProcess/statistics/mbs_item_standard_report",
                                "DRILL" = "ag",
                                "group" = paste(current_batch, collapse = ","),
                                "VAR" = "services",                     # Report Variable; options: "services" or "benefits"
                                "STAT" = "count",                       # Report Statistic; options: "count" or "per capita"
                                "RPT_FMT" = "by time period and state", # Report Format, not sure what other options are available
                                "PTYPE" = "month",                      # Time Period; options: "calyear", "finyear", "month"
                                "START_DT" = "199307",                  # Start Date; or other date in YYYYMM format
                                "END_DT" = "202502"                     # latest available release YYYYMM format
                              ),
```
    2. Open the R script in RStudio or your preferred R environment.
    3. Run the script.
    4. The script will download the data, process it, and save it to medicare_services_data.xlsx in the same directory.

## Output Data
# MBS Output
The columns in the MBS output CSV are:

    Item: MBS item number.
    Month: Time period (month).
    NSW, VIC, QLD, SA, WA, TAS, ACT, NT: Number of services provided in each state/territory.
    Total: Total number of services provided across all states/territories.
    Category: MBS Category
    Group: MBS Group
    SubGroup: MBS SubGroup
    
# PBS Output
The columns in the PBS output CSV are:

    Item: PBS item code.
    Scheme: PBS or RPBS scheme
    Month: Time period (month).
    NSW, VIC, QLD, SA, WA, TAS, ACT, NT: Number of services provided in each state/territory.
    Total: Total number of services provided across all states/territories.

## Notes
    This script can be really slow, this is due to slow response time from SAS backend of Services Australia's server
    Infrequently, the server doesn't respond in time and misses one or two batches, you can individually download the one or two missing batches using the function in "redownload batch and merge.R"
    The script includes a delay between batches to avoid overloading the server.
    Error handling is implemented to catch and report any issues during the scraping process.
    Ensure you have a stable internet connection.
    The structure of the Medicare Statistics website may change, which could break the script. If the script stops working, you may need to update the HTML node selectors.
    The script is designed to handle up to 30 items per batch, this is to comply with the website's limitations.
    The script filters out rows containing "Total" and "Month" in the month column, and rows containing non numeric characters in the Item column.
    The script converts the service columns to numeric, removing commas.
    The script creates empty dataframes, if there is no data in the table, to ensure the script does not stop running.

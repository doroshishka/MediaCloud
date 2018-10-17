# MediaCloud - searching for relevant articles using outlet/keywords/time period and scraping content of the searched articles
This repository contains codes for Python and R to scrape content of news articles using Media Cloud platform (https://mediacloud.org). First, you have to generate a csv file with URLs from MediaCloud based on your search criteria using Python code. To get the content of the articles, you can use the following R code.

## Python code - getting URLs of the articles.
```
import datetime
from datetime import datetime as dt
import mediacloud
import numpy
import pandas as pd
import xlsxwriter

api_key = '12bd72aca75cd3cebaaf4a51bace549357b268584984514baecdfa7d29d0020d'
outlets = ['New York Times', 'Washington Post', 'Wall Street Journal']
start_date = [2012,1,1]
end_date = [2014,12,31]
key_word = ['second AND amendment', '2nd AND amendment', 'gun AND rights', '(National AND rifle AND association) OR (nra)', 'gun AND control', 'gun AND laws','background AND check', 'gun AND regulation', 'mass AND shooting','gun AND violence']
```
###### Input the api_key and call the api.
```
mc = mediacloud.api.MediaCloud(api_key)

def datelist(beginDate, endDate):
    # beginDate, endDate
    date_l=[dt.strftime(x,'%Y-%m-%d') for x in list(pd.date_range(start=beginDate, end=endDate))]
    return date_l
```
###### Build time_list(day_list) and month_list.
```
time_list = datelist(datetime.date(start_date[0],start_date[1],start_date[2]),datetime.date(end_date[0],end_date[1],end_date[2]))
month_list = []
for t in range(start_date[0],end_date[0]+1):
    for x in range(1, 13):
        if x < 10:
            month_list.append(str(t)+'0' + str(x))
        else:
            month_list.append(str(t) + str(x))
```
###### Get the media_id of each outlet.
```
outlets_id = []
for outlet in outlets:
    l = len(outlets_id)
    relevant_outlet = mc.mediaList(name_like=outlet)
    for x in relevant_outlet:
        if x['name'] == outlet:
            outlets_id.append(x['media_id'])
    l2 = len(outlets_id)
    if l == l2:
        print ('We can not find '+outlet+', what we can get:')
        for x in relevant_outlet:
            print (x['name'])
```						
###### 5 parameters of the function. The first one is the start date and second one is the end_date. Outlet_id is a factor of the list outlets_is.
```
```
def store(start_date,end_date,outlet_id,keyword_list,monthornot):
    if monthornot == 1:
        workbook = xlsxwriter.Workbook(str(outlets[outlets_id.index(outlet_id)]) + '  month.xlsx')
        worksheet = workbook.add_worksheet()
        w = 0
        for x in range(0, len(keyword_list)):
            worksheet.write(0, x + 1, keyword_list[x])
        for i in range(0, len(month_list)):
            worksheet.write(i + 1, 0 + w, month_list[i])
        for word in keyword_list:
            month = {}
            for x in month_list:
                month[x] = 0
            story = mc.storyPublicList(word, wc=True, solr_filter=[
                mc.publish_date_query(datetime.date(start_date[0], start_date[1], start_date[2]),
                                      datetime.date(end_date[0], end_date[1], end_date[2])),
                'media_id:' + str(outlet_id)], rows=100000)
            for s in story:
                pd = s['publish_date'].split('-')[0] + s['publish_date'].split('-')[1]
                month[pd] += 1
            for i in range(0, len(month_list)):
                worksheet.write(i + 1, 1 + w, month[month_list[i]])
            w += 1
        workbook.close()
    else:
        workbook = xlsxwriter.Workbook(str(outlets[outlets_id.index(outlet_id)]) + '.xlsx')
        worksheet = workbook.add_worksheet()
        w = 0
        for x in range(0, len(keyword_list)):
            worksheet.write(0, x + 1, keyword_list[x])
        for i in range(0, len(time_list)):
            worksheet.write(i+1, 0 + w, time_list[i])
        for word in keyword_list:
            time = {}
            for x in time_list:
                time[x] = 0
            story = mc.storyPublicList(word, wc=True, solr_filter=[
                mc.publish_date_query(datetime.date(start_date[0], start_date[1], start_date[2]),
                                      datetime.date(end_date[0], end_date[1], end_date[2])),
                'media_id:' + str(outlet_id)], rows=100000)
            for s in story:
                pd = s['publish_date'].split(' ')[0]
                time[pd] += 1
            for i in range(0,len(time_list)):
                worksheet.write(i+1,1+w,time[time_list[i]])
            w += 1
        workbook.close()
    return

#

def main():

    # This command should output a file named New York Times.xlsx
    store([2012,1,1],[2012,1,2],outlets_id[0],key_word,0)

    # This command should output a file named Washington Post month.xlsx
    store([2014,12,1],[2014,12,2],outlets_id[1],key_word,1)

    return

if __name__ == '__main__':
    main()
```    
## R code - scraping the content of the articles.
```
library(dplyr)
library(stringr)
library(RcppRoll)
library(boilerpipeR)
library(rvest)
install.packages("rJava")
library (rJava)

getwd()
setwd("address of the derictory")
```
###### Reading file with URLs
```
storyURL <- read.csv("TheHill.csv", header = TRUE, sep = ",") # SCSV of websites go here
storyChar <- matrix(as.character(storyURL[,4]))
num <- length(storyURL[[1]]) 
num
```
###### Creating a list of URLs
```
xml <- try(apply(storyChar, 1, read_html))
```
###### Scraping articles' content (note - check html code for the relevant content first!)
```
textScraper <- function(x) {
  as.character(html_text(html_nodes (x, ".content-wrapper") %>% html_nodes("p"))) %>%
    str_replace_all("[\n]", "") %>%
    str_replace_all("    ", "") %>%
    str_replace_all("[\t]", "") %>%
    paste(collapse = '')} 
```
###### Exporting text
```
articleText <- lapply(xml, textScraper) #list of article text
head(articleText)
```
###### Scraping articles' creation dates (note - check html code for the relevant content first!)
```
timeScraper <- function(x) {
  timestampHold <- as.character(html_text(html_nodes(x, ".submitted-date"))) %>% str_replace_all("[\n]", "")
  matrix(unlist(timestampHold))
  timestampHold[1]} 
```
###### Exporting dates
```
timestamp <- lapply(xml, timeScraper) #list of timestamps
head(timestamp)
```
###### Creating a final file with story ID, headline, text, date, and themes (as defined by MediaCloud) for each article
```
articleDF <- data.frame(storyID = as.character(storyURL[,1]), 
                        headline = as.character(storyURL[,3]), 
                        matrix(unlist(articleText), nrow = num), 
                        matrix(unlist(timestamp), nrow = num), 
                        themes = as.character(storyURL[,7]), 
                        stringAsFactors = FALSE)
articleDF$stringAsFactors <- NULL
names(articleDF)[3] <- 'text'
names(articleDF)[4] <- 'time'
```
###### Writing the file
```
write.csv(articleDF, file = "TheHill txt.csv") #exports a csv of your text
```





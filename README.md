# MediaCloud
This repository contains codes for R and Python to scrape content of news articles using Media Cloud platform.

library(dplyr)
library(stringr)
library(RcppRoll)
library(boilerpipeR)
library(rvest)
install.packages("rJava")
library (rJava)

getwd()
setwd("address of the derictory")

storyURL <- read.csv("TheHill.csv", header = TRUE, sep = ",") # SCSV of websites go here
storyChar <- matrix(as.character(storyURL[,4]))
num <- length(storyURL[[1]]) 
num

xml <- try(apply(storyChar, 1, read_html))

textScraper <- function(x) {
  as.character(html_text(html_nodes (x, ".content-wrapper") %>% html_nodes("p"))) %>%
    str_replace_all("[\n]", "") %>%
    str_replace_all("    ", "") %>%
    str_replace_all("[\t]", "") %>%
    paste(collapse = '')} 

articleText <- lapply(xml, textScraper) #list of article text
head(articleText)

timeScraper <- function(x) {
  timestampHold <- as.character(html_text(html_nodes(x, ".submitted-date"))) %>% str_replace_all("[\n]", "")
  matrix(unlist(timestampHold))
  timestampHold[1]} 

timestamp <- lapply(xml, timeScraper) #list of timestamps
head(timestamp)

articleDF <- data.frame(storyID = as.character(storyURL[,1]), 
                        headline = as.character(storyURL[,3]), 
                        matrix(unlist(articleText), nrow = num), 
                        matrix(unlist(timestamp), nrow = num), 
                        themes = as.character(storyURL[,7]), 
                        stringAsFactors = FALSE)
articleDF$stringAsFactors <- NULL
names(articleDF)[3] <- 'text'
names(articleDF)[4] <- 'time'

write.csv(articleDF, file = "TheHill txt.csv") #exports a csv of your text

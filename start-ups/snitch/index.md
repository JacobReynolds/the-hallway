---
title: snitch
description: the government is always watching
order: -12
icon: squirrel
---

# 🤫 Snitch

A [recent SEC ruling](https://www.sec.gov/news/press-release/2023-139) now requires companies to file an 8-K form declaring they have experienced a data breach within 4 business days of discovering it happened. This is huge and likely a big PITA for a lot of companies. Researching what happened and properly communicating can take a lot of time, but unfortunately a lot of companies abused this to try and sweep the breach under the rug. The SEC is having none of it. I had the idea that it'd be nice to build a notification service so you can be notified every time an Item 1.05 8-K filing is made.

## Thanks Obama

Luckily it's the government, and in _some_(ish) cases their data is public. You can search all public SEC filings via [EDGAR](https://www.sec.gov/edgar/searchedgar/companysearch), the Electronic Data Gathering, Analysis, and Retrieval system. You can go even deeper by using their [full-text search](https://www.sec.gov/edgar/search/#) to dive into all the filings.

Unfortunately an 8-K is a common form used to notify investors and the SEC about a variety of things, only one of those items being a data breach. Searching EDGAR for [8-K filings](https://www.sec.gov/edgar/search/#/category=custom&forms=8-K) returns a ton of documents. Mostly about changes in executive staffing or compensation. But thanks to their, likely [lucene](https://lucene.apache.org) backend, we can add a full text match for the words "Item 1.05" and get [more specific results](https://www.sec.gov/edgar/search/#/q=%2522item%25201.05%2522&category=custom&forms=8-K). At the time of writing this, there are 5 results across 3 companies. When I originally performed this research there was only 1 filing. Because of that, I wasn't sure how often new filings would come out, and how they would differ in formats so it wasn't an appropriate time to start writing automation around it. However as more filings get created and we can start to measure their velocity, this could be a fun data feed or subscription service to provide those that like others' dirty cyber laundry.

## Further thoughts

Reading the SEC press release more closely, I now realize it also requires annual filings by all publicly traded companies to make a statement about their cybersecurity processes. Although reading some of them, they all seem to be written by a lawyer and lack any substantial information.

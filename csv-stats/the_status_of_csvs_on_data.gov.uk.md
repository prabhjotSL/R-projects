# What is a CSV? A case study of CSV files on data.gov.uk


## Abstract

CSV stands for *comma-separated values*. It is a simple format for tabular data and relatively easy to process. We analysed more than 20,000 links to CSV files on data.gov.uk – only around one third turned out to be machine-readable. Around 4,000 were other formats, another 4,000 are no longer available and the rest did not conform to a minimal standard such as a header row in the first line. A typical CSV is between 1kb-1mb in size and has around eight columns.

Our analysis of the header names of 7,390 machine-readable CSVs suggests that there are clear patterns. The most common type are spend records, usually split by month and type. Other organisations such as the Crown Prosecution Service also release vast amounts of CSVs. If publishers embrace standards for CSV data releases, we see a great potential such as aggregate analyses.



## Metadata

In January 2014 we created a file that contains all CSVs published on [data.gov.uk](http://data.gov.uk). We searched for all CSVs and then grabbed [the accompanying Atom feed](http://data.gov.uk/feeds/custom.atom?res_format=CSV). We then looped through all the results, and dumped the data we needed into a CSV file (the [Ruby code](https://github.com/theodi/csv_finder) and [resulting CSV](http://theodi.github.io/csv_finder/csv_urls.csv) is on Github).

The result is a simple file that mainly lists all the URLs on data.gov.uk that are classified as data in a CSV format. 
 
<table class="table offers table-horizontally-condensed">
  <tr>
    <td>dataset</td>
    <td>Usually title and publisher</td>
  </tr>
  <tr>
    <td>url</td>
    <td>The URL of the dataset</td>
  </tr>
  <tr>
    <td>description</td>
    <td>More information, usually date published</td>
  </tr>
  <tr>
    <td>size</td>
    <td>Size of the file in bytes</td>
  </tr>
</table>

##### Figure 1. Number of valid CSVs on data.gov.uk
![overall-stats](https://raw.github.com/theodi/R-projects/master/csv-stats/graphics/overall-stats.png)

Figure 1 shows how the overall number of 20,692 dwindles to 7,390:

1. Almost 4,000 URLs are not CSVs and link to other files, web pages or resolve in some download link. Some of them may be valid, but many links don't work, for example [this URL](http://www.roh.nhs.uk/EasySiteWeb/GatewayLink.aspx?alId=1763).

2. Of the URLs that do end in `.csv`, around 4,000 yield an error. For example, some files that detail the [Crossrail Spend 2013](http://www.crossrail.co.uk/assets/library/document/c/original/crossrail_payments_period_13_2012-13.csv) are no longer available. 

3. Even if the CSV exists, it loses some of its value if we cannot read it automatically. Standards are hence important. An example of what did **not** get parsed is below. The first line is the title and the second one is empty (from [here](http://www.royalwolverhamptonhospitals.nhs.uk/files/mth%206%20september%202013%20(3).csv)).

   ![example](https://raw.github.com/theodi/R-projects/master/csv-stats/graphics/miss-header-example.png)


There are some limitations. For instance, it is hard to replicate the exact numbers as a website may be temporarily offline. We verified "machine-readable" (and excluded ones) in several ways, but it is likely that we still have false positives and vice versa. As a side note, of all the CSVs listed on data.gov.uk, 3169 (19%) are served over a secure connection, i.e. `https`.

The whole analysis and its code is on [GitHub](https://github.com/theodi/R-projects/blob/master/csv-stats/csv-meta-analysis-data-gov.R).

## A CSV is not an Excel sheet with a different extension

What is going on? One simple explanation is that data.gov.uk combines many different publishers. All  of them have their own quality and update schedule. In fact, we suspect most of them have not implemented an automated process that would make publication of CSVs easier and more reliable. 

We can summarise the problems by recognising the source of many CSV files: **often a direct copy of an Excel sheet** (or, in rare cases, an alternative tool). Excel sheets are optimised to be read by humans. They usually provide rich metadata, related information, and nice formatting. However, Excel files are difficult to process because they are unpredictable and possibly unique. Therefore, saving a `xlsx` file with a `csv` extension, as we observe, cannot be the solution. It is, in fact, the reason for many of the issues we encountered. 

There are numerous problems that prohibit importing, or even reading, a CSV file with a machine. 

1. Not available, for example because the link changed or the website is down.
2. Errors that may be circumvented such as SSL certificate warnings.
3. Non-standard symbols that are not recognised. For example, an `invalid multibyte string` or erroneous line ending.
4. Files where the header row is not in line 1 or not specified. More in the next section.
5. Multiple header rows
6. Multiple tables in one file
7. And many more.


## Size 

The vast majority of CSV files is between 1kb and 1mb in size. The largest file on data.gov.uk is at the moment the complete data of the Land Registry Price Paid Data with 3.2gb. 

##### Figure 2. Histogram of the size of CSVs
![size-histogram](https://raw.github.com/theodi/R-projects/master/csv-stats/graphics/histogram-size-of-csvs.png)


## Automatically recognising a header row

A header row in a CSV includes short descriptive names of the data underneath. The header is paramount to get a minimal understanding of the data. Unfortunately, if the header is not in line 1, the machine has to guess where else it might be. This may be a simple task if the first few rows are empty. In many cases, however, the first few lines contain metadata such as the title or the source. Some otherwise genuine CSVs have multiple headers. In our analysis we only analysed those files that are already machine-readable. 

The following steps create an algorithm that has a good chance of recognising the header row:

1. Find the maximum number of columns for any row (`max.no.col`)
1. Discard all, if any, empty rows from the beginning to the first non-empty row (`firstrow`) 
1. Test whether `firstrow` has an equal number compared to `max.no.col`
1. If the test fails, discard `firstrow` and go to step 2.
1. If the test passes, use `firstrow` as header row. 

A more sophisticated algorithm would inspect the last header. If its length is very long (we set it at > 100 characters) it is likely to be something else mistaken as a CSV. To recognise a case of multiple header rows, it may be promising to count the number of missing header names.

Moreover, it is a good idea to check whether the CSV contains more than one table, for example by looking for a repeated header row.


## Header names

After much experimentation we automatically read 7,390 CSV-files. All of them have header names, of course, that can be analysed. For example, a typical CSV file on data.gov.uk has eight headers, i.e. columns (see figure 3).

There is also a peculiar spike at 41 headers per dataset. Some of the datasets have all the information arranged in only one row with many different columns. The prevalent theme, in this case it seems, are various ways of counting payroll staff.

##### Figure 3. Histogram of the number of headers (columns)
![header-length](https://raw.github.com/theodi/R-projects/master/csv-stats/graphics/header-length-histogram.png)

What are the most popular header names? We had to clean up plenty of names because in its raw format, it is *messy data* such as "amount", " amount", "AMUONT" etc. After some [Open Refine](http://openrefine.org) magic, we produced the following rank table. 

<table class="table offers table-horizontally-condensed">
 <tr><td>Expense Type</td><td>3,144</td></tr>
 <tr><td>Entity</td><td>3,039</td></tr>
 <tr><td>Expense Area</td><td>3,029</td></tr>
 <tr><td>Supplier</td><td>2,889</td></tr>
 <tr><td>Date</td><td>2,820</td></tr>
 <tr><td>Transaction Number</td><td>2,791</td></tr>
 <tr><td>Amount</td><td>2,734</td></tr>
 <tr><td>Department Family</td><td>2,498</td></tr>
 <tr><td>VAT Registration Number</td><td>916</td></tr>
 <tr><td>Convictions Percentage</td><td>836</td></tr>
</table>

The headers may not be representative overall because it only features the machine-readable CSVs. They are frequency counts which also implies that departments that release similar data in individual CSVs (e.g. for each month) are overrepresented. Be that as it may, the header ranking shows how common certain data types are.

##### Figure 4. Co-occurrence of header names
![header-map](https://raw.github.com/theodi/R-projects/master/csv-stats/co-occurrence/top-headers-coocc.png)

We took the 50 most popular header names and calculated how often they each appear together. The lines' thickness stands for the frequency of their co-occurence. For example, as we would expect, *Payscale Minimum* and *Payscale Maximum* are next to each other. A generic header name such as *Unit* appears in many files and has therefore stronger links to various others.

The most common cluster is around *Expense Type* because many files document government spending. A typical "Spend over £25,000" CSV is very likely to have similar headers as listed in the previous table. 

We also see an independent cluster around prosecutions. For example, the Crown Prosecution Service releases a lot of individual CSVs, which increases how many times they appear in our header analysis. 

##### Figure 5. Co-occurrence of header names excluding popular datasets
![header-map2](https://raw.github.com/theodi/R-projects/master/csv-stats/co-occurrence/top-headers-coocc-excl-popular.png)

Figure 5 shows the co-occurrence of headers excluding popular datasets. For example, CSVs with *Expense Type* no longer appear. Here the 200 most popular header names reveal quite a tangle. The large, overplotted area on the bottom are CSV headers, among others, from the BIS Workforce Management Information.

The big cluster on the right, dubbed the "Death Star", comes from one series of CSVs. The [Foreign Office Consular](http://data.gov.uk/dataset/foreign-office-consular-data) publishes "data on assistance provided to British nationals worldwide this includes numbers of incidents handled by UK consulates overseas, including hospitalisations, deaths, detentions, with geographical location". 

A similar cluster appears on the top left published by the [Metoffice](http://data.gov.uk/dataset/latest-marine-observational-data). It includes the latest 24 hours marine data for observed weather. Both exemplify why some headers are more related than others: a publisher may release many almost identical files split by time or geography.







## Concluding remarks

We appreciate that more and more people recognise CSV as a desirable format for sharing data on the web. However, publishing any tabular data simply with a `.csv` extension will not bring as much further. The key is to follow a minimal standard. Tools for doing so are emerging, for instance the ODI's [CSV validator](http://csvlint.io/). As Jeni wrote, 2014 may turn out to be the [Year of the CSV](http://theodi.org/blog/2014-the-year-of-csv).

Our analysis is far from exhaustive relative to the open data ecosystem; it only looks at data.gov.uk. Even there we have to acknowledge certain limitations such as incorrect headers or temporarily unavailable URLs. What it shows us, however, is that few publishers follow leading practice yet. 

By making access and aggregation of data via CSV easier, we enable a huge potential. For example, only an automated analysis may be able to look at the spending across all government departments and bring the often proclaimed transparency. Companies may use CSVs to integrate them into their dashboards or to build  services. Even non-technical citizens may be able to use the data more easily. Many user-friendly tools such as [Datawrapper](http://datawrapper.de/) have CSV features built-in. 

Let's aim for a world with less [data munging](http://en.wikipedia.org/wiki/Data_wrangling).





 

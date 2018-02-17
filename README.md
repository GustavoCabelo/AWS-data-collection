# AWS-data-collection

This respository includes the methodology used to gather data from AWS.

# The AWS services employed:
- Elastic Compute Cloud (EC2): web-based service that enables you to run application programs in the Amazon computing environment. 

- Elastic MapReduce (EMR): web service that enables you to easily and cost-effectively process vast amounts of data. It utilizes a hosted Hadoop framework running on the web-scale infrastructure of Amazon Elastic Compute Cloud (Amazon EC2) and Amazon Simple Storage Service (Amazon S3). This is where you will process the Common Crawl data to develop the large data matrix.

- Simple Storage Service (S3): is storage designed to make web-scale computing easier for developers and provides a simple web services interface that can be used to store and retrieve any amount of data, at any time, from anywhere on the web. This is a general platform that stores all the data (input and output) for the EC2 and EMR services.

# Data Processing
The data will come from Common Crawl - open repository of web crawl data (over 5 billion pages so far) that is stored on Amazon’s Public Data Sets - and processed through "map-reducing" and stored in S3 buckets.

## What Data to Collect?
Scan the Common Crawl for documents that express meaningful sentiment about one of the Keywords (phones).
Then, for each relevant document, collect information about the sentiment toward key features of the phone (camera, display, performance, and operating system). 

### Calculate the relevancy of the document and the sentiment toward phone features, 
Look for and count relevant words: 
  - For document relevancy: look for references to the device itself and references to words that indicate the document contains a meaningful view of the device. 

  - For sentiment toward each feature: look for references to the feature itself and references to positive, negative, and neutral words. Documents are retained only if they have at least one mention of a phone or phone OS, and at least one of the following terms is present in the document: review; critique; looks at; in depth; analysis; evaluate; evaluation; assess.
  
## Approach  
Execute scripts in Python to run several map and reduce jobs on Common Crawl data using EMR.

These EMR jobs run multiple instances of the mapper Python script *(Mapper.py)* on multiple servers (nodes). 

Each of the mapper instances processes a subset of the overall set of Common Crawl files for the job. When a mapper completes it sends its results to a reducer process *(Reducer.py)*. 

The reducer process combines the results from some number of mapper processes, then writes the combined results out to the S3 location. 

When a mapper or reducer completes, another process starts up in its place if there are still files to process. The data from the job flows is in comma separated value format (.csv) and could be opened in Excel or a text editor. 

However, since the results are split into one folder per step and one file per reducer, I run a Python script *(ConcatenateFiles.py)* and a .csv file *(MatrixHeader.csv)* that combines everything together and adds in a header row.

# Objective
Collect websites that contains the relevant keywords from 100 Common Crawl files.

# AWS-data-collection

This respository includes the methodology used to gather data from AWS.

# Objective
Collect around 10k websites that contains the relevant keywords from 100 Common Crawl files.

# 1. The AWS services employed:
- Elastic Compute Cloud (EC2): web-based service that enables you to run application programs in the Amazon computing environment. 

- Elastic MapReduce (EMR): web service that enables you to easily and cost-effectively process vast amounts of data. It utilizes a hosted Hadoop framework running on the web-scale infrastructure of Amazon Elastic Compute Cloud (Amazon EC2) and Amazon Simple Storage Service (Amazon S3). This is where you will process the Common Crawl data to develop the large data matrix.

- Simple Storage Service (S3): is storage designed to make web-scale computing easier for developers and provides a simple web services interface that can be used to store and retrieve any amount of data, at any time, from anywhere on the web. This is a general platform that stores all the data (input and output) for the EC2 and EMR services.

# 2. Data Processing
The data will come from Common Crawl - open repository of web crawl data (over 5 billion pages so far) that is stored on Amazon’s Public Data Sets - and processed through "map-reducing" and stored in S3 buckets.

## 2.1 Approach  
Execute scripts in Python to run several map and reduce jobs on Common Crawl data using EMR.

These EMR jobs run multiple instances of the mapper Python script *(Mapper.py)* on multiple servers (nodes). 

Each of the mapper instances processes a subset of the overall set of Common Crawl files for the job. When a mapper completes it sends its results to a reducer process *(Reducer.py)*. 

The reducer process combines the results from some number of mapper processes, then writes the combined results out to the S3 location. 

When a mapper or reducer completes, another process starts up in its place if there are still files to process. The data from the job flows is in comma separated value format (.csv) and could be opened in Excel or a text editor. 

However, since the results are split into one folder per step and one file per reducer, I run a Python script *(ConcatenateFiles.py)* and a .csv file *(MatrixHeader.csv)* that combines everything together and adds in a header row.

## 2.2 What Data to Collect?
Scan the Common Crawl for documents that express meaningful sentiment about one of the Keywords (phones).
Then, for each relevant document, collect information about the sentiment toward key features of the phone (camera, display, performance, and operating system). 

### 2.2.1 Calculate the relevancy of the document and the sentiment toward phone features, 
Look for and count relevant words: 
  - For document relevancy: look for references to the device itself and references to words that indicate the document contains a meaningful view of the device. 

  - For sentiment toward each feature: look for references to the feature itself and references to positive, negative, and neutral words. Documents are retained only if they have at least one mention of a phone or phone OS, and at least one of the following terms is present in the document: review; critique; looks at; in depth; analysis; evaluate; evaluation; assess.

## 2.3 Collecting Data

The EMR console provides an easy-to-use graphical interface for launching and monitoring your job flows directly from a web browser, but you may find that it is not efficient enough when running large jobs. In this case, you will need to use the command line interface. This interface will be accessed from the 'terminal' in Mac OSX and 'Command Prompt' on a Windows machine.

The Command Line Interface (CLI) in AWS provide the ability to programmatically launch and monitor progress of running job flows, to create additional custom functionality around job flows (such as sequences with multiple processing steps, scheduling, workflow, or monitoring). You should have already installed the CLI. 

Below are the Amazon Command Line Interface steps for running 100 WET files needed for your sentiment analysis. The steps will cover 3 types of files:

- BDF File – This file contains s3 addresses for searching WET files
- CreateJson Python File – Running this file uses the s3 addresses from the BDF file and creates JSON files
- JSON File – This file’s information will be “uploaded” through CLI to create the EMR cluster steps for each of the s3 addresses from your BDF File.

Prior to running this job, ensure that you have the AWS CLI installed and that credentials are configured correctly. Check in the internet if not.

 

### a) Select 100 WET file addresses. 
Open the WET paths file with a text editor and choose 100 addresses. The WET files are not indexed by topic so any 100 addresses can be used.
- Copy your 100 addresses and paste them into a new tab of your text editor
- Save this new tab as a .bdf file
- Add the S3 address snippet to the beginning of each your addresses. See step two if you need a reminder:     s3://commoncrawl/
- See the sample .bdf file in the Resources if you need help

### b) Copy the CreateJsonFiles.py Python script into the folder with the .bdf file and personalize the script. This script will generate a .json file from your .bdf file. *You must make updates to this python script with a text editor like Sublime before you run it.

- Update the python script to have the correct S3 locations for your Mapper.py and Reducer.py files – see files parameter on line 17
- Update the python script to have the correct S3 address for your output bucket – see output parameter on line 18
- Review all lines of script to assure that the mapper and reducer file names are correct
- Save the file

### c) Run the CreateJsonFiles.py Python File from the command shell to generate your json file. 

Open the ‘terminal’ (OSX) or ‘Command Prompt’ (windows) command shell. In your command shell, use the change directory command to point to the folder containing your BDF and CreateJSON files. If you don’t know how to change directories, spend a few minutes researching google and youtube for instructions. Now run the CreateJSON file from command line. The command format is below. You will be asked for the input .bdf file’s name and a name for the output file.
- python    filenameyouwanttorun.py      

At this point, you should have a .json file containing all of the appropriate markup needed by AWS EMR. The .json files will contain one step per WET address from the .bdf file.

### d) Checking the validity of your .json file.

In order for the CLI to correctly process .json files the .json file has to be formatted corrected or be structured in the correct manner. Thankfully there are numerous online validation tools that we can use to check the validity of JSON files.

- In order to check the structure of your .json file open it in Sublime and copy/paste the text into JSONLint. If your file structure is valid the site will inform you of such; if not, you might need to modify the structure of your .json file so the CLI will process it correctly.
- After you make any necessary modifications to your file structure copy the text out of JSONLint and paste it back into Sublime and save it as a .json file - you can either overwrite your original file or save it as a new one - just remember which one is valid!

### e) Run the the .json files from CLI to create a EMR Cluster. 

Running a json file from CLI will initiate the creation of a new AWS EMR cluster that will search for the desired sentiment data. You can monitor the progress of your cluster from the EMR Console. You may run only one cluster at a time. 

Below is an AWS command to run the JSON file (run it from the Terminal (Mac) or Command Prompt (Windows)); change the following areas (shown as XXXX):
------
"aws emr create-cluster --name "cluster_name" --ec2-attributes SubnetId=subnet-XXXXX --emr-version 5.11.0 --auto-terminate --log-uri s3://XXXXX --use-default-roles --enable-debugging --instance-groups InstanceGroupType=MASTER,InstanceCount=1,InstanceType=c4.large InstanceGroupType=CORE,InstanceCount=3,InstanceType=c4.large --steps file://XXXXX.json"
------
- Cluster Name
- subnetID: SubnetId can be found by visiting one of your recent, successful clusters using Console. Look under Network and Hardware. You should see something with a structure similar to: subnet-61e40522
- json file name
- log-uri: This s3 address should point to your debugging bucket

Note: The AWS CLI script can be different for Windows users: http://docs.aws.amazon.com/ElasticMapReduce/latest/DeveloperGuide/add-steps-cli-console.html 

### Monitor your Cluster from AWS Console. Running 100 WET files can be a lengthy process and can take up to 2 days. From the EMR console, open the cluster you created. Scroll down the page and click on Steps. You should see a step for each of the WET addresses. You may have an occasional failed step, this is expected and OK. If all of your steps are failing you should terminate the cluster and check the Log files to understand the failures. 


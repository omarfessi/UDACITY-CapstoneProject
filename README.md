# UDACITY DEND - CAPSTONE PROJECT

## Introduction 
The project is the final part of my journey along with Udacity Data Engineering Nanodegree program.
I have acquired a lot through this experience, I'll try to briefly encapsulate all logistics and pre-requisites to make you try this project on your own.


## Dataset
Udacity gives students the option to use their suggested project or pick one dataset and scope it by themselves. In my case I went for the second option. 
Gathering the right data is one of the most important task for data engineers and I can tell you finding the suited data to work with took me about 80-90% of the project's time.
I finally picked the AirBnB dataset of Amsterdam which I found  [here](https://www.kaggle.com/erikbruin/airbnb-amsterdam) on Kaggle.
I must give credits to Erik Bruin whose [Notebook](https://www.kaggle.com/erikbruin) inspired me a lot ! 
The [data](https://www.kaggle.com/erikbruin/airbnb-amsterdam) from kaggle consists of plain data in csv format, however as the Udacity requires at least two different files format, I choosed one file to ingest it as a json format.
All input files are located in my public S3 bucket which I'll specify its path in the Spark read commands later in code.
  - calendar.csv ( 7,310,950 records)  
  - listings.csv (20,048 records)
  - listings_details.csv (20,048 records)
  - reviews_details.json(431,830 records)
  
## Project Scope
The purpose of the project is to build an ETL pipeline that reads data from Amazon S3, processes it with Spark in order to create dimensions and facts, loads data back to the data lake (Amazon S3), later copy transformed data into Redshift to run Quality Checks and do Analysis as it is the goal of an etl pipeline.
The outcome is a set of tables linked to each other (snowflake schema)to make it easier for BI applications to make use of the data ( sql is always easier to run analytics than other frameworks).

## Data model
#### ERD
The data model includes seven tables, being five of them dimensions and two facts.
![ERD-Amsterdam-AirBnB](https://user-images.githubusercontent.com/47854692/105017008-9db31500-5a43-11eb-9f33-37bcef0af1f5.png)

#### Data Dictionary 

`date_time` : dimension table about the date-time

| Column's name | type | example | 
| --- | --- | --- |
| date  | date `PRIMARY KEY` | (2019, 7, 30) |
| day_of_month | integer | 30 |
| week | integer | 1 |
| month | integer | 7 |
| year | integer | 2019 |
| day | varchar | 'Tuesday' |


`location` : dimension table about hosts

| Column's name | type | example | 
| --- | --- | --- |
| location_id  | integer `PRIMARY KEY`| 33 |
| latitude | float | 52.296070098877 |
| longitude | float | 4.99365091323853 |
| street | varchar | 'Amsterdam-Zuidoost, Noord-Holland, Netherlands' |
| zipcode | varchar | '1107' |
| city | varchar | 'Amsterdam-Zuidoost |
| country | varchar | 'Netherlands' |


`hosts` : dimension table about hosts

| Column's name | type | example | 
| --- | --- | --- |
| host_id  | integer `PRIMARY KEY`| 3552086 | 
| host_name | varchar | 'Dennie' |
| host_about | varchar | "World traveller, born and raised in the hospitable South of the Netherlands, but nowadays at home in one of Amsterdam's most famous neighborhoods: de Jordaan. I work in HR for a sportswear brand, and am interested in people, places, music, history and politics; my sport is running.\r\nAn AirBnB user for many years, my better half Elske and I are happy to share a room in our recently renovated home with travelers interested in our lovely city of Amsterdam. We hope to meet you soon!" |
| host_response_time | varchar | 'within an hour' |
| host_response_rate | float | 100.0 |
| host_is_superhost | boolean | True |
| host_location | varchar | 'Amsterdam, Noord-Holland, The Netherlands' |
| host_identity_verified | boolean | True |


`reviewers` : dimension table about reviewers

| Column's name | type | example |
| --- | --- | --- |
| reviewer_id  | integer `PRIMARY KEY`| 99431047 | 
| reviewer_name | varchar |'Madeleine' | 


`listings` : dimension table about listings

| Column's name | type | example |  
| --- | --- | --- |
| id  | integer `PRIMARY KEY`  | 26246938 |
| host_id  | integer  `FOREIGN KEY` references to `host_id` in hosts table| 2945351 |
| location_id  | integer `FOREIGN KEY` references to `location_id` in location table| 926 |
| price | float | 135.0 |
| weekly_price | float | None| 
| monthly_price  | float |  None |
| security_deposit | float | 250.0 |
| cleaning_fee | float |  60.0 |
| guests_included | float | 2.0 |
| extra_people| float | 0.0 |
| listing_url | varchar | 'https://www.airbnb.com/rooms/26246938' |
| property_type | varchar | 'Apartment' |
| accommodates | integer | 2 |
| room_type | varchar | 'Entire home/apt' |
| bathrooms | float | 1.5 |
| bedrooms | float | 3.0 |
| beds | float | 3.0|
| amenities | varchar | '{TV,Wifi,Kitchen,"Paid parking off premises",Heating,Washer,Dryer,"Smoke detector","Fire extinguisher",Essentials,Hangers,"Hair dryer",Iron,"Outlet covers",Bathtub,"Baby bath","Changing table","High chair","Stair gates","Children’s books and toys","Babysitter recommendations",Crib,"Room-darkening shades","Children’s dinnerware","Bed linens",Microwave,"Coffee maker",Refrigerator,Dishwasher,"Dishes and silverware","Cooking basics",Oven,Stove,"Patio or balcony","Garden or backyard","Wide clearance to bed"}' |
| bed_type | varchar | 'Real Bed' |   
| review_scores_rating| float | 90.0 |
| review_scores_accuracy | float | 9.0 |
| review_scores_cleanliness | float | 8.0 |
| review_scores_location | float | 9.0 |
| review_scores_value | float | 8.0 |
| cancellation_policy | varchar | 'strict_14_with_grace_period' |
| require_guest_profile_picture | boolean | False | 
| require_guest_phone_verification | boolean | False |


`calendar` : fact table about calendar

| Column's name | type | example |
| --- | --- | --- |
| calendar_id | integer `PRIMARY KEY`| 7 |
| listing_id | integer `FOREIGN KEY` references to `id` in listings table| 2818 |
| date | date `FOREIGN KEY` references to `date` in date_time table| (2018, 12, 12) |
| available | boolean | True |
| price | float | 59.0) |

`reviews` : fact table about reviews

| Column's name | type | example |
| --- | --- | --- |
| id | integer `PRIMARY KEY`| 9131 |
| listing_id | integer `FOREIGN KEY` references to `id` in listings table| 2818 |
| date | date `FOREIGN KEY` references to `date` in date_time table| (2009, 9, 6) |
| reviewer_id | integer `FOREIGN KEY` references to `reviewer_id` in reviewers table | 26343 |
| comments | varchar | 'You can´t have a nicer start in Amsterdam. Daniel is such a great and welcoming host. The room was really light and charming, so well decorated. Daniel has a great sense of hospitality, he even helped with our luggage and gave us maps and a travelguide. He´s very open minded and helpful. We had a great time, we would stay with him again anytime we´re back in Amsterdam. Daniel made sure we had everything for a great trip, thanks again for the bikes! Hope to see you soon, Daniel!' |




## Technology and tools
The dataset is not yet considered as big data, it can be processed with pandas, however the running time would be longer than using a EMR cluster with spark running on top of it.
  - Amazon S3 as Data Lake for storing data before and after processing.
  - Pyspark to wrangle the data and build the data model.
  - Amazon Redshift as a Dataware house for querying data (structured): IaC to create a Cluster where Redshift will be running 
 

## Future proof scenario 
  -  If I had 100x times the size of the processed files I would still load the data into AWS S3, then use spark to do EDA, load it back to S3 and finally ETL into Redshift.
  -  I would set up an Airflow pipeline which runs Python scripts or Airflow operators, with a SLA to ensure the job runs by 7 am each day.
  -  Redshift is a good fit if 100 persons would need to access the data, it should be able to handle this with no problem. We could increase the specs of our cluster if it was not fast enough to serve everyone.

## Infrastruce and Configuration 
To laverage the power of Spark -which is basically a distributed processing system used for big data workloads- I set up an EMR cluster on Amazon Web Services to execute fast calculation via a cluster of machines.
Please refer to this [blog](https://towardsdatascience.com/how-to-create-and-run-an-emr-cluster-using-aws-cli-3a78977dc7f0) which details how to set up and lauch an EMR cluster.
In my case, here is my EMR configuration (you can skip the AWS CLI instruction with in the blog above and just use the AWS management console) : 
##### Software configuration
  - emr-5.20.0 release
  - Hadoop 2.8.5
  - JupyterHub 0.9.4
  - Hive 2.3.4
  - Spark 2.4

##### Hardware configuration
Note that this service is quite expensive, so do not forget to terminate the cluster after finishing the work, and there is no need to use high performant nodes 
In my case I used :
  - one single node ( Master node of type m5.xlarge with 16 Gib of memory) and no core nodes. It should work fine.

##### Security options
If you have followed instructions on the blog mentionned above, you should have a EC2 key pair created and downloaded in a safe location on your local machine. !! DO NOT save it in a github repo or public accessible file.
The EC2 key pair allows users to ssh to their distant machine.

## Project Execution Steps
  1  - Now you hit create cluster button and grap a cup of coffee ( it should be ready in about 5 minutes).
  2  - Once your cluster is in running state, you can create a notebook and choose to have it run on this cluster ( you should see the create button on the left menu ).
  3  - Your notebook should start within few seconds, open it in JupyterLab and upload the CAPSTONE-PROJECT.ipynb 

##### Notes : 
  - You may want to upload all the resources to your own S3 Bucket ( data files and CAPSTONE-PROJECT.ipynb ), feel free to try it, S3 is quite cheap.
  - You need to set up your own S3 where to output your processed files and change the file path in the notebook.
  - In order to use external libraries in your notebook, you can ssh to your cluster via AWS CLI and use DNS hostname which you'll find in the summary page of you emr cluster ( always refer to the blog above), do not forget the step where you edit your inbound rules in your security policy.
  - Once ready, issue some bash commands as follow to install pandas and seaborn ( do not forget that the notebook is run on a Pyspark kernel):
 ```sh
$ sudo easy_install-3.6 --upgrade pip
$ sudo /usr/local/bin/pip3 install -U pandas scikit-learn matplotlib  seaborn
sudo ln -sf /usr/bin/python3 /usr/bin/python
```
Once parquet files are sent to S3, go to your AWS management console and terminate the cluster, otherwise you will be charged for something you did not really use.
## Creating Redshift Cluster 
The second step of the project is to create the Redshift cluster to run quality checks and run diverse analyis. you find my cluster config in the cfg file.
You first must set up a new AWS IAM user with administrative rights. Go to the AWS console, then IAM, then Users, and create a new user with "AdministratorAccess". Download the credentials and set these as KEY and SECRET in the cfg file provided in this repo.
Download the "COPY-TO-REDSHIFT-FOR-ANALYSIS.ipynb" and the cfg file and put them in the same directory.
If you want, you can change the name, user, password in the cfg file, as well as the CLUSTER_IDENTIFIER, CLUSTER_TYPe, NUM_NODES, NODE_TYPE.
Now you should run the notebook ("COPY-TO-REDSHIFT-FOR-ANALYSIS.ipynb" ) in your local machine with no problem.
The cfg file contains the parquets file path for all 7 tables; please edit it with your own path.
In step1 in the notebook,if it is the first time you run the notebook, uncomment the code where you create a role that enables Redshift to access S3 buckets. then you comment it once again because that role should be always attached with the IAM user.
Now, finally execute the next "create a Redshift cluster" cell code and wait a moment, you should be seeing an "available status " in the description with in the next cell.
Once the cluster is ready copy paste the Endpoint adress value ( without quotes) in the blank Host field with in the cfg file.
Run the next cell code to create the connexion and enjoy the analysis.

DO not forget to clean up the resources, the 2 final code cells in the notebook should do the job.



Any suggestion or contribution request would be much appreciated. 










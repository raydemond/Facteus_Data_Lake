# Faceteus Data Lake

> The project helps a music streaming startup Sparkify to move their user and song datawarehouse to a data lake hosted on AWS S3. 
An ETL pipleline is built to extract data from S3, process them using Spark, and tranform them into dimensional tables back to S3 for analytic queries.

## Table of contents
* [Setup](#setup)
* [Data Lake](#data-warehouse)
* [ETL Pipeline](#etl-pipeline)
* [Instructions](#instructions)
* [Example Queries and Results](#example-queries-and-results)

## Setup
* etl.py to reads and processes files from song_data and log_data and loads them into your tables.
* dwh.cfg contains credentials configuration to connect to AWS account.

## Data Lake
The data lake contains one fact table and four dimensional tables based on a star schema.

### Project Datasets
Sparkify stores two datasets in S3:
```
Song data: s3://udacity-dend/song_data
Log data: s3://udacity-dend/log_data

```
**song data**: Each file is in JSON format and contains metadata about a song and the artist of that song. The files are partitioned by the first three letters of each song's track ID. For example, here are filepaths to two files in this dataset.

```
song_data/A/B/C/TRABCEI128F424C983.json
song_data/A/A/B/TRAABJL12903CDCF1A.json
```

Below is an example of what a single song file, TRAABJL12903CDCF1A.json, looks like.
```
{"num_songs": 1, "artist_id": "ARJIE2Y1187B994AB7", "artist_latitude": null, 
"artist_longitude": null, "artist_location": "", "artist_name": "Line Renaud", "song_id": "SOUPIRU12A6D4FA1E1", 
"title": "Der Kleine Dompfaff", "duration": 152.92036, "year": 0}
```

**log data**: Each file is in JSON format and contains app activity logs. The files are partitioned by year and month.

And below is an example of what the data in a log file, 2018-11-12-events.json, looks like.
![log_data](./log-data.png)


### Database Schema 

The database is created on a star schema including five tables. 

The fact table is **songplays** - records in log data associated with song plays i.e. records with page NextSong.
**songplays** as the fact table consists of foreign key to four dimension tables: **users**,**songs**,**artists**,**time**.

![Database ER diagram](./ERdiagram.png)


## ETL Pipeline

ETL Pipeline is built to 
+ extract data from a directory of JSON logs on user activity and JSON metadata on the songs storein S3.
+ process the data into Spark DataFrames using Spark.
+ transforms Spark DataFrame into analytics tables.

### Data Cleaning

* Filter on column page = 'NextSong' when creating log DataFrame.
* Use dropDuplicates() to handles duplicate records where appropriate.

## Instructions

* run etl.py to transfer data into the data lake.


## Example Queries and Results
Question: Is there subscription percentage difference between male and female users?

Queries:
```
"SELECT gender, 
        COUNT(*) AS gender_count, 
        ROUND(AVG(CASE WHEN level = 'paid' THEN 100 ELSE 0 END),2) AS paid_percentage
FROM (SELECT DISTINCT s.user_id, 
                      gender, 
                      s.level 
      FROM songplays s 
      JOIN users u 
      ON s.user_id = u.user_id) t
GROUP BY 1"
```

Results:
```
gender gender_count paid_percentage
F        60           25.00%
M        44           15.91%
```

**Subscription rate is higher for female in the sample**

# PROJECT-4: Data Lake (Submitted by Abdulrahman ElKhereiji DEND 2019)

## Introduction

*A music streaming startup, Sparkify, has grown their user base and song database even more 
and want to move their data warehouse to a data lake. 
Their data resides in S3, in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app*

*As their data engineer, you are tasked with building an ETL pipeline that extracts their data from S3, processes them using Spark, and loads the data back into S3 as a set of dimensional tables. This will allow their analytics team to continue finding insights in what songs their users are listening to.*

## Purpose of this database

In this project,  Spark and data lakes shall be used to build an ETL pipeline for a data lake hosted on S3. To complete the project, loading data from S3 shall be required, process the data into analytics tables using Spark, and load them back into S3. Furthermore,  this Spark process shall be deployed on a cluster using AWS.

## Setup Instructions and Steps followed 

The steps of the project:

* Rename dl_template.cfg to dl.cfg and fill in the open fields. 
* Fill in AWS acces key (KEY) and secret (SECRET).
* Create an AWS S3 bucket under the name of **(dend-project4-udacity)**
* Edit dl.cfg: add your S3 bucket name.
* Copy log_data and song_data folders to your own S3 bucket.
* Create output_data folder in your S3 bucket.

## Datasets available

### /data: 
*This folder consists of two zip files, helpful for data exploration*

* This Project-4 handles data of a music streaming startup, Sparkify. Data set is a set of files in JSON format stored in AWS S3 buckets and contains two parts:

1) Song data (14897 files)

* **s3://udacity-dend/song_data**: static data about artists and songs
  Song-data example:
  `{"num_songs": 1, "artist_id": "ARJIE2Y1187B994AB7", "artist_latitude": null, "artist_longitude": null, "artist_location": "", "artist_name": "Line Renaud", "song_id": "SOUPIRU12A6D4FA1E1", "title": "Der Kleine Dompfaff", "duration": 152.92036, "year": 0}`

2) Log data (31 files)

* **s3://udacity-dend/log_data**: event data of service usage e.g. who listened what song, when, where, and with which client

### etl.py:
*The ETL engine done with Spark, data normalization and parquet file writing*

### dl.cfg:
*Configuration file that contains info about AWS credentials*

### Schema Design:
Sparkify analytics database (called here sparkifydb) schema has a star design. Start design means that it has one Fact Table having business data, and supporting Dimension Tables. Star DB design is maybe the most common schema used in ETL pipelines since it separates Dimension data into their own tables in a clean way and collects business critical data into the Fact table allowing flexible queries. The Fact Table answers one of the key questions: what songs users are listening to. DB schema is shown in **Schema Design.png**

#### Results:

* Input data was available first offline during the development phase and later from s3://udacity-dend/song_data and s3://udacity-dend/log_data
* As expected, reading and especially writing data (song_data, log_data) to and from S3 is very slow process due to large amount of input data and slow network connection. See above for figures with 1 song_data file and 1 log_data file.

##### Fact Table

* **songplays**: song play data together with user, artist, and song info (songplay_id, start_time, user_id, level, song_id, artist_id, session_id, location, user_agent)

##### Dimension Tables

* **users**: user info (columns: user_id, first_name, last_name, gender, level)
* **songs**: song info (columns: song_id, title, artist_id, year, duration)
* **artists**: artist info (columns: artist_id, name, location, latitude, longitude)
* **time**: detailed time info about song plays (columns: start_time, hour, day, week, month, year, weekday)


## Program Execution:
* Output data was written to own AWS S3 bucket.
* Both Fact table (songplays) and Dimension tables (users, songs, artists, time) are extracted from input data and written back to S3 as Spark parquet files.
* Each time `etl.py` is run, it creates new directories for each table to output_data/ directory based on the script start time (e.g. songs_table.parquet_2019-06-10-10-07-20-873438). This way there's no collision or need to erase old data between different runs.
* Spark writes temporary folders and files during the parquet file creation process. Used AWS S3 user requires admin rights for Spark to be able to rename (copy + paste + delete) created temp folders and files.
* ETL script requires some optimisation for bigger data amounts (e.g. write parquet files locally and only then upload them to S3).
* Two folders were created after running the etl.py , which are: **songplays_table.parquet** & **spark-warehouse**
* an **etl_test.ipynb** file was created to test the code quality of the project. 
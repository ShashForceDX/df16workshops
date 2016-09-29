+++
date = "2016-07-29T16:18:23+05:30"
draft = true
title = "Text Classification : Spam Detection Engine using Logistical Regression : Local"

+++

## Introduction

In this workshop you will learn how to use PredictionIO Machine Learning library to build a Spam Detection Engine using Classification Technique of Logisitical Regression. PredictionIO uses Spark MLlib's implementation and provide convenient APIs and REST endpoints to get the infrastructure up and running fast.

## Pre-requisities

* git command line
* JDK 1.8.x or above

## Source Code

Source code of this workshop resides in two repos listed below

* https://github.com/rajdeepd/pio-eventserver-heroku
* https://github.com/rajdeepd/pio-engine-textclassfication-heroku
* https://github.com/rajdeepd/pio-upload-data

**pio-eventserver-heroku** : Provides storage for events being generated based on which we want to create our training model.

**pio-textclassfication-engine-heroku** : Engine which wraps the Logistical Regression Algorithm implementation and provides APIs to create a model, train it and use it to make prediction.

We will be using PostgreSQL database for this workshop.

## Step 1 : Clone the Source code 

Clone the source code

``` bash

$ git clone https://github.com/rajdeepd/pio-eventserver-heroku

$ git clone https://github.com/rajdeepd/pio-engine-textclassfication-heroku

$ git clone https://github.com/rajdeepd/pio-upload-data


```

## step 2 : Start the PostgreSQL server

Make sure PostgreSQL server is running locally

**Linux**

``` bash

$ sudo service postgresql status

```

**Mac OS X**

``` bash

$ /Library/PostgreSQL/9.6/bin/pg_ctl restart -D /Library/PostgreSQL/9.6/data/

```

Create a pio role with password pio in PostgreSQL

``` bash

CREATE USER pio WITH PASSWORD 'pio';

```

## Step 3 : Run the Event Server

Use the following command to run the Event Server.

``` bash

source bin/env.sh && ./sbt run

```

## Step 4 : Create an Event Application

``` bash
cd pio-eventserver-heroku
source bin/env.sh && ./sbt \
  "runMain io.prediction.tools.console.Console app new MyAppText"

```
Output of the command above will generate information about accesskey. We will be using this Access Key later.

``` bash

[INFO] [App$] Initialized Event Store for this app ID: 4.
[INFO] [App$] Created new app:
[INFO] [App$]       Name: MyAppText
[INFO] [App$]         ID: 4
[INFO] [App$] Access Key: SOME_ACCESS_KEY

```

You can see the pio app related data and meta data tables being created in PostgreSQL


``` bash
$ psql -h 127.0.0.1 -U pio -W
Password for user pio: 
psql (9.5.3, server 9.3.9)
SSL connection (protocol: TLSv1.2, cipher: ..., bits: 256, compression: off)
Type "help" for help.

pio=> \d+
                                  List of relations
 Schema |             Name             |   Type   | Owner |    Size    | 
--------+------------------------------+----------+-------+------------+-
 public | pio_event_1                  | table    | pio   | 16 kB      | 
 public | pio_meta_accesskeys          | table    | pio   | 16 kB      | 
 public | pio_meta_apps                | table    | pio   | 16 kB      | 
 public | pio_meta_apps_id_seq         | sequence | pio   | 8192 bytes | 
 public | pio_meta_channels            | table    | pio   | 8192 bytes | 
 public | pio_meta_channels_id_seq     | sequence | pio   | 8192 bytes | 
 public | pio_meta_engineinstances     | table    | pio   | 64 kB      | 
 public | pio_meta_enginemanifests     | table    | pio   | 16 kB      | 
 public | pio_meta_evaluationinstances | table    | pio   | 8192 bytes | 
 public | pio_model_models             | table    | pio   | 16 kB      | 
(11 rows)

pio=> 

```

``` bash

pio=> select * from pio_meta_apps;
 id |  name  | description 
----+--------+-------------
  1 | MyAppText | 


```

## Step 4 : Add Demo Data from pio-upload-data

We are going to upload emails and stopwords stored in json files.

We will use a Scala Application cloned from [https://github.com/rajdeepd/pio-upload-data](https://github.com/rajdeepd/pio-upload-data) to upload data. It will make a Http Post request and upload email and stop word data.

Sample email event

``` json

{
  "eventTime": "2015-06-08T16:45:00.595+0000",
  "entityId": 2,
  "properties": {
    "text": " some spam text",
    "label": "spam"
  },
  "event": "e-mail",
  "entityType": "content"
}

```


1. Copy application.conf.sample to application.conf as shown below

    ``` bash

    $ cd pio-upload-data
    $ cp pio-upload-data/src/main/resources/application.conf.sample \
       pio-upload-data/src/main/resources/application.conf
      
    ```

2. Open `application.conf` file

    ``` bash

    pio {
      access_token = "TODO"
      app_name = "TODO"
      host = "localhost:7070"
    }

    ```
    
    Update with appropriate values as shown below


    ``` bash
      
    pio {
      access_token = "2eNMw5lydFtFl4uT..Jz9sIEUuigshZJHttReO5lpiNDeZwELVV3_7"
      app_name = "MyTextLR"
      host = "rd-pio-eventserver-1.herokuapp.com"
    }

    ```
3. Execute the following commands on `UploadEmails` and `UploadStopWords` 


    ``` bash

    $ cd ~/pio-upload-data
    $ ./sbt "runMain UploadEmails"
    $ ./sbt "runMain UploadStopWords"


    ```

Check the data generated using the following curl command

```

http://localhost:7070/events.json?accessKey=SOME_ACCESS_KEY&limit=-1

```

<img src="/workshop/prediction-io/text_classification/images/pio-events-text-classification-local.png" width="100%" height="100%">

## Step 5 : Train the Model

1. Set your PredictionIO app's access key and app name in your env vars:
   
     ``` bash

     export ACCESS_KEY=<YOUR ACCESS KEY>
     export APP_NAME=<YOUR APP NAME>

     ```

    Note: These values come from the apps defined in your event server.

2. Train the app:
    
     ``` bash

     source bin/env.sh && ./sbt "runMain TrainApp"

     ```

3. Start the server:

    ``` bash

    source bin/env.sh && ./sbt "runMain ServerApp"

    ```

4. Check the status of your engine at `http://localhost:8000`


     <img src="/workshop/prediction-io/text_classification/images/pio-engine-text-classification-local.png" width="100%" height="100%">

## Step 6 : Predict Spam vs Non Spam

   ``` bash

$ curl -H "Content-Type: application/json" -d '{ "text":"Earn extra cash!" }' \
    http://localhost:8000/queries.json

   ```
   
Response

   ``` json 
{"category":"not spam","confidence":0.7219493232910391}

   ```


**Note:** If the confidence level is showing as Nan, it means the data used to train is not sufficient for calculating the confidence level.




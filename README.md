# mastodon-stream-nlp-data-analysis

This repository contains source files for mastodon stream npl polarity analysis. I built an end-to-end batching pipeline keeping all simple and understable. There are two ways to generate streaming data: first one you must have a mastodon account and get an AccessToken (is free). The second ways is generating a fake stream of data (included in this files). Either way you will be able to run the pipeline to see how these services work together.

![alt text](assetsdiagram.png "P2")

Prerequisites:

Make sure you have:

Installed the latest version of Docker Compose

The firt step is build and start all the services in [text](docker-compose.yml)

```
cd ./mastodon-stream-nlp-data-analysis

docker-compose up -d
```

If you have installed docker desktop you should see something like this
![alt text](assets/diagram.png)

Next we need to start the producer!

curl localhost:8080/streaming_csv

This will send to kafka service a string every 5 seconds

If you want see it in action:

```
docker exec -it kafka-server bash

/opt/bitnami/kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic kafka_topic
```

The output will be the same that previuos command

Now we need to start the batch process:

```
docker exec -it spark-server bash  

cd /code/src/spark

spark-submit \  
--packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.2.3 \  
--conf spark.driver.extraJavaOptions="-Divy.cache.dir=/tmp -Divy.home=/tmp" \  
--driver-class-path postgresql-42.6.2.jar \  
--jars postgresql-42.6.2.jar \  
--master spark://spark:7077 \  
--deploy-mode client \  
--driver-memory 3g \  
--num-executors 2 \  
--executor-cores 4 \  
--executor-memory 3g \  
./main.py
```

The batch process save the transformated data in PostgreSQL
![alt text](assets/postgresql.png)

And now we need to connect to python server to start the streamlit app:

```
docker exec -it python-server bash  

cd src/app/streamlit  

streamlit run main.py
```

And just open the following url in your browser:

http://localhost:8501/

![alt text](assets/streamlit.gif)

And that's it, now you have end-to-end streaming pipeline running locally!




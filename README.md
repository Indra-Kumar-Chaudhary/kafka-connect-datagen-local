# kafka-connect-datagen
How to generate mock data to a local Kafka topic using the Kafka Connect Datagen

<h1>Question:</h1>
<p>How can I produce mock data to Kafka topics to test my Kafka applications?</p>

<h1>Example use case</h1>
We will run a local instance of the <b>Kafka Connect Datagen connector </b> to produce mock data  to a local Kafka cluster. This facilates learning about Kafka and testing your applications. 

<h1> 1 Initialize the project </h1>
<p>
    To ge started, make a new directory anywhere you'd like for this project:
</p>
<i>mkdir kafka-connect-datagen-local && cd kafka-connect-datagen-local</i> 

<h1>2 Get Confluent Platform</h1>
<p>
    Create a <b>Dockerfile</b> that builds a custom container for Kafka Connect bundled with the free and open source <b>Kafka Connect Datagen</b> connector, installed from <b>Confluent Hub.</b>
</p>
<pre>
FROM confluentinc/cp-kafka-connect-base:5.5.0

ENV CONNECT_PLUGIN_PATH="/usr/share/java,/usr/share/confluent-hub-components"

RUN confluent-hub install --no-prompt confluentinc/kafka-connect-datagen:0.3.2
</pre>

<p>Next, create the following <b>docker-compose.yml</b> file to obtain Confluent Platform. Note that it also configured to build a local image. 
</p> 


<p> Now launch Confluent Platform by running the following command. Note the <b> --build </b>
argument which automactically builds the Docker image for Kafka Connect and the bundled kafka-connect-datagen connectory. 
</p>
<i> docker-compose up -d --build </li> 

<h1>3 Create the connector </h1> 
<p>
    Create the Kafka Connect Datagen source connector. It automatically creates the Kafka topic <b>pageviews</b> and produces data to it with a schema specification from <b>https://github.com/confluentinc/kafka-connect-datagen/blob/master/src/main/resources/pageviews_schema.avro</b>
</p>

<pre>
    CURL -i -X PUT http://localhost:8083/connectors/datagen_local_01/config \ 
    -H "Content-Type: application/json" \ 
    -d '{
            "connector.class": "io.confluent.kafka.connect.datagen.DatagenConnector",
            "key.converter": "org.apache.kafka.connect.storage.StringConverter",
            "kafka.topic": "pageviews",
            "quickstart": "pageviews",
            "max.interval": 1000,
            "iterations": 10000000,
            "tasks.max": "1
    }' 
</pre>
<p>
If you run this before Kafka Connect has finished starting up we'll get error <b> curl: (52) Empty reply from server </b> - in which case, rerun the above command. 

Check that the connector is running: 
<i>curl -s http://localhost:8083/connectors/datagen_local_01/status</i>

</p>
<p> You should see that state is <b>Running<b> for both <b>connector</b> and <b>tasks </b> elements
<b>{"name":"datagen_local_01","connector":{"state":"RUNNING","worker_id":"connect:8083"},"tasks":[{"id":0,"state":"RUNNING","worker_id":"connect:8083"}],"type":"source"}</b>
<p>

<p>
If you get the message <b>{"error_code":404,"message":"No status found for connector datagen_local_01"}</b> then check that the step above in which you created the connector actually succeeded.
</p>


<h1>4 Consume events from the Kafka topic</h1> 
<p>
Now that the kafka-connect-datagen is running, run the Kafka Avro console consumer to see the data streaming into the Kafka topic.
</p>

<p>Note the added properties of <b>print.key</b> and <b>key.separator</b>

<p>
<pre>
   docker-compose exec connect kafka-avro-console-consumer \
 --bootstrap-server broker:9092 \
 --property schema.registry.url=http://schema-registry:8081 \
 --topic pageviews \
 --property print.key=true \
 --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
 --property key.separator=" : " \
 --max-messages 10

</pre>
# qcon-ai-workshop
Exercises for QCon Workshop

## Exercise 2 - Schemas, Schema Registry and Compatibility
In this exercise we'll design an Avro schema, registry it in the Confluent Schema Registry, produce and consume events using this schema, and then modify the schema in compatible and incompatible ways.

We assume you already have the environment up and running from the first exercise.

1. Think of a stream processing use-case that interests you. What kind of data do you have? Which topics will you need? select one of the topics and decide on key and value schema for records in the topic. How did the choice of topics influence the event schema? What trade-offs did you make in designing the data model?

2. Write down the schema definition in JSON format. You can see the schema definition rules for Avro here: https://avro.apache.org/docs/1.8.1/spec.html#schemas

If you are stuck coming up with your own schema, you can find a schema that we created for our movies topic in `movies-raw.avsc`.

3. Now, lets register the schema in the Confluent Schema Registry. 
Instructions can be found in Schema Registry documentation: https://docs.confluent.io/current/schema-registry/docs/intro.html#quickstart
This is schema for event values, and in our case it will belong to topic `movies-raw`, so I'll register the schema under the subject `movies-raw-key`.

It is important to note the details of the Schema Registry API for registering a schema: https://docs.confluent.io/current/schema-registry/docs/api.html#post--subjects-(string-%20subject)-versions
It says:
```
Request JSON Object:
 	
schema – The Avro schema string
```
Which means that we need to pass to the API a JSON record, with one key "schema" and the value is a string containing our schema.
We can't pass the schema itself when registering it.
I used a rather new version of `jq` to wrap our Avro Schema appropriately: `jq -n --slurpfile schema movies-raw.avsc  '$schema | {schema: tostring}'`
And then passed the output of `jq` to `curl` with a pipe:
```
jq -n --slurpfile schema movies-raw.avsc  '$schema | {schema: tostring}' | curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" --data @- http://localhost:8081/subjects/movies-raw-value/versions
```

The output should be an ID. Remember the ID you got, so you can use it when producing and consuming events.


# iDaaS-KIC
iDaas Knowledge, Insight and Confromance Platform. This repository is designed to contain all the assets needed to support implementing the entire design pattern. 

# iDaaS-KIC_DataTiers
Within this module within this repository you will find all the needed datatier DDLs to implement and any other customized data tier centric needs.

## Supported RDBMS
The intent is to be able to leverage ANY RDBMS that organizations are comfortable with. For testing we tested it with the open source implementations of PostGres and MariaDB/MySQL Community (version 8 or greater). MySQL 8 has been prodimently used and implemented.

# iDaaS-KIC-Integration
Within this module is the design pattern that persists errors, auditing and any relevant data activities that occur within iDaaS. The current main usage is for auditing and logging activities.

## Pre-Requisites
For all iDaaS design patterns it should be assumed that you will either install as part of this effort, or have the following:

1. An existing Kafka (or some flavor of it) up and running. Red Hat currently implements AMQ-Streams based on Apache Kafka; however, we
have implemented iDaaS with numerous Kafka implementations. Please see the following files we have included to try and help: <br/>
[Kafka](https://github.com/RedHat-Healthcare/iDaaS-Demos/blob/master/Kafka.md)<br/>
[KafkaWindows](https://github.com/RedHat-Healthcare/iDaaS-Demos/blob/master/KafkaWindows.md)<br/>
No matter the platform chosen it is important to know that the Kafka out of the box implementation might require some changes depending
upon your implementation needs. Here are a few we have made to ensure: <br/>
In <kafka>/config/consumer.properties file we will be enhancing the property of auto.offset.reset to earliest. This is intended to enable any new 
system entering the group to read ALL the messages from the start. <br/>
auto.offset.reset=earliest <br/>
2. Some understanding of building, deploying Java artifacts and the commands associated. If using Maven commands then Maven would need to be intalled and runing for the environment you are using. More details about Maven can be found [here](https://maven.apache.org/install.html)<br/>
3. An internet connection with active internet connectivity, this is to ensure that if any Maven commands are
run and any libraries need to be pulled down they can.<br/>

## Scenario: Integration
This repository follows a very common implementation. The implementation
is processing all the data from the topic named opmgmt_transactions.

### Integration Data Flow Steps

1. The Topic opmgmt_transactions is subscribed to for transactions.<br/>
2. The data will be processed from the topic and the header attributes will all be parsed.<br/>
3. The header attributes should then be persisted to the appropriate database fields in appauditing_auditlog and
   appauditing_auditlog_msgs depending upon data attributes.<br/>

# Start The Engine!!!
This section covers the running of the solution. There are several options to start the Engine Up!!!

## Step 1: Kafka Server To Connect To
In order for ANY processing to occur you must have a Kafka server running that this accelerator is configured to connect to.
Please see the following files we have included to try and help: <br/>
[Kafka](https://github.com/RedHat-Healthcare/iDaaS-Demos/blob/master/Kafka.md)<br/>
[KafkaWindows](https://github.com/RedHat-Healthcare/iDaaS-Demos/blob/master/KafkaWindows.md)<br/>

## Step 2: Running the App: Maven Commands or Code Editor
This section covers how to get the application started.
+ Maven: go to the directory of where you have this code. Specifically, you want to be at the same level as the POM.xml file and execute the
following command: <br/>
```
mvn clean install
 ```
You can run the individual efforts with a specific command, it is always recommended you run the mvn clean install first.
Here is the command to run the design pattern from the command line: <br/>
```
mvn spring-boot:run
 ```
Depending upon if you have every run this code before and what libraries you have already in your local Maven instance it could take a few minutes.
+ Code Editor: You can right click on the Application.java in the /src/<application namespace> and select Run

# Running the Java JAR
If you don't run the code from an editor or from the maven commands above. You can compile the code through the maven
commands above to build a jar file. Then, go to the /target directory and run the following command: <br/>
```
java -jar <jarfile>.jar 
 ```

### Design Pattern/Accelerator Configuration
All iDaaS Design Pattern/Accelelrators have application.properties files to enable some level of reusability of code and simplfying configurational enhancements.<br/>
In order to run multiple iDaaS integration applications we had to ensure the internal http ports that
the application uses. In order to do this we MUST set the server.port property otherwise it defaults to port 8080 and ANY additional
components will fail to start. iDaaS Connect HL7 uses 9980. You can change this, but you will have to ensure other applications are not
using the port you specify.	

Once built you can run the solution by executing `./platform-scripts/start-solution.sh`.
The script will startup Kafka and iDAAS DataHub Service.

Alternatively, if you have a running instance of Kafka, you can start a solution with:
`./platform-scripts/start-solution-with-external-kafka.sh --idaas.kafkaBrokers=host1:port1,host2:port2`.
The script will startup iDAAS DataHub Service.

Optionally you can configure the service to store audit data in DB. You can run an instance of Postgres db with:

`docker run --rm --name datahub-db -e POSTGRES_PASSWORD=Postgres123 -p 5432:5432 postgres:alpine`

You can enable storing audit in DB by running the service with `--idaas.storeInDb=true` parameter. You can customize
default DB connection details by providing the following properties:
```properties
idass.dbDriverClassName=org.postgresql.Driver
idaas.dbUrl=jdbc:postgresql://localhost:5432/audit_db
idaas.dbUsername=postgres
idass.dbPassword=Postgres123
idaas.dbTableName=audit
idaas.createDbTable=true
```

It is possible to overwrite configuration by:
1. Providing parameters via command line e.g.
`./start-solution.sh --idaas.auditDir=some/other/audit/dir`
2. Creating an application.properties next to the idaas-datahub.jar in the target directory
3. Creating a properties file in a custom location `./start-solution.sh --spring.config.location=file:./config/application.properties`

Supported properties include:
```properties
server.port=9070

idaas.kafkaBrokers=localhost:9092 #a comma separated list of kafka brokers e.g. host1:port1,host2:port2
idaas.kafkaTopicName=opsmgmt_platformtransactions
idaas.auditDir=audit

idaas.storeInDb=false
idass.dbDriverClassName=org.postgresql.Driver
idaas.dbUrl=jdbc:postgresql://localhost:5432/audit_db
idaas.dbUsername=postgres
idass.dbPassword=Postgres123
ioaas.dbTableName=audit
idaas.createDbTable=true
```

### Usage

The service listens for Kafka messages on the `opsmgmt_platformtransactions` queue by default and outputs them to
'audit' dir as json files.

The output includes both message and headers.

There's also an endpoint available for putting messages on the queue. The REST service runs on port 8080 by default, but
it can be customized by providing `--server.port=8082` parameter.

You can POST a message like this:
```shell script
curl --location --request POST 'http://localhost:9070/message' \
--header 'Content-Type: application/json' \
--data-raw '{
    "auditEntireMessage": "test",
    "processingtype": "data",
	"industrystd": "HL7",
	"messagetrigger": "ADT"
}'
```

# Admin Interface - Management and Insight of Components
Within each specific repository there is an administrative user interface that allows for monitoring and insight into the
connectivity of any endpoint. Additionally, there is also the implementation to enable implementations to build there own
by exposing the metadata. The data is exposed and can be used in numerous very common tools like Data Dog, Prometheus and so forth.
This capability to enable would require a few additional properties to be set.

Below is a generic visual of how this looks (the visual below is specific to iDaaS Connect HL7): <br/>
![iDaaS Platform - Visuals - iDaaS Data Flow - Detailed.png](https://github.com/RedHat-Healthcare/iDAAS/blob/master/images/iDAAS-Platform/iDaaS-Mgmt-UI.png)

Every asset has its own defined specific port, we have done this to ensure multiple solutions can be run simultaneously.

## Administrative Interface(s) Specifics
For all the URL links we have made them localhost based, simply change them to the server the solution is running on.

|<b> iDaaS Connect Asset | Port | Admin URL / JMX URL |
| :---        | :----   | :--- | 
|iDaaS KIC | 9970| http://localhost:9970/actuator/hawtio/index.html / http://localhost:9970/actuator/jolokia/read/org.apache.camel:context=*,type=routes,name=* | 

If you would like to contribute feel free to, contributions are always welcome!!!! 

Happy using and coding....



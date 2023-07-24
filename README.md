# MongoDB Atlas Java Lab

## Introduction
This objective of this lab is to demostrate the following:
1.  Anything related to MongoDB Java driver
2.  Anything related to the use of Java JDK in the context of MongoDB

## Pre-requisites and Notes
1.  Software Versions & MongoDB Atlas configurations
    - Git version 2.39.2
    - Java(TM) SE Runtime Environment (build 20.0.2+9-78)
    - Maven version 13.4.1
    - Intellij IDEA CE 2023.1.4
    - Visual Studio Code version 1.80.1 (Universal)
    - [MongoDB Atlas Sample Datasets](https://www.mongodb.com/developer/products/atlas/atlas-sample-datasets/)
    - MongoDB Atlas connection string:  ```mongodb+srv://USERNAME:PASSWORD@wt-cluster-0.k0k8u7u.mongodb.net/?retryWrites=true&w=majority```
      
2.  Compiling Java
    - To compile a specific .java, in the root/src folder of the java folder, run the following command in command shell/CLI: ```mvn compile exec:java -Dexec.mainClass="<.java path>"``` e.g. ```mvn compile exec:java -Dexec.mainClass="com.mongodb.quickstart.HelloMongoDB"```
    - To compile all .java in the library, in the root/src folder of the java folder, run the following command in command shell/CLI: ```mvn clean compile```
      
3.  Relevant URLs
    - [GitHub - mongodb-developer/java-quick-start](https://github.com/mongodb-developer/java-quick-start)
    - [Blog - Getting Started with MongoDB and Java - CRUD Operations Tutorial](https://www.mongodb.com/developer/languages/java/java-setup-crud-operations/)

## Steps

### 1 - Connecting to Atlas & list all databases
1.  Java class Connection
```
package com.mongodb.quickstart;

import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import org.bson.Document;

import java.util.ArrayList;
import java.util.List;

public class Connection {

    public static void main(String[] args) {
        String connectionString = System.getProperty("mongodb.uri");
        try (MongoClient mongoClient = MongoClients.create(connectionString)) {
            List<Document> databases = mongoClient.listDatabases().into(new ArrayList<>());
            databases.forEach(db -> System.out.println(db.toJson()));
        }
    }
}
```

2.  Run this example command/class to execute:
```
mvn compile exec:java -Dexec.mainClass="com.mongodb.quickstart.Connection" -Dmongodb.uri="mongodb+srv://USERNAME:PASSWORD@wt-cluster-0.k0k8u7u.mongodb.net/test?w=majority" -Dexec.cleanupDaemonThreads=false
```

3.  Output example:
```
{"name": "sample_airbnb", "sizeOnDisk": 59469824, "empty": false}
{"name": "sample_analytics", "sizeOnDisk": 9564160, "empty": false}
{"name": "sample_geospatial", "sizeOnDisk": 1294336, "empty": false}
{"name": "sample_guides", "sizeOnDisk": 40960, "empty": false}
{"name": "sample_mflix", "sizeOnDisk": 115322880, "empty": false}
{"name": "sample_restaurants", "sizeOnDisk": 9203712, "empty": false}
{"name": "sample_supplies", "sizeOnDisk": 1097728, "empty": false}
{"name": "sample_training", "sizeOnDisk": 52293632, "empty": false}
{"name": "sample_weatherdata", "sizeOnDisk": 2711552, "empty": false}
{"name": "admin", "sizeOnDisk": 344064, "empty": false}
{"name": "local", "sizeOnDisk": 1525587968, "empty": false}
```

### 2 - Insert operations of 1 and many documents
1.  Java class Create
```
package com.mongodb.quickstart;

import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.model.InsertManyOptions;
import org.bson.Document;
import org.bson.types.ObjectId;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;

import static java.util.Arrays.asList;

public class Create {

    private static final Random rand = new Random();

    public static void main(String[] args) {
        try (MongoClient mongoClient = MongoClients.create(System.getProperty("mongodb.uri"))) {

            MongoDatabase sampleTrainingDB = mongoClient.getDatabase("sample_training");
            MongoCollection<Document> gradesCollection = sampleTrainingDB.getCollection("grades");

            insertOneDocument(gradesCollection);
            insertManyDocuments(gradesCollection);
        }
    }

    private static void insertOneDocument(MongoCollection<Document> gradesCollection) {
        gradesCollection.insertOne(generateNewGrade(10000d, 1d));
        System.out.println("One grade inserted for studentId 10000.");
    }

    private static void insertManyDocuments(MongoCollection<Document> gradesCollection) {
        List<Document> grades = new ArrayList<>();
        for (double classId = 1d; classId <= 10d; classId++) {
            grades.add(generateNewGrade(10001d, classId));
        }

        gradesCollection.insertMany(grades, new InsertManyOptions().ordered(false));
        System.out.println("Ten grades inserted for studentId 10001.");
    }

    private static Document generateNewGrade(double studentId, double classId) {
        List<Document> scores = asList(new Document("type", "exam").append("score", rand.nextDouble() * 100),
                                       new Document("type", "quiz").append("score", rand.nextDouble() * 100),
                                       new Document("type", "homework").append("score", rand.nextDouble() * 100),
                                       new Document("type", "homework").append("score", rand.nextDouble() * 100));
        return new Document("_id", new ObjectId()).append("student_id", studentId)
                                                  .append("class_id", classId)
                                                  .append("scores", scores);
    }
}
```

2.  Run this example command/class to execute:
```
mvn compile exec:java -Dexec.mainClass="com.mongodb.quickstart.Create" -Dmongodb.uri="mongodb+srv://USERNAME:PASSWORD@wt-cluster-0.k0k8u7u.mongodb.net/test?w=majority" -Dexec.cleanupDaemonThreads=false
```

3.  Output example:
```
{
    "_id": {
        "$oid": "5d97c375ded5651ea3462d0f"
    },
    "student_id": {
        "$numberDouble": "10000"
    },
    "class_id": {
        "$numberDouble": "1"
    },
    "scores": [{
        "type": "exam",
        "score": {
            "$numberDouble": "4.615256396625178"
        }
    }, {
        "type": "quiz",
        "score": {
            "$numberDouble": "73.06173415145801"
        }
    }, {
        "type": "homework",
        "score": {
            "$numberDouble": "19.378205578990727"
        }
    }, {
        "type": "homework",
        "score": {
            "$numberDouble": "82.3089189278531"
        }
    }]
}
```

### 3 - Read operations of 1 specific document and a range of documents
1.  Java class Read
```
package com.mongodb.quickstart;

import com.mongodb.client.*;
import org.bson.Document;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Consumer;

import static com.mongodb.client.model.Filters.*;
import static com.mongodb.client.model.Projections.*;
import static com.mongodb.client.model.Sorts.descending;

public class Read {

    public static void main(String[] args) {
        try (MongoClient mongoClient = MongoClients.create(System.getProperty("mongodb.uri"))) {
            MongoDatabase sampleTrainingDB = mongoClient.getDatabase("sample_training");
            MongoCollection<Document> gradesCollection = sampleTrainingDB.getCollection("grades");

            // find one document with new Document
            Document student1 = gradesCollection.find(new Document("student_id", 10000)).first();
            System.out.println("Student 1: " + student1.toJson());

            // find one document with Filters.eq()
            Document student2 = gradesCollection.find(eq("student_id", 10000)).first();
            System.out.println("Student 2: " + student2.toJson());

            // find a list of documents and iterate throw it using an iterator.
            FindIterable<Document> iterable = gradesCollection.find(gte("student_id", 10000));
            MongoCursor<Document> cursor = iterable.iterator();
            System.out.println("Student list with a cursor: ");
            while (cursor.hasNext()) {
                System.out.println(cursor.next().toJson());
            }

            // find a list of documents and use a List object instead of an iterator
            List<Document> studentList = gradesCollection.find(gte("student_id", 10000)).into(new ArrayList<>());
            System.out.println("Student list with an ArrayList:");
            for (Document student : studentList) {
                System.out.println(student.toJson());
            }

            // find a list of documents and print using a consumer
            System.out.println("Student list using a Consumer:");
            Consumer<Document> printConsumer = document -> System.out.println(document.toJson());
            gradesCollection.find(gte("student_id", 10000)).forEach(printConsumer);

            // find a list of documents with sort, skip, limit and projection
            List<Document> docs = gradesCollection.find(and(eq("student_id", 10001), lte("class_id", 5)))
                                                  .projection(fields(excludeId(), include("class_id", "student_id")))
                                                  .sort(descending("class_id"))
                                                  .skip(2)
                                                  .limit(2)
                                                  .into(new ArrayList<>());

            System.out.println("Student sorted, skipped, limited and projected: ");
            for (Document student : docs) {
                System.out.println(student.toJson());
            }
        }
    }
}
```

2.  Run this example command/class to execute:
```
mvn compile exec:java -Dexec.mainClass="com.mongodb.quickstart.Read" -Dmongodb.uri="mongodb+srv://USERNAME:PASSWORD@wt-cluster-0.k0k8u7u.mongodb.net/test?w=majority" -Dexec.cleanupDaemonThreads=false
```

3.  Output examples with DEBUG statements:
```
17:02:52.084 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-srv-wt-cluster-0.k0k8u7u.mongodb.net] INFO org.mongodb.driver.cluster - Adding discovered server ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017 to client view of cluster

17:02:52.085 [com.mongodb.quickstart.Read.main()] INFO org.mongodb.driver.client - MongoClient with metadata {"driver": {"name": "mongo-java-driver|sync", "version": "4.8.1"}, "os": {"type": "Darwin", "name": "Mac OS X", "architecture": "aarch64", "version": "13.4.1"}, "platform": "Java/Oracle Corporation/20.0.2+9-78"} created with settings MongoClientSettings{readPreference=primary, writeConcern=WriteConcern{w=majority, wTimeout=null ms, journal=null}, retryWrites=true, retryReads=true, readConcern=ReadConcern{level=null}, credential=MongoCredential{mechanism=null, userName='admin', source='admin', password=<hidden>, mechanismProperties=<hidden>}, streamFactoryFactory=null, commandListeners=[], codecRegistry=ProvidersCodecRegistry{codecProviders=[ValueCodecProvider{}, BsonValueCodecProvider{}, DBRefCodecProvider{}, DBObjectCodecProvider{}, DocumentCodecProvider{}, CollectionCodecProvider{}, IterableCodecProvider{}, MapCodecProvider{}, GeoJsonCodecProvider{}, GridFSFileCodecProvider{}, Jsr310CodecProvider{}, JsonObjectCodecProvider{}, BsonCodecProvider{}, EnumCodecProvider{}, com.mongodb.Jep395RecordCodecProvider@394e5205]}, clusterSettings={hosts=[127.0.0.1:27017], srvHost=wt-cluster-0.k0k8u7u.mongodb.net, srvServiceName=mongodb, mode=MULTIPLE, requiredClusterType=REPLICA_SET, requiredReplicaSetName='atlas-wdblrt-shard-0', serverSelector='null', clusterListeners='[]', serverSelectionTimeout='30000 ms', localThreshold='30000 ms'}, socketSettings=SocketSettings{connectTimeoutMS=10000, readTimeoutMS=0, receiveBufferSize=0, sendBufferSize=0}, heartbeatSocketSettings=SocketSettings{connectTimeoutMS=10000, readTimeoutMS=10000, receiveBufferSize=0, sendBufferSize=0}, connectionPoolSettings=ConnectionPoolSettings{maxSize=100, minSize=0, maxWaitTimeMS=120000, maxConnectionLifeTimeMS=0, maxConnectionIdleTimeMS=0, maintenanceInitialDelayMS=0, maintenanceFrequencyMS=60000, connectionPoolListeners=[], maxConnecting=2}, serverSettings=ServerSettings{heartbeatFrequencyMS=10000, minHeartbeatFrequencyMS=500, serverListeners='[]', serverMonitorListeners='[]'}, sslSettings=SslSettings{enabled=true, invalidHostNameAllowed=false, context=null}, applicationName='null', compressorList=[], uuidRepresentation=UNSPECIFIED, serverApi=null, autoEncryptionSettings=null, contextProvider=null}

17:02:52.098 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-srv-wt-cluster-0.k0k8u7u.mongodb.net] INFO org.mongodb.driver.cluster - Adding discovered server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017 to client view of cluster

17:02:52.098 [com.mongodb.quickstart.Read.main()] INFO org.mongodb.driver.cluster - Cluster description not yet available. Waiting for 30000 ms before timing out

17:02:52.098 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-srv-wt-cluster-0.k0k8u7u.mongodb.net] INFO org.mongodb.driver.cluster - Adding discovered server ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017 to client view of cluster

17:02:52.099 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-srv-wt-cluster-0.k0k8u7u.mongodb.net] DEBUG org.mongodb.driver.cluster - Updating cluster description to  {type=REPLICA_SET, servers=[{address=ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017, type=UNKNOWN, state=CONNECTING}, {address=ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017, type=UNKNOWN, state=CONNECTING}, {address=ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017, type=UNKNOWN, state=CONNECTING}]

17:02:52.101 [com.mongodb.quickstart.Read.main()] INFO org.mongodb.driver.cluster - No server chosen by com.mongodb.client.internal.MongoClientDelegate$1@6ebc6027 from cluster description ClusterDescription{type=REPLICA_SET, connectionMode=MULTIPLE, serverDescriptions=[ServerDescription{address=ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017, type=UNKNOWN, state=CONNECTING}, ServerDescription{address=ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017, type=UNKNOWN, state=CONNECTING}, ServerDescription{address=ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017, type=UNKNOWN, state=CONNECTING}]}. Waiting for 30000 ms before timing out

17:02:52.336 [cluster-rtt-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.connection - Opened connection [connectionId{localValue:4, serverValue:57194}] to ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017

17:02:52.336 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.connection - Opened connection [connectionId{localValue:5, serverValue:56357}] to ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017

17:02:52.336 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.connection - Opened connection [connectionId{localValue:1, serverValue:60850}] to ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.336 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.connection - Opened connection [connectionId{localValue:6, serverValue:57195}] to ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017

17:02:52.336 [cluster-rtt-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.connection - Opened connection [connectionId{localValue:3, serverValue:60839}] to ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.336 [cluster-rtt-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.connection - Opened connection [connectionId{localValue:2, serverValue:56352}] to ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017

17:02:52.342 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017] INFO org.mongodb.driver.cluster - Monitor thread successfully connected to server with description ServerDescription{address=ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017, type=REPLICA_SET_SECONDARY, state=CONNECTED, ok=true, minWireVersion=0, maxWireVersion=17, maxDocumentSize=16777216, logicalSessionTimeoutMinutes=30, roundTripTimeNanos=154960666, setName='atlas-wdblrt-shard-0', canonicalAddress=ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017, hosts=[ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017, ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017, ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017], passives=[], arbiters=[], primary='ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017', tagSet=TagSet{[Tag{name='nodeType', value='ELECTABLE'}, Tag{name='provider', value='AWS'}, Tag{name='region', value='AP_SOUTHEAST_1'}, Tag{name='workloadType', value='OPERATIONAL'}]}, electionId=null, setVersion=7, topologyVersion=TopologyVersion{processId=64bb909f03b28dd82e11e4f9, counter=4}, lastWriteDate=Sun Jul 23 17:02:52 SGT 2023, lastUpdateTimeNanos=217466902985000}

17:02:52.342 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017] INFO org.mongodb.driver.cluster - Monitor thread successfully connected to server with description ServerDescription{address=ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017, type=REPLICA_SET_PRIMARY, state=CONNECTED, ok=true, minWireVersion=0, maxWireVersion=17, maxDocumentSize=16777216, logicalSessionTimeoutMinutes=30, roundTripTimeNanos=154983917, setName='atlas-wdblrt-shard-0', canonicalAddress=ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017, hosts=[ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017, ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017, ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017], passives=[], arbiters=[], primary='ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017', tagSet=TagSet{[Tag{name='nodeType', value='ELECTABLE'}, Tag{name='provider', value='AWS'}, Tag{name='region', value='AP_SOUTHEAST_1'}, Tag{name='workloadType', value='OPERATIONAL'}]}, electionId=7fffffff0000000000000081, setVersion=7, topologyVersion=TopologyVersion{processId=64bb91c320452214b3911605, counter=6}, lastWriteDate=Sun Jul 23 17:02:52 SGT 2023, lastUpdateTimeNanos=217466902988750}

17:02:52.342 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017] INFO org.mongodb.driver.cluster - Monitor thread successfully connected to server with description ServerDescription{address=ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017, type=REPLICA_SET_SECONDARY, state=CONNECTED, ok=true, minWireVersion=0, maxWireVersion=17, maxDocumentSize=16777216, logicalSessionTimeoutMinutes=30, roundTripTimeNanos=154955875, setName='atlas-wdblrt-shard-0', canonicalAddress=ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017, hosts=[ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017, ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017, ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017], passives=[], arbiters=[], primary='ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017', tagSet=TagSet{[Tag{name='nodeType', value='ELECTABLE'}, Tag{name='provider', value='AWS'}, Tag{name='region', value='AP_SOUTHEAST_1'}, Tag{name='workloadType', value='OPERATIONAL'}]}, electionId=null, setVersion=7, topologyVersion=TopologyVersion{processId=64bb92e777765998e3089984, counter=3}, lastWriteDate=Sun Jul 23 17:02:52 SGT 2023, lastUpdateTimeNanos=217466902988750}

17:02:52.343 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.connection - Marking the connection pool for ServerId{clusterId=ClusterId{value='64bcecbca52fed71a56c4132', description='null'}, address=ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017} as 'ready'

17:02:52.343 [MaintenanceTimer-4-thread-1] DEBUG org.mongodb.driver.connection - Pruning pooled connections to ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017

17:02:52.343 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.cluster - Updating cluster description to  {type=REPLICA_SET, servers=[{address=ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017, type=UNKNOWN, state=CONNECTING}, {address=ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017, type=UNKNOWN, state=CONNECTING}, {address=ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017, type=REPLICA_SET_SECONDARY, TagSet{[Tag{name='nodeType', value='ELECTABLE'}, Tag{name='provider', value='AWS'}, Tag{name='region', value='AP_SOUTHEAST_1'}, Tag{name='workloadType', value='OPERATIONAL'}]}, roundTripTime=155.0 ms, state=CONNECTED}]

17:02:52.343 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.cluster - Checking status of ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017

17:02:52.344 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.connection - Marking the connection pool for ServerId{clusterId=ClusterId{value='64bcecbca52fed71a56c4132', description='null'}, address=ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017} as 'ready'

17:02:52.344 [MaintenanceTimer-2-thread-1] DEBUG org.mongodb.driver.connection - Pruning pooled connections to ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017

17:02:52.344 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.cluster - Updating cluster description to  {type=REPLICA_SET, servers=[{address=ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017, type=REPLICA_SET_SECONDARY, TagSet{[Tag{name='nodeType', value='ELECTABLE'}, Tag{name='provider', value='AWS'}, Tag{name='region', value='AP_SOUTHEAST_1'}, Tag{name='workloadType', value='OPERATIONAL'}]}, roundTripTime=155.0 ms, state=CONNECTED}, {address=ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017, type=UNKNOWN, state=CONNECTING}, {address=ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017, type=REPLICA_SET_SECONDARY, TagSet{[Tag{name='nodeType', value='ELECTABLE'}, Tag{name='provider', value='AWS'}, Tag{name='region', value='AP_SOUTHEAST_1'}, Tag{name='workloadType', value='OPERATIONAL'}]}, roundTripTime=155.0 ms, state=CONNECTED}]

17:02:52.344 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.cluster - Checking status of ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017

17:02:52.344 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.connection - Marking the connection pool for ServerId{clusterId=ClusterId{value='64bcecbca52fed71a56c4132', description='null'}, address=ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017} as 'ready'

17:02:52.344 [MaintenanceTimer-3-thread-1] DEBUG org.mongodb.driver.connection - Pruning pooled connections to ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.344 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017] INFO org.mongodb.driver.cluster - Discovered replica set primary ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017 with max election id 7fffffff0000000000000081 and max set version 7

17:02:52.344 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.cluster - Updating cluster description to  {type=REPLICA_SET, servers=[{address=ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017, type=REPLICA_SET_SECONDARY, TagSet{[Tag{name='nodeType', value='ELECTABLE'}, Tag{name='provider', value='AWS'}, Tag{name='region', value='AP_SOUTHEAST_1'}, Tag{name='workloadType', value='OPERATIONAL'}]}, roundTripTime=155.0 ms, state=CONNECTED}, {address=ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017, type=REPLICA_SET_PRIMARY, TagSet{[Tag{name='nodeType', value='ELECTABLE'}, Tag{name='provider', value='AWS'}, Tag{name='region', value='AP_SOUTHEAST_1'}, Tag{name='workloadType', value='OPERATIONAL'}]}, roundTripTime=155.0 ms, state=CONNECTED}, {address=ac-bszzhwt-shard-00-00.k0k8u7u.mongodb.net:27017, type=REPLICA_SET_SECONDARY, TagSet{[Tag{name='nodeType', value='ELECTABLE'}, Tag{name='provider', value='AWS'}, Tag{name='region', value='AP_SOUTHEAST_1'}, Tag{name='workloadType', value='OPERATIONAL'}]}, roundTripTime=155.0 ms, state=CONNECTED}]

17:02:52.344 [cluster-ClusterId{value='64bcecbca52fed71a56c4132', description='null'}-ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017] DEBUG org.mongodb.driver.cluster - Checking status of ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.424 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.connection - Opened connection [connectionId{localValue:7, serverValue:60850}] to ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.429 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Sending command '{"find": "grades", "filter": {"student_id": 10000}, "limit": 1, "singleBatch": true, "$db": "sample_training", "lsid": {"id": {"$binary": {"base64": "VUiFpT8IQ3eh4OS+Cn2YCQ==", "subType": "04"}}}}' with request id 13 to database sample_training on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.500 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Execution of command with request id 13 completed successfully in 73.12 ms on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.502 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.operation - Received batch of 1 documents with cursorId 0 from server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

Student 1: {"_id": {"$oid": "64bce76d77d04766ae0279a6"}, "student_id": 10000.0, "class_id": 1.0, "scores": [{"type": "exam", "score": 14.079652517497598}, {"type": "quiz", "score": 70.22611310417194}, {"type": "homework", "score": 37.2895109669653}, {"type": "homework", "score": 46.66607584649181}]}

17:02:52.505 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Sending command '{"find": "grades", "filter": {"student_id": 10000}, "limit": 1, "singleBatch": true, "$db": "sample_training", "$clusterTime": {"clusterTime": {"$timestamp": {"t": 1690102972, "i": 5}}, "signature": {"hash": {"$binary": {"base64": "GQqir5kU3CUj+58UBzCIWLcHiBE=", "subType": "00"}}, "keyId": 7204913235105939459}}, "lsid": {"id": {"$binary": {"base64": "VUiFpT8IQ3eh4OS+Cn2YCQ==", "subType": "04"}}}}' with request id 14 to database sample_training on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.579 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Execution of command with request id 14 completed successfully in 74.00 ms on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.579 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.operation - Received batch of 1 documents with cursorId 0 from server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

Student 2: {"_id": {"$oid": "64bce76d77d04766ae0279a6"}, "student_id": 10000.0, "class_id": 1.0, "scores": [{"type": "exam", "score": 14.079652517497598}, {"type": "quiz", "score": 70.22611310417194}, {"type": "homework", "score": 37.2895109669653}, {"type": "homework", "score": 46.66607584649181}]}

17:02:52.581 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Sending command '{"find": "grades", "filter": {"student_id": {"$gte": 10000}}, "$db": "sample_training", "$clusterTime": {"clusterTime": {"$timestamp": {"t": 1690102972, "i": 6}}, "signature": {"hash": {"$binary": {"base64": "GQqir5kU3CUj+58UBzCIWLcHiBE=", "subType": "00"}}, "keyId": 7204913235105939459}}, "lsid": {"id": {"$binary": {"base64": "VUiFpT8IQ3eh4OS+Cn2YCQ==", "subType": "04"}}}}' with request id 15 to database sample_training on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.652 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Execution of command with request id 15 completed successfully in 71.46 ms on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.653 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.operation - Received batch of 11 documents with cursorId 0 from server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

Student list with a cursor: 
{"_id": {"$oid": "64bce76d77d04766ae0279a6"}, "student_id": 10000.0, "class_id": 1.0, "scores": [{"type": "exam", "score": 14.079652517497598}, {"type": "quiz", "score": 70.22611310417194}, {"type": "homework", "score": 37.2895109669653}, {"type": "homework", "score": 46.66607584649181}]}
{"_id": {"$oid": "64bce76d77d04766ae0279a7"}, "student_id": 10001.0, "class_id": 1.0, "scores": [{"type": "exam", "score": 76.96524467076867}, {"type": "quiz", "score": 11.345904851219856}, {"type": "homework", "score": 87.39046137565815}, {"type": "homework", "score": 92.05678952967556}]}
{"_id": {"$oid": "64bce76d77d04766ae0279a8"}, "student_id": 10001.0, "class_id": 2.0, "scores": [{"type": "exam", "score": 1.8242079171403525}, {"type": "quiz", "score": 67.81826548413564}, {"type": "homework", "score": 46.92395147869989}, {"type": "homework", "score": 3.569053699590341}]}
{"_id": {"$oid": "64bce76d77d04766ae0279a9"}, "student_id": 10001.0, "class_id": 3.0, "scores": [{"type": "exam", "score": 27.23629156867068}, {"type": "quiz", "score": 24.762607718023844}, {"type": "homework", "score": 40.096701904514106}, {"type": "homework", "score": 91.39630184044661}]}
{"_id": {"$oid": "64bce76d77d04766ae0279aa"}, "student_id": 10001.0, "class_id": 4.0, "scores": [{"type": "exam", "score": 15.581739914951031}, {"type": "quiz", "score": 46.20441634062177}, {"type": "homework", "score": 87.32081554284896}, {"type": "homework", "score": 63.03630481849956}]}
{"_id": {"$oid": "64bce76d77d04766ae0279ab"}, "student_id": 10001.0, "class_id": 5.0, "scores": [{"type": "exam", "score": 91.22267216324926}, {"type": "quiz", "score": 97.20144451555461}, {"type": "homework", "score": 90.28889498450266}, {"type": "homework", "score": 17.233689531337316}]}
{"_id": {"$oid": "64bce76d77d04766ae0279ac"}, "student_id": 10001.0, "class_id": 6.0, "scores": [{"type": "exam", "score": 16.021289255174544}, {"type": "quiz", "score": 68.6120816631519}, {"type": "homework", "score": 58.39142010546009}, {"type": "homework", "score": 15.041207989436046}]}
{"_id": {"$oid": "64bce76d77d04766ae0279ad"}, "student_id": 10001.0, "class_id": 7.0, "scores": [{"type": "exam", "score": 86.40205932698721}, {"type": "quiz", "score": 16.569444160278845}, {"type": "homework", "score": 25.966745021851466}, {"type": "homework", "score": 19.799725100634014}]}
{"_id": {"$oid": "64bce76d77d04766ae0279ae"}, "student_id": 10001.0, "class_id": 8.0, "scores": [{"type": "exam", "score": 98.7626864502663}, {"type": "quiz", "score": 51.47876004911579}, {"type": "homework", "score": 49.03421526601378}, {"type": "homework", "score": 59.653247345124285}]}
{"_id": {"$oid": "64bce76d77d04766ae0279af"}, "student_id": 10001.0, "class_id": 9.0, "scores": [{"type": "exam", "score": 70.76134870427505}, {"type": "quiz", "score": 37.234398099846054}, {"type": "homework", "score": 37.446081418249356}, {"type": "homework", "score": 81.15671598953026}]}
{"_id": {"$oid": "64bce76d77d04766ae0279b0"}, "student_id": 10001.0, "class_id": 10.0, "scores": [{"type": "exam", "score": 24.85530175324817}, {"type": "quiz", "score": 90.35688441920536}, {"type": "homework", "score": 73.80404539980042}, {"type": "homework", "score": 30.600737493610232}]}

17:02:52.658 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Sending command '{"find": "grades", "filter": {"student_id": {"$gte": 10000}}, "$db": "sample_training", "$clusterTime": {"clusterTime": {"$timestamp": {"t": 1690102972, "i": 6}}, "signature": {"hash": {"$binary": {"base64": "GQqir5kU3CUj+58UBzCIWLcHiBE=", "subType": "00"}}, "keyId": 7204913235105939459}}, "lsid": {"id": {"$binary": {"base64": "VUiFpT8IQ3eh4OS+Cn2YCQ==", "subType": "04"}}}}' with request id 16 to database sample_training on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.729 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Execution of command with request id 16 completed successfully in 71.29 ms on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.730 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.operation - Received batch of 11 documents with cursorId 0 from server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

Student list with an ArrayList:
{"_id": {"$oid": "64bce76d77d04766ae0279a6"}, "student_id": 10000.0, "class_id": 1.0, "scores": [{"type": "exam", "score": 14.079652517497598}, {"type": "quiz", "score": 70.22611310417194}, {"type": "homework", "score": 37.2895109669653}, {"type": "homework", "score": 46.66607584649181}]}
{"_id": {"$oid": "64bce76d77d04766ae0279a7"}, "student_id": 10001.0, "class_id": 1.0, "scores": [{"type": "exam", "score": 76.96524467076867}, {"type": "quiz", "score": 11.345904851219856}, {"type": "homework", "score": 87.39046137565815}, {"type": "homework", "score": 92.05678952967556}]}
{"_id": {"$oid": "64bce76d77d04766ae0279a8"}, "student_id": 10001.0, "class_id": 2.0, "scores": [{"type": "exam", "score": 1.8242079171403525}, {"type": "quiz", "score": 67.81826548413564}, {"type": "homework", "score": 46.92395147869989}, {"type": "homework", "score": 3.569053699590341}]}
{"_id": {"$oid": "64bce76d77d04766ae0279a9"}, "student_id": 10001.0, "class_id": 3.0, "scores": [{"type": "exam", "score": 27.23629156867068}, {"type": "quiz", "score": 24.762607718023844}, {"type": "homework", "score": 40.096701904514106}, {"type": "homework", "score": 91.39630184044661}]}
{"_id": {"$oid": "64bce76d77d04766ae0279aa"}, "student_id": 10001.0, "class_id": 4.0, "scores": [{"type": "exam", "score": 15.581739914951031}, {"type": "quiz", "score": 46.20441634062177}, {"type": "homework", "score": 87.32081554284896}, {"type": "homework", "score": 63.03630481849956}]}
{"_id": {"$oid": "64bce76d77d04766ae0279ab"}, "student_id": 10001.0, "class_id": 5.0, "scores": [{"type": "exam", "score": 91.22267216324926}, {"type": "quiz", "score": 97.20144451555461}, {"type": "homework", "score": 90.28889498450266}, {"type": "homework", "score": 17.233689531337316}]}
{"_id": {"$oid": "64bce76d77d04766ae0279ac"}, "student_id": 10001.0, "class_id": 6.0, "scores": [{"type": "exam", "score": 16.021289255174544}, {"type": "quiz", "score": 68.6120816631519}, {"type": "homework", "score": 58.39142010546009}, {"type": "homework", "score": 15.041207989436046}]}
{"_id": {"$oid": "64bce76d77d04766ae0279ad"}, "student_id": 10001.0, "class_id": 7.0, "scores": [{"type": "exam", "score": 86.40205932698721}, {"type": "quiz", "score": 16.569444160278845}, {"type": "homework", "score": 25.966745021851466}, {"type": "homework", "score": 19.799725100634014}]}
{"_id": {"$oid": "64bce76d77d04766ae0279ae"}, "student_id": 10001.0, "class_id": 8.0, "scores": [{"type": "exam", "score": 98.7626864502663}, {"type": "quiz", "score": 51.47876004911579}, {"type": "homework", "score": 49.03421526601378}, {"type": "homework", "score": 59.653247345124285}]}
{"_id": {"$oid": "64bce76d77d04766ae0279af"}, "student_id": 10001.0, "class_id": 9.0, "scores": [{"type": "exam", "score": 70.76134870427505}, {"type": "quiz", "score": 37.234398099846054}, {"type": "homework", "score": 37.446081418249356}, {"type": "homework", "score": 81.15671598953026}]}
{"_id": {"$oid": "64bce76d77d04766ae0279b0"}, "student_id": 10001.0, "class_id": 10.0, "scores": [{"type": "exam", "score": 24.85530175324817}, {"type": "quiz", "score": 90.35688441920536}, {"type": "homework", "score": 73.80404539980042}, {"type": "homework", "score": 30.600737493610232}]}

Student list using a Consumer:

17:02:52.734 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Sending command '{"find": "grades", "filter": {"student_id": {"$gte": 10000}}, "$db": "sample_training", "$clusterTime": {"clusterTime": {"$timestamp": {"t": 1690102972, "i": 7}}, "signature": {"hash": {"$binary": {"base64": "GQqir5kU3CUj+58UBzCIWLcHiBE=", "subType": "00"}}, "keyId": 7204913235105939459}}, "lsid": {"id": {"$binary": {"base64": "VUiFpT8IQ3eh4OS+Cn2YCQ==", "subType": "04"}}}}' with request id 17 to database sample_training on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.805 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Execution of command with request id 17 completed successfully in 71.27 ms on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.805 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.operation - Received batch of 11 documents with cursorId 0 from server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

{"_id": {"$oid": "64bce76d77d04766ae0279a6"}, "student_id": 10000.0, "class_id": 1.0, "scores": [{"type": "exam", "score": 14.079652517497598}, {"type": "quiz", "score": 70.22611310417194}, {"type": "homework", "score": 37.2895109669653}, {"type": "homework", "score": 46.66607584649181}]}
{"_id": {"$oid": "64bce76d77d04766ae0279a7"}, "student_id": 10001.0, "class_id": 1.0, "scores": [{"type": "exam", "score": 76.96524467076867}, {"type": "quiz", "score": 11.345904851219856}, {"type": "homework", "score": 87.39046137565815}, {"type": "homework", "score": 92.05678952967556}]}
{"_id": {"$oid": "64bce76d77d04766ae0279a8"}, "student_id": 10001.0, "class_id": 2.0, "scores": [{"type": "exam", "score": 1.8242079171403525}, {"type": "quiz", "score": 67.81826548413564}, {"type": "homework", "score": 46.92395147869989}, {"type": "homework", "score": 3.569053699590341}]}
{"_id": {"$oid": "64bce76d77d04766ae0279a9"}, "student_id": 10001.0, "class_id": 3.0, "scores": [{"type": "exam", "score": 27.23629156867068}, {"type": "quiz", "score": 24.762607718023844}, {"type": "homework", "score": 40.096701904514106}, {"type": "homework", "score": 91.39630184044661}]}
{"_id": {"$oid": "64bce76d77d04766ae0279aa"}, "student_id": 10001.0, "class_id": 4.0, "scores": [{"type": "exam", "score": 15.581739914951031}, {"type": "quiz", "score": 46.20441634062177}, {"type": "homework", "score": 87.32081554284896}, {"type": "homework", "score": 63.03630481849956}]}
{"_id": {"$oid": "64bce76d77d04766ae0279ab"}, "student_id": 10001.0, "class_id": 5.0, "scores": [{"type": "exam", "score": 91.22267216324926}, {"type": "quiz", "score": 97.20144451555461}, {"type": "homework", "score": 90.28889498450266}, {"type": "homework", "score": 17.233689531337316}]}
{"_id": {"$oid": "64bce76d77d04766ae0279ac"}, "student_id": 10001.0, "class_id": 6.0, "scores": [{"type": "exam", "score": 16.021289255174544}, {"type": "quiz", "score": 68.6120816631519}, {"type": "homework", "score": 58.39142010546009}, {"type": "homework", "score": 15.041207989436046}]}
{"_id": {"$oid": "64bce76d77d04766ae0279ad"}, "student_id": 10001.0, "class_id": 7.0, "scores": [{"type": "exam", "score": 86.40205932698721}, {"type": "quiz", "score": 16.569444160278845}, {"type": "homework", "score": 25.966745021851466}, {"type": "homework", "score": 19.799725100634014}]}
{"_id": {"$oid": "64bce76d77d04766ae0279ae"}, "student_id": 10001.0, "class_id": 8.0, "scores": [{"type": "exam", "score": 98.7626864502663}, {"type": "quiz", "score": 51.47876004911579}, {"type": "homework", "score": 49.03421526601378}, {"type": "homework", "score": 59.653247345124285}]}
{"_id": {"$oid": "64bce76d77d04766ae0279af"}, "student_id": 10001.0, "class_id": 9.0, "scores": [{"type": "exam", "score": 70.76134870427505}, {"type": "quiz", "score": 37.234398099846054}, {"type": "homework", "score": 37.446081418249356}, {"type": "homework", "score": 81.15671598953026}]}
{"_id": {"$oid": "64bce76d77d04766ae0279b0"}, "student_id": 10001.0, "class_id": 10.0, "scores": [{"type": "exam", "score": 24.85530175324817}, {"type": "quiz", "score": 90.35688441920536}, {"type": "homework", "score": 73.80404539980042}, {"type": "homework", "score": 30.600737493610232}]}

17:02:52.810 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Sending command '{"find": "grades", "filter": {"$and": [{"student_id": 10001}, {"class_id": {"$lte": 5}}]}, "sort": {"class_id": -1}, "projection": {"_id": 0, "class_id": 1, "student_id": 1}, "skip": 2, "limit": 2, "$db": "sample_training", "$clusterTime": {"clusterTime": {"$timestamp": {"t": 1690102972, "i": 9}}, "signature": {"hash": {"$binary": {"base64": "GQqir5kU3CUj+58UBzCIWLcHiBE=", "subType": "00"}}, "keyId": 7204913235105939459}}, "lsid": {"id": {"$binary": {"base64": "VUiFpT8IQ3eh4OS+Cn2YCQ==", "subType": "04"}}}}' with request id 18 to database sample_training on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.886 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Execution of command with request id 18 completed successfully in 76.26 ms on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

17:02:52.886 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.operation - Received batch of 2 documents with cursorId 0 from server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017

Student sorted, skipped, limited and projected: 
{"student_id": 10001.0, "class_id": 3.0}
{"student_id": 10001.0, "class_id": 2.0}
17:02:52.887 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Sending command '{"endSessions": [{"id": {"$binary": {"base64": "VUiFpT8IQ3eh4OS+Cn2YCQ==", "subType": "04"}}}], "$db": "admin", "$clusterTime": {"clusterTime": {"$timestamp": {"t": 1690102972, "i": 10}}, "signature": {"hash": {"$binary": {"base64": "GQqir5kU3CUj+58UBzCIWLcHiBE=", "subType": "00"}}, "keyId": 7204913235105939459}}, "$readPreference": {"mode": "primaryPreferred"}}' with request id 19 to database admin on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017
17:02:52.893 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.protocol.command - Execution of command with request id 19 completed successfully in 6.19 ms on connection [connectionId{localValue:7, serverValue:60850}] to server ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017
17:02:52.894 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.connection - Closing connection connectionId{localValue:5, serverValue:56357}
17:02:52.896 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.connection - Closing connection connectionId{localValue:2, serverValue:56352}
17:02:52.904 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.connection - Closed connection [connectionId{localValue:7, serverValue:60850}] to ac-bszzhwt-shard-00-01.k0k8u7u.mongodb.net:27017 because the pool has been closed.
17:02:52.904 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.connection - Closing connection connectionId{localValue:7, serverValue:60850}
17:02:52.905 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.connection - Closing connection connectionId{localValue:1, serverValue:60850}
17:02:52.905 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.connection - Closing connection connectionId{localValue:3, serverValue:60839}
17:02:52.913 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.connection - Closing connection connectionId{localValue:6, serverValue:57195}
17:02:52.913 [com.mongodb.quickstart.Read.main()] DEBUG org.mongodb.driver.connection - Closing connection connectionId{localValue:4, serverValue:57194}
```

### 4 - Update operations of 1 and multiple documents, upsert and findOneAndUpdate of 1 document
1.  Java code
```
package com.mongodb.quickstart;

import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.model.FindOneAndUpdateOptions;
import com.mongodb.client.model.ReturnDocument;
import com.mongodb.client.model.UpdateOptions;
import com.mongodb.client.result.UpdateResult;
import org.bson.Document;
import org.bson.conversions.Bson;
import org.bson.json.JsonWriterSettings;

import static com.mongodb.client.model.Filters.and;
import static com.mongodb.client.model.Filters.eq;
import static com.mongodb.client.model.Updates.*;

public class Update {

    public static void main(String[] args) {
        JsonWriterSettings prettyPrint = JsonWriterSettings.builder().indent(true).build();

        try (MongoClient mongoClient = MongoClients.create(System.getProperty("mongodb.uri"))) {
            MongoDatabase sampleTrainingDB = mongoClient.getDatabase("sample_training");
            MongoCollection<Document> gradesCollection = sampleTrainingDB.getCollection("grades");

            // update one document
            Bson filter = eq("student_id", 10000);
            Bson updateOperation = set("comment", "You should learn MongoDB!");
            UpdateResult updateResult = gradesCollection.updateOne(filter, updateOperation);
            System.out.println("=> Updating the doc with {\"student_id\":10000}. Adding comment.");
            System.out.println(gradesCollection.find(filter).first().toJson(prettyPrint));
            System.out.println(updateResult);

            // upsert
            filter = and(eq("student_id", 10002d), eq("class_id", 10d));
            updateOperation = push("comments", "You will learn a lot if you read the MongoDB blog!");
            UpdateOptions options = new UpdateOptions().upsert(true);
            updateResult = gradesCollection.updateOne(filter, updateOperation, options);
            System.out.println("\n=> Upsert document with {\"student_id\":10002.0, \"class_id\": 10.0} because it doesn't exist yet.");
            System.out.println(updateResult);
            System.out.println(gradesCollection.find(filter).first().toJson(prettyPrint));

            // update many documents
            filter = eq("student_id", 10001);
            updateResult = gradesCollection.updateMany(filter, updateOperation);
            System.out.println("\n=> Updating all the documents with {\"student_id\":10001}.");
            System.out.println(updateResult);

            // findOneAndUpdate
            filter = eq("student_id", 10000);
            Bson update1 = inc("x", 10); // increment x by 10. As x doesn't exist yet, x=10.
            Bson update2 = rename("class_id", "new_class_id"); // rename variable "class_id" in "new_class_id".
            Bson update3 = mul("scores.0.score", 2); // multiply the first score in the array by 2.
            Bson update4 = addToSet("comments", "This comment is uniq"); // creating an array with a comment.
            Bson update5 = addToSet("comments", "This comment is uniq"); // using addToSet so no effect.
            Bson updates = combine(update1, update2, update3, update4, update5);
            // returns the old version of the document before the update.
            Document oldVersion = gradesCollection.findOneAndUpdate(filter, updates);
            System.out.println("\n=> FindOneAndUpdate operation. Printing the old version by default:");
            System.out.println(oldVersion.toJson(prettyPrint));

            // but I can also request the new version
            filter = eq("student_id", 10001);
            FindOneAndUpdateOptions optionAfter = new FindOneAndUpdateOptions().returnDocument(ReturnDocument.AFTER);
            Document newVersion = gradesCollection.findOneAndUpdate(filter, updates, optionAfter);
            System.out.println("\n=> FindOneAndUpdate operation. But we can also ask for the new version of the doc:");
            System.out.println(newVersion.toJson(prettyPrint));
        }
    }
}
```

2.  Run this example command/class to execute:
```
mvn compile exec:java -Dexec.mainClass="com.mongodb.quickstart.Update" -Dmongodb.uri="mongodb+srv://USERNAME:PASSWORD@wt-cluster-0.k0k8u7u.mongodb.net/?retryWrites=true&w=majority" -Dexec.cleanupDaemonThreads=false
```

3.  Output example:
```
=> Updating the doc with {"student_id":10000}. Adding comment.
23:11:27.611 [com.mongodb.quickstart.Update.main()] DEBUG org.mongodb.driver.protocol.command - Sending command '{"find": "grades", "filter": {"student_id": 10000}, "limit": 1, "singleBatch": true, "$db": "sample_training", "$clusterTime": {"clusterTime": {"$timestamp": {"t": 1690211487, "i": 7}}, "signature": {"hash": {"$binary": {"base64": "msM3OL51kemms3RcwSxj8a1RjGI=", "subType": "00"}}, "keyId": 7204913235105939459}}, "lsid": {"id": {"$binary": {"base64": "ei7XXk4eSMqBQECoJQ+C8Q==", "subType": "04"}}}}' with request id 14 to database sample_training on connection [connectionId{localValue:7, serverValue:4057}] to server ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017
23:11:27.689 [com.mongodb.quickstart.Update.main()] DEBUG org.mongodb.driver.protocol.command - Execution of command with request id 14 completed successfully in 78.55 ms on connection [connectionId{localValue:7, serverValue:4057}] to server ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017
23:11:27.694 [com.mongodb.quickstart.Update.main()] DEBUG org.mongodb.driver.operation - Received batch of 1 documents with cursorId 0 from server ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017
{
  "_id": {
    "$oid": "64bce76d77d04766ae0279a6"
  },
  "student_id": 10000.0,
  "class_id": 1.0,
  "scores": [
    {
      "type": "exam",
      "score": 14.079652517497598
    },
    {
      "type": "quiz",
      "score": 70.22611310417194
    },
    {
      "type": "homework",
      "score": 37.2895109669653
    },
    {
      "type": "homework",
      "score": 46.66607584649181
    }
  ],
  "comment": "You should learn MongoDB!"
}
AcknowledgedUpdateResult{matchedCount=1, modifiedCount=1, upsertedId=null}

=> Upsert document with {"student_id":10002.0, "class_id": 10.0} because it doesn't exist yet.
AcknowledgedUpdateResult{matchedCount=0, modifiedCount=0, upsertedId=BsonObjectId{value=64be949f17c938c272f6bc8b}}
23:11:27.820 [com.mongodb.quickstart.Update.main()] DEBUG org.mongodb.driver.protocol.command - Sending command '{"find": "grades", "filter": {"$and": [{"student_id": 10002.0}, {"class_id": 10.0}]}, "limit": 1, "singleBatch": true, "$db": "sample_training", "$clusterTime": {"clusterTime": {"$timestamp": {"t": 1690211487, "i": 13}}, "signature": {"hash": {"$binary": {"base64": "msM3OL51kemms3RcwSxj8a1RjGI=", "subType": "00"}}, "keyId": 7204913235105939459}}, "lsid": {"id": {"$binary": {"base64": "ei7XXk4eSMqBQECoJQ+C8Q==", "subType": "04"}}}}' with request id 16 to database sample_training on connection [connectionId{localValue:7, serverValue:4057}] to server ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017
23:11:27.902 [com.mongodb.quickstart.Update.main()] DEBUG org.mongodb.driver.protocol.command - Execution of command with request id 16 completed successfully in 82.20 ms on connection [connectionId{localValue:7, serverValue:4057}] to server ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017
23:11:27.902 [com.mongodb.quickstart.Update.main()] DEBUG org.mongodb.driver.operation - Received batch of 1 documents with cursorId 0 from server ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017
{
  "_id": {
    "$oid": "64be949f17c938c272f6bc8b"
  },
  "class_id": 10.0,
  "student_id": 10002.0,
  "comments": [
    "You will learn a lot if you read the MongoDB blog!"
  ]
}

=> Updating all the documents with {"student_id":10001}.
AcknowledgedUpdateResult{matchedCount=10, modifiedCount=10, upsertedId=null}
23:11:28.180 [com.mongodb.quickstart.Update.main()] DEBUG org.mongodb.driver.protocol.command - Sending command '{"findAndModify": "grades", "query": {"student_id": 10000}, "new": false, "update": {"$inc": {"x": 10}, "$rename": {"class_id": "new_class_id"}, "$mul": {"scores.0.score": 2}, "$addToSet": {"comments": "This comment is uniq"}}, "writeConcern": {"w": "majority"}, "txnNumber": 3, "$db": "sample_training", "$clusterTime": {"clusterTime": {"$timestamp": {"t": 1690211488, "i": 11}}, "signature": {"hash": {"$binary": {"base64": "NNAxEXWAP3Z7m4ratELp9s/+kkw=", "subType": "00"}}, "keyId": 7204913235105939459}}, "lsid": {"id": {"$binary": {"base64": "ei7XXk4eSMqBQECoJQ+C8Q==", "subType": "04"}}}}' with request id 18 to database sample_training on connection [connectionId{localValue:7, serverValue:4057}] to server ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017
23:11:28.277 [com.mongodb.quickstart.Update.main()] DEBUG org.mongodb.driver.protocol.command - Execution of command with request id 18 completed successfully in 96.98 ms on connection [connectionId{localValue:7, serverValue:4057}] to server ac-bszzhwt-shard-00-02.k0k8u7u.mongodb.net:27017

=> FindOneAndUpdate operation. Printing the old version by default:
{
  "_id": {
    "$oid": "64bce76d77d04766ae0279a6"
  },
  "student_id": 10000.0,
  "class_id": 1.0,
  "scores": [
    {
      "type": "exam",
      "score": 14.079652517497598
    },
    {
      "type": "quiz",
      "score": 70.22611310417194
    },
    {
      "type": "homework",
      "score": 37.2895109669653
    },
    {
      "type": "homework",
      "score": 46.66607584649181
    }
  ],
  "comment": "You should learn MongoDB!"
}

=> FindOneAndUpdate operation. But we can also ask for the new version of the doc:
{
  "_id": {
    "$oid": "64bce76d77d04766ae0279ad"
  },
  "student_id": 10001.0,
  "scores": [
    {
      "type": "exam",
      "score": 172.80411865397443
    },
    {
      "type": "quiz",
      "score": 16.569444160278845
    },
    {
      "type": "homework",
      "score": 25.966745021851466
    },
    {
      "type": "homework",
      "score": 19.799725100634014
    }
  ],
  "comments": [
    "You will learn a lot if you read the MongoDB blog!",
    "This comment is uniq"
  ],
  "new_class_id": 7.0,
  "x": 10
}
```

### 5 - Delete operations of deleting 1 document, FindOneAndDelete() i.e. retrieve 1 doc and delete it in a single atomic operation, deleting multiple docs and deleting a collection using drop() method
1.  Java code
```
package com.mongodb.quickstart;

import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.result.DeleteResult;
import org.bson.Document;
import org.bson.conversions.Bson;
import org.bson.json.JsonWriterSettings;

import static com.mongodb.client.model.Filters.eq;
import static com.mongodb.client.model.Filters.gte;

public class Delete {

    public static void main(String[] args) {
        try (MongoClient mongoClient = MongoClients.create(System.getProperty("mongodb.uri"))) {
            MongoDatabase sampleTrainingDB = mongoClient.getDatabase("sample_training");
            MongoCollection<Document> gradesCollection = sampleTrainingDB.getCollection("grades");

            // delete one document
            Bson filter = eq("student_id", 10000);
            DeleteResult result = gradesCollection.deleteOne(filter);
            System.out.println(result);

            // findOneAndDelete operation
            filter = eq("student_id", 10002);
            Document doc = gradesCollection.findOneAndDelete(filter);
            System.out.println(doc.toJson(JsonWriterSettings.builder().indent(true).build()));

            // delete many documents
            filter = gte("student_id", 10000);
            result = gradesCollection.deleteMany(filter);
            System.out.println(result);

            // delete the entire collection and its metadata (indexes, chunk metadata, etc).
            gradesCollection.drop();
        }
    }
}
```

2.  Run this example command/class to execute:
```
mvn compile exec:java -Dexec.mainClass="com.mongodb.quickstart.Delete" -Dmongodb.uri="mongodb+srv://USERNAME:PASSWORD@wt-cluster-0.k0k8u7u.mongodb.net/?retryWrites=true&w=majority" -Dexec.cleanupDaemonThreads=false
```

3.  Output example:
```
```

### X - Topics
1.  Java code
```
```

2.  Run this example command/class to execute:
```
```

3.  Output example:
```
```
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
    - MongoDB Atlas connection string:  ```mongodb+srv://<admin_id>:<password>@wt-cluster-0.k0k8u7u.mongodb.net/?retryWrites=true&w=majority```
      
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

### 2 - Insert Operations of 1 and many documents
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

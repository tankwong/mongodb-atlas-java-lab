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
1.  Java code
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

### 2 - Insert Operations
1.  Java code
```
```

2.  Run this example command/class to execute:
```
```

3.  Output example:
```
```
### X - <Topics>
1.  Java code
```
```

2.  Run this example command/class to execute:
```
```

3.  Output example:
```
```

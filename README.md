# Object Store Reference Application
Object Store reference application is built to showcase the method of developing a single code-line multi cloud application to consume Object Store service of SAP Cloud Platform Cloud Foundry Environment.

## Description
Object Store service enables the storage and management of objects, which involves creation, upload, download, and deletion of objects. SAP provides Object Store Service on its SAP Cloud Platform Cloud Foundry environment running on different IaaS providers like Amazon Web Service, Azure, Google Cloud Platform, OpenStack. [Please click here for more information on Object Store Service](https://help.sap.com/viewer/2ee77ef7ea4648f9ab2c54ee3aef0a29/Cloud/en-US/9f82aa99c6fb443495495a67b8e0f924.html).

Though the Object Store Service is provided by SAP on multiple IaaS providers, the way to connect to and use the service varies for each IaaS provider due to changes in the structure of credentials and in the configurations. Writing a single code-line application that works seamlessly on all these IaaS providers is a challenge that many developers face. 

We have developed a single code line reference application that can work with Object Store Service on SAP Cloud Platform Cloud Foundry Environment hosted on multiple IaaS providers . This application perform operations like upload, download, delete and list of files. It is a spring boot application that uses [Apache jclouds](https://jclouds.apache.org/) library which provides a multi-cloud toolkit that gives a developer the freedom to create applications that are portable across IaaS providers.

#### Features of the Application

•	The application provides RESTful endpoints to upload, download, delete and list files.

•	It calls jclouds library's API enpoints to perform the above operations on the files in Object Store Service. jclouds abstracts the code to perform these operation on the different providers like AWS S3, Google Cloud Storage, OpenStack Swift.

## Architecture

![Alt text](./documents/objectstore-sample-architecture.jpg "Architecture")

A single REST controller accepts the request (GET, POST, DELETE).

Separate service implementations and configuration classes are provided for each of the Object Store Service provider. The right service implementation and configuration is loaded by spring boot based on the IaaS provider that the application is deployed on. 

A single DAO (Data Access Object)/ repository class calls the jclouds api’s to perform upload, download, delete operations on the Object Store. 

## Referenced Libraries
Following jclouds dependencies are used in the application.

~~~
    <!-- jclouds dependencies -->
    <dependency>
      <groupId>org.apache.jclouds.provider</groupId>
      <artifactId>aws-s3</artifactId>
      <version>${jclouds.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.jclouds.provider</groupId>
      <artifactId>google-cloud-storage</artifactId>
      <version>${jclouds.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.jclouds.api</groupId>
      <artifactId>openstack-swift</artifactId>
      <version>${jclouds.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.jclouds.api</groupId>
      <artifactId>openstack-keystone</artifactId>
      <version>${jclouds.version}</version>
    </dependency>
~~~
The size of each of the jclouds dependencies are as follows: 
~~~
    Dependency                Size
    -------------             -------
    aws-s3                    29kb
    google-cloud-storage      158kb
    openstack-swift           137kb
    openstack-keystone        294kb
~~~

Besides spring-boot and jclouds the other dependencies used in the application are: 

•	jackson-databind: to parse json file

•	guava: to build BlobStoreContext and to convert input stream to byte array 

•	commons-fileupload: to upload files 

For more information about the dependencies please refer [pom.xml file](./pom.xml).


## Requirements
- [Java 8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
- [Apache Maven 3.3+](https://maven.apache.org/download.cgi)
- [Cloud Foundry CLI](https://github.com/cloudfoundry/cli#downloads)
- [SAP Cloud Platform Global account](https://help.sap.com/viewer/e275296cbb1e4d5886fa38a2a2c78c06/Cloud/en-US/667f34ba9222450491c2b848cd17e189.html)
- [Provision Object Store service](https://cloudplatform.sap.com/capabilities/product-info.Object-Store-on-SAP-Cloud-Platform.55e3958b-e872-49e3-8d2c-8634d200c36a.html). It is a paid service on Cloud Foundry.
- A Cloud Foundry user with SpaceDeveloper role to deploy the application

## Download and Installation

#### Build the Application
- [Clone](https://help.github.com/articles/cloning-a-repository/) the application `objectstore-sample` to your system

   Clone URL :  `https://github.wdf.sap.corp/refapps/objectstore-sample.git`
- Navigate to the root folder of the application and run the below maven command to build the application:
  ```
  mvn clean install
  ```

#### Deploy the Application on Cloud Foundry

  1. Logon to the Cloud Foundry environment using the following commands on the command prompt:
     ```
     cf api <api>
     cf login
     ```
     `api` - [URL of the Cloud Foundry landscape](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/350356d1dc314d3199dca15bd2ab9b0e.html) that you are trying to connect to.
        
     Enter username, password, org and space when prompted to. [Please click here for more information](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/75125ef1e60e490e91eb58fe48c0f9e7.html#loio4ef907afb1254e8286882a2bdef0edf4).
    

  2. Create the Cloud Foundry Object Store Service Instance

     - To run the application on AWS landscape, create a service by executing the below command:

       `cf create-service objectstore s3-standard objectstore-service`

     - To run the application on GCP landscape, create a service by executing the below command:

       `cf create-service objectstore gcs-standard objectstore-service`
        
     - To run the application on OpenStack landscape, create a service by executing the below command:

       `cf create-service objectstore swift-standard objectstore-service`

      > Important: <b>*Please don't change the service instance name i.e. `objectstore-service`*</b>

  3. Edit manifest.yml file. Replace the `<unique_id>` placeholder with any unique string. You can use your *SAP User ID* so that the host name is unique in the CF landscape. You can find your *SAP User ID* in [your sap.com profile](https://people.sap.com/#personal_info).

  ~~~
    ---
    applications:
    - name: objectstore-sample-svc
     ------------------------------------------
    | host: <unique_id>-objectstore-sample-svc |
     ------------------------------------------
      memory: 2G
      buildpack: https://github.com/cloudfoundry/java-buildpack.git
      path: target/objectstore-sample-1.0.0.jar
      services:
        - objectstore-service
  ~~~

  4. To deploy the application, navigate to the root of the application and execute the below command:
     ```
     cf push
     ```

#### Test the Application

[Postman Client](https://www.getpostman.com/apps) can be used to test / access the REST API endpoints.

Replace the `<application URL>` placeholder in the below steps with the URL of the application you deployed. 

##### Upload a file / object

<b>POST</b>

To upload a file / object set the below request body and hit the endpoint url.

EndPoint URL :   `<application URL>/objectstorage.svc/api/v1/storage/`

Request Body : form-data with key-value pair. Pass the name of the key as `file` and the value is `the path of the file`.

> For the file upload, we have provided a [test file](/documents/test.rtf) in the documents folder which you can use if needed for the upload testing.

![Alt text](./documents/postRequest.png "post request")

A successful upload operation gives the following response : 

Status: 202

Response Body: `<uploaded_filename> uploaded successfully`


##### List all the files / objects

<b>GET</b>

To get the list of a files / objects set the content-type and hit the below endpoint url.

EndPoint URL :   `https://<application URL>/objectstorage.svc/api/v1/storage/`

Content-Type : `application/json`

A successful upload operation gives the following response :

Status: 200

Response Body:
~~~
[
    {
        "etag": "CIjak4uDxeACEAE=",
        "bucket": "sap-cp-osaas-a78345d3-e45d-42eb-9c03-c47393d0d436",
        "name": "SampleFile.pdf",
        "url": "https://www.googleapis.com/storage/v1/b/sap-cp-osaas-a78345d3-e45d-42eb-9c03-c47393d0d436/o/SampleFile.pdf",
        "lastModified": "Mon Feb 18 15:30:22 IST 2019",
        "size": "245.7 KB",
        "contentType": "application/pdf",
        "userMetadata": {
            "description": "sample content"
        }
    },
    {
        "etag": "COf+0p7uxOACEAE=",
        "bucket": "sap-cp-osaas-a78345d3-e45d-42eb-9c03-c47393d0d436",
        "name": "SampleImage.jpg",
        "url": "https://www.googleapis.com/storage/v1/b/sap-cp-osaas-a78345d3-e45d-42eb-9c03-c47393d0d436/o/SampleImage.jpg",
        "lastModified": "Mon Feb 18 13:57:06 IST 2019",
        "size": "46.1 KB",
        "contentType": "image/jpeg",
        "userMetadata": {
            "description": "sample content"
        }
    }
  ...

]
~~~

##### Download a file / object

<b>GET</b>

> Please open any browser and hit the below endpoint url to download a file / object rather than using Postman to test it.</b>

EndPoint URL :   `https://<application URL>/objectstorage.svc/api/v1/storage/{file-name}`

##### Delete a file / object

<b>DELETE</b>

To delete a file / object hit the below endpoint url by appending the file / object name in postman.

EndPoint URL :   `https://<application URL>/objectstorage.svc/api/v1/storage/{file-name}`

A successful upload operation gives the following response :

Status: 200

Response Body: `<file-name> deleted from ObjectStore.`

## How to Obtain Support

In case you find a bug, or you need additional support, please open an issue here in GitHub.

## Known Issues
- BlobStore removeBlob() api does not work in Jclouds OpenStack Swift and a bug is reported to jclouds team. For more information see [here](https://issues.apache.org/jira/browse/JCLOUDS-1483).
- Multipart Upload feature is not supported in jclouds OpenStack Swift and a bug is reported to jclouds team. For more information see [here](https://issues.apache.org/jira/browse/JCLOUDS-1482).
- Using InpuStream instead of ByteArray to upload large files to make the read operation faster causes issues. For more information see [here](https://issues.apache.org/jira/browse/JCLOUDS-1451). 

## TO-DO
- To provide support for Azure Object Store

## License

Copyright (c) 2019 SAP SE or an SAP affiliate company. All rights reserved. This file is licensed under SAP Sample Code License Agreement, except as noted otherwise in the [LICENSE](/LICENSE) file.

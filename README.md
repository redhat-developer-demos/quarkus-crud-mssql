
# Developing Quarkus App consuming Microsoft SQL Server running on Red Hat Enterprise Linux

![](https://lh4.googleusercontent.com/LR2qID0eSPLTJKIBoRVLoQReo61KGWLlxot4PXxNOkgaB9rL06aktF5QAk8SjsXpWd9Ii1C6uA3fQQL5iGK-Ob6S2UK0aHLe0yvSl4voBOyMP78WVReMrC3Y50woojss1VBinmgyJPszu10glH41am_-S3UVb_XzQmSgdO0tfaKCJX_dLojXSs8)

  
  

### Introduction

Modern businesses need a solid data platform to quickly process large volumes of data and meet

growing operational and analytical workloads. Microsoft SQL Server on Red Hat Enterprise Linux offers businesses additional flexibility, superior performance, enhanced security, and ultra-high availability. This combination provides a scalable foundation with a consistent application experience, whether deployed across bare-metal, virtual machine, container, or cloud environments.

  

In this blog post, you will be learning how to implement [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) application running on RHEL Podman container engine which implements CRUD API functionality by connecting to Microsoft SQL Server database running on Red Hat Enterprise Linux.

  

### Pre-requisites

  

-   RHEL9 (or RHEL8) installed on your local system.
    
-   [Open JDK 11](https://access.redhat.com/documentation/en-us/openjdk/11/html/installing_and_using_openjdk_11_on_rhel/installing-openjdk11-on-rhel8).
    
-   [Quarkus 2.1+](https://quarkus.io/get-started/).
    
-   [Quarkus CLI](https://quarkus.io/guides/cli-tooling).
    
-   [Maven](https://tecadmin.net/install-apache-maven-on-centos/).
    
-   [Podman](https://podman.io/getting-started/installation).
    

  

### Table of content:

  

1. Microsoft SQL serverInstalltion & Setup

    1.1. Install Microsoft SQL Server

   1.2. Installation of Microsoft SQL CLI tool

   1.3. Connect Database locally

  

2. Quarkus Application setup

   2.1. Quarkus Introduction.

   2.2. Qurkus Project Create.

   2.3. CRUD Application Flow Diagram.

  

3. Quarkus App API Connectivity with Database

   3.1. Person.java & Person.java files add

   3.2. Application.properties file

   3.3. Run the application

  

4. Containerise the Quarkus App using Podman.

   4.1 building dockerfile

   4.2 Run quarkus container

   4.3 TEST

  

5. Summary

  

6. Reference links
  

## 1. Microsoft SQL Server Installation & Setup
    

To begin with install Microsoft SQL Server on RHEL (locally), here are the steps to do it.

  
  

### 1.1. Install Microsoft SQL Server

  

1.  Download SQL Server 2022 Preview Red Hat repository configuration file.
    

  

``` sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/8/mssql-server-preview.repo ```

  

2.  Install MSsql server.
    

  

```sudo yum install -y mssql-server ```

  

3.  MSsql-conf setup using its full path and follow the prompts to set the SA password and choose your edition.
    

  

``` sudo /opt/mssql/bin/mssql-conf setup```

  

4.  Verify whether the MSsql service is running or not.
    

  

``` systemctl status mssql-server ```

  

5.  open the SQL Server port on the RHEL firewall. The default SQL Server port is TCP 1433. If you're using FirewallD for your firewall.
    

  

```sudo firewall-cmd --zone=public --add-port=1433/tcp --permanent  ```
```sudo firewall-cmd --reload```

SQL Server is now running on your RHEL machine and is ready to use!

  

### 1.2. Installation of Microsoft SQL CLI tool

  

The following steps are to install the SQL Server command line tool that is an [sqlcmd](https://learn.microsoft.com/en-us/sql/tools/sqlcmd-utility?view=sql-server-ver16).

1. Download the Red Hat repository configuration file.

  

```sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/8/prod.repo```

  

2.Install mssql-tools with the unixODBC developer package

  

```sudo yum install -y mssql-tools unixODBC-devel```

  

3. add /opt/mssql-tools/bin/ to your PATH environment variable, to make sqlcmd or bcp accessible from the bash shell. Modify the PATH environment variable in your ~/.bash_profile file with the following command:

  

```echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile```

For non-interactive sessions, modify the PATH environment variable in your ~/.bashrc file with the following command:

```echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc  ```
```source ~/.bashrc```

  
  

## 1.3. Connect to Database locally

1.  Run the sqlcmd with parameters for your SQL Server IP/name (-S), username (-U), and password (-P). In this tutorial, you are connecting locally, so the server name is localhost. The user name is sa and the password is the one you provided for the SA account during setup.
    

  

```sqlcmd -S localhost -U sa -P '<YourPassword>' ```

  

2.  Create a new Database
    

``` CREATE DATABASE TestDB; ```

  

3.  write a query to return the name of all of the databases on the server.
    

``` SELECT Name from sys.databases; ```

  

4.  To execute the above 2 queries write GO
    

```GO```

  
  

## 2. Quarkus Application Setup

### 2.1. Quarkus Introduction

Quarkus was invented to enable Java developers to create applications for a modern, cloud-native world. Quarkus is a Kubernetes-native Java framework tailored for GraalVM and HotSpot, crafted from best-of-breed Java libraries and standards. The goal is to make Java the leading platform in Kubernetes and serverless environments while offering developers a framework to address a wider range of distributed application architectures.

![](https://lh4.googleusercontent.com/fbuLYN-7f0_biylORqajOIuWRRnIWuVqTMjVIGprjdMbJYHt0eCfb2qddyZfourTanduLpSH9vXXU_sGMto-0-845mP4kvzu28ztMNDGTigqFz3NaW81VCpW3Icg6O_r9cbrUVriNsp4VTVhCs6OR3QodmVW3ac8K-HVvYAJ-2_5AxKkecwJJczi)

  

### 2.2. Qurkus Project Create.

Now let’s create quarkus project with help of quarkus cli.

quarkus create app org.acme:quarkus-crud-app --extension=agroal,resteasy-reactive, quarkus-hibernate-orm-panache,quarkus-resteasy-jsonb ,quarkus-jdbc-mssql

  

After running the above command one ‘quarkus-crud-app’ directory may be created & you can see a lot of line print on your terminal.

  

Also, we have added a few extensions which we required during our crud application.

  

As per our CRUD Application requirement, we have to create GET, POST, PUT, and DELETE API from our Microsoft SQL database. For that, we need to consider one variable which would be ‘Person’.

  

### 2.3. CRUD Application Flow Diagram

  
  
  

![](https://lh3.googleusercontent.com/4DAel_R4Nh1NcnC4UMDcXUjVYSct7i7H_RIete7eZZlayvzJyRcyar8GrWORAOdbeRwPMbMPd2EgPgqxoNZkBrN60GtbuJiB1KmfLr-1T74AcD5taHcJjK3cSAYYKmgenELK0LVll5-FlswkTJEJNMly6RsIzY_meqk8LjBhXeTzLPhK4m4xH94a)

  

## 3. Quarkus App API Connectivity with Database

You can find the source code with instructions used in this blog post in this [GitHub](https://github.com/redhat-developer-demos/quarkus-crud-mssql) repository

### 3.1. Person.java & Person.java files add

  

Let’s call that entity Person, so you’re going to create a Person.java file. at ‘/quarkus-crud-app/src/main/java/org/acme/Person.java’ location.

```
package org.acme;  
  
import javax.persistence.Column;  
import javax.persistence.Entity;  
  
import io.quarkus.hibernate.orm.panache.PanacheEntity;  
  
@Entity  
public class  Person  extends  PanacheEntity {  
@Column(name="first_name")  
public String firstName;  
  
@Column(name="last_name")  
public String lastName;  
  
public String salutation;  
}
```
  

The second file we need to create is a PersonResponce.java at ‘/quarkus-crud-app/src/main/java/

org/acme/PersonResponce.java’ location in this file we import multiple packages which are required during our api functionality. And in this file we include GET, POST, PUT and DELETE API.

  
```
package org.acme;  
  
import java.util.List;  
  
import javax.transaction.Transactional;  
import javax.ws.rs.Consumes;  
import javax.ws.rs.GET;  
import javax.ws.rs.POST;  
import javax.ws.rs.PUT;  
import javax.ws.rs.DELETE;  
import javax.ws.rs.Path;  
import javax.ws.rs.PathParam;  
import javax.ws.rs.Produces;  
import javax.ws.rs.WebApplicationException;  
import javax.ws.rs.core.MediaType;  
import javax.ws.rs.core.Response;  
  
import io.quarkus.panache.common.Sort;  
  
@Path("/person")  
@Consumes(MediaType.APPLICATION_JSON)  
@Produces(MediaType.APPLICATION_JSON)  
public  class  PersonResource {  
  
@GET  
public List<Person> getAll() throws Exception {  
return Person.findAll(Sort.ascending("last_name")).list();  
}  
  
  
@POST  
@Transactional  
public Response create(Person p) {  
if (p == null || p.id != null)  
throw new WebApplicationException("id != null");  
p.persist();  
return Response.ok(p).status(200).build();  
}  
  
  
  
@PUT  
@Transactional  
@Path("/{id}")  
public Person update(@PathParam("id")  Long id, Person p) {  
Person entity = Person.findById(id);  
if (entity == null) {  
throw new WebApplicationException("Person with id of " + id + " does not exist.", 404);  
}  
if(p.salutation != null ) entity.salutation = p.salutation;  
if(p.firstName != null ) entity.firstName = p.firstName;  
if(p.lastName != null) entity.lastName = p.lastName;  
return entity;  
}  
  
@DELETE  
@Path("/{id}")  
@Transactional  
public Response delete(@PathParam("id")  Long id) {  
Person entity = Person.findById(id);  
if (entity == null) {  
throw new WebApplicationException("Person with id of " + id + " does not exist.", 404);  
}  
entity.delete();  
return Response.status(204).build();  
}  
}
```
  

### 3.2. Application.properties file

Application.properties is a configuration/environment file of our application in which we add multiple parameters to connect with the Microsoft SQL database from quarks application like username, password, endpoint, size, encrypted-connection, etc.,

  
```
quarkus.datasource.db-kind=mssql  
quarkus.datasource.username=sa  
quarkus.datasource.password=123@Nagesh  
quarkus.datasource.jdbc.driver=com.microsoft.sqlserver.jdbc.SQLServerDriver  
quarkus.datasource.jdbc.url=jdbc:sqlserver://127.0.0.1:1433;databaseName=TestDB;integratedSecurity=false;encrypt=false;trustServerCertificate=true;  
quarkus.datasource.jdbc.max-size=16  
quarkus.hibernate-orm.scripts.generation=drop-and-create  
quarkus.hibernate-orm.scripts.generation.create-target=import.sql  
  
quarkus.hibernate-orm.log.format-sql=true  
quarkus.hibernate-orm.log.sql=true  
quarkus.hibernate-orm.sql-load-script=import.sql  
quarkus.datasource.users.new-connection-sql=CREATE TABLE IF NOT EXISTS Person (id int8 not null, first_name varchar(255),last_name varchar(255),salutation varchar(255), PRIMARY KEY ( id ))  
  ```

  

Following is a snapshot of quarkus project directory.

  

![](https://lh5.googleusercontent.com/GPztNCaEAqnFGvpWuIa90kp03mQSgcNJGrQc0x-Z4_SjUM8esqUagRpZeUsOIUWNYhEDmNj2ML05aricbV4hYXKh6zvhCokJArfSdIbaU4BO1XaIjbne5-yvnixmh7q8iLQk7k65YA5KeW4kzRcFvvUiiiDHRwMqJ9b92P2PM2UPT1EWE3Dr8_jx)

  
  

### 3.3. Run the application

To run quarkus application you have to first check out on application folder which is a quarkus-crud-app

  

```cd quarkus-crud-app```

  

After this, you have to run a command with help of quarkus CLI

  

```qurkus dev```

![](https://lh4.googleusercontent.com/2OC0Zpl0n1QI2_M5VkGh3G2TS2dZwK95zY_wPcmKEzGGCLF90QA2jFYZpHQ4bURiA0IwNGVdbCJkvjw-FJepwoEumxOtGKayWdRZQZPBqFf8ZnroJoZrEvKkzoIR_8R5ZaN18j1Rm_oxpnw7d7zd6T8jLdgfEZ1r2mXELZoxZEYpMzmwKhYCRtWD)

  
  

Now open your browser, type the following URL & check whether your application is running or not.

```[http://localhost:8080](http://localhost:8080)```

![](https://lh4.googleusercontent.com/Iy1Ttyb8WPuQNKsDs4unwStPwAp4mjef5dZ3UuSlD-aUY-zZ9T0IjK1te-Stq6UIGgG7OC67cftk6YTPsbczEIGuyuKOqFJcsWBRcLtQJ2Q9ZA6CKwatWc22VEiaEc3xQeDUIY4L101l7l0OAsATVMLu25tItHH0dTegoAxpnNShXuBjxb8lN8pu)

  

When the webpage successfully arrives on the browser we need to check if the API’ are working or not.

### List a new person

```http :8080/person```

  

![](https://lh4.googleusercontent.com/mgQ6tjRjQgUME5Du1CJDcsxOOMGuJxBNvbHKwjlEY65zLSeHG5X4cjcnoAbmStukT8krEdL8GS30prx0txjXxp4ChB8x3pWObs9HwGvY_13U5U5qVVsOUvzeAnbwb--CQoPSiHHiKwxwNvf49U4dsVwJeJUC0aikyzzw7xJWlqLpwe0q2nCkDMf5)

  

### Create a new person

```http POST :8080/person firstName=Carlos lastName=Santana salutation=Mr```

![](https://lh5.googleusercontent.com/1jKQXwVqSJGQVUUlnyILGGt8KIJL1oIwKMKAAmIISeHpGwq4hf9km0iPAe-dTvhyRh-P4TKziVzarwcU4Foux_E8VhjENoYNtMQHvBaWynd3S_JIlPJA8QLV0llt68jWzaPcecIeU6IXLRoMQpwK8gtdie6IfJ3qNahmToTTLo-TOK4S7qjDdGjF)

  
  

### Updating an existing person

```http PUT :8080/person/9 firstName=Jimi lastName=Hendrix```

![](https://lh5.googleusercontent.com/y4gJRSA9iDCIjDerMC_dR_1L3bdmNmGpeD3qB4lfNp8D8ZzexE56CabaXnp9OHsrIZo3o56nPsYrbnTcazfj7HcnlUkHN3JeVQoV-zNNi55LWbaM6wieu8FbK0lTTGmxeyLjhNJcusKZeYA7MVBX-03fWuESOa44wy5Do8P-excA-3DRFAEi9Bu9)

  
  
  
  
  
  
  
  

### Deleting an existing person

```http DELETE :8080/person/9```

  

![](https://lh3.googleusercontent.com/5iifGwdnmtroE1WmCQsC_rRW3PpQ-4P3pooNP4oLiuJYc0Kt_fmuuyyRIFqFdBC72oHj48AANbr0dgFYAsulj57egB6FB1FCxbU2YEm5DdtTpO81K9maGquELmw687y7XLxFMv5rQZykr6qBr05pAm-jkbIvyu9oOxuWRyq7d9YGmM-2Ah8Fa80u)

  

Now with help of API we created, deleted & updated the database. To check all changes are reflected in the database we have to cross-verify. For that Please follow the below steps.

  

First, we need to login into the Microsoft SQL database. For that, we need to use sqlcmd. please run the following command:

```sqlcmd -S localhost -U sa -P '<YourPassword>'```

  

Now select the sample database which we created earlier.

```USE TestDB  ```
```GO```

  

List the data from our collection for that run the following command to  list  all  data.

```SELECT * FROM Person  ```
```GO```

  
  
  
  
  
  
  

After running the above command you will get the following result in your terminal.

![](https://lh4.googleusercontent.com/xT2NAdYyE_-5EoKhdmHa8P1yFUkIPf4xIveePGQW5dduq3eTKkmeKkfnfx8h1vS5pKfA_MtfUg5ObY9zvPXuGYso0odIQ7N18ZWk_CQtslU81YQKKqhemGyEPSKnXNZGfUkF2SYI4efrqF5Fk2Kbn23SHHG_-ULX_zdvoR5mbRQ2jdEX6x2xrg2N)

  
  
  

### 4. Containerise the Quarkus App using Podman

  

With the help of podman, we will containerize the quarkus application so that it becomes more portable and ready to be deployed on Kubernetes (OpenShift).

For that, we need to first create an image of it. Before creating a container image, we need to run one maven command which creates all the dependency files we want while creating the image.

  

```./mvnw package```

  

This Dockerfile is used to build a container that runs the Quarkus application in JVM mode. You can also build in native,native-micro, and legacy-jar as per your requirement.

  

Quarkus provide us with multiple Dockerfiles with their location of it:

  

├── mvnw

├── mvnw.cmd

├── pom.xml

├── README.md

├── src

│ ├── main

│ │ ├── docker

│ │ │ ├── Dockerfile.jvm

│ │ │ ├── Dockerfile.legacy-jar

│ │ │ ├── Dockerfile.native

│ │ │ └── Dockerfile.native-micro

  

To build a podman image, you have to run the following command.

  

### 4.1. building dockerfile

Now build the container image with help of podman

```podman build -f src/main/docker/Dockerfile.jvm -t quarkus/quarks-crud-app .```

  

### 4.2. Run Quarkus container

Now run the container with help of podman run command on port 8080. We are taking the help of the host network for that add --network=host  [link](https://www.metricfire.com/blog/what-is-docker-network-host/#:~:text=Docker%20network%20host%2C%20also%20known,(e.g.%2C%20port%2080).).

```podman run -it -p 8080:8080 --network=host quarkus/quarks-crud-app```

  

![](https://lh3.googleusercontent.com/1h5dLuSTS4AjTMb3dYdXDMlFjSxQFG-ObS_NHH02lUsRq8qDUtXn6JN2QwtvHQVpsWJ45XOQunE1T1P_qwjq5b367QYgtQCYnkFyhKtbH67QY3AWT1IaerbQ-ftW9tQsQmpaSOHlyFA22HrR7UNFW3aE72L490TtR1g7FlSWnaCvabmZI6uAUuUJ)

  

As we tested all the API above we have to test it again after containerization like GET. POST, PUT, and DELETE.

  

```GET: http :8080/person  ```
  
```POST: http POST :8080/person firstName=Carlos lastName=Santana salutation=Mr  ```
  
```PUT: http PUT :8080/person/1 firstName=Jimi lastName=Hendrix  ```
  
```DELETE: http DELETE :8080/person/1```

  
  
  
  

### 5. Summary
    

Using Quarkus dramatically reduces the lines of code you have to write. As you have seen, creating a simple REST CRUD service is just a piece of cake. If you then want to move your app to Kubernetes, it’s just a matter of adding another extension to the build process.

Thanks to the Dev Services, you’re even able to do fast prototyping without worrying to install 3rd party apps like databases.

Minimizing the amount of boilerplate code makes your app easier to maintain – and lets you focus on what you have to do implementing the business case.

This is why I fell in love with Quarkus.

6.  ### Reference link:
    

-   [https://github.com/nageshredhat/quarkus-crud-mssql-](https://github.com/nageshredhat/quarkus-crud-mssql-)
    
-   [https://www.opensourcerers.org/2021/12/20/how-to-quickly-create-a-crud-service-with-quarkus/](https://www.opensourcerers.org/2021/12/20/how-to-quickly-create-a-crud-service-with-quarkus/)
    
-   [https://github.com/nageshredhat/quarkus-crud-mssql-](https://github.com/nageshredhat/quarkus-crud-mssql-)
    
-   [https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-red-hat?view=sql-server-ver16](https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-red-hat?view=sql-server-ver16)
    
-   [https://quarkus.io/guides/getting-started](https://quarkus.io/guides/getting-started)
    
-   [https://medium.com/javarevisited/a-very-simple-crud-with-quarkus-7b066c9c44e8](https://medium.com/javarevisited/a-very-simple-crud-with-quarkus-7b066c9c44e8)

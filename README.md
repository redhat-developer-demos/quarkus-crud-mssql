
# mssql-try1 Project

This project uses Quarkus, the Supersonic Subatomic Java Framework.

If you want to learn more about Quarkus, please visit its website: https://quarkus.io/ .


## Quarkus Extentions
1. resteasy-reactive
2. agroal
3. quarkus-hibernate-orm-panache
4. quarkus-resteasy-jsonb
5. quarkus-jdbc-mssql
```
quarkus create app org.acme:mssql-try1     --extension=resteasy-reactive 
```
## application.properties
```
quarkus.datasource.db-kind=mssql
quarkus.datasource.username=sa
quarkus.datasource.password=<password>
quarkus.datasource.jdbc.driver=com.microsoft.sqlserver.jdbc.SQLServerDriver
quarkus.datasource.jdbc.url=jdbc:sqlserver://localhost:1433;databaseName=TestDB;integratedSecurity=false;encrypt=false;trustServerCertificate=true;
quarkus.datasource.jdbc.max-size=16
# quarkus.hibernate-orm.scripts.generation=drop-and-create
quarkus.hibernate-orm.scripts.generation.create-target=import.sql

quarkus.hibernate-orm.log.format-sql=true
quarkus.hibernate-orm.log.sql=true
quarkus.hibernate-orm.sql-load-script=import.sql
# quarkus.datasource.users.new-connection-sql=CREATE TABLE IF NOT EXISTS Person (id int8 not null, first_name varchar(255),last_name varchar(255),salutation varchar(255), PRIMARY KEY ( id ))
```


## Running the application in dev mode

You can run your application in dev mode that enables live coding using:
```shell script
./mvnw compile quarkus:dev
```

> **_NOTE:_**  Quarkus now ships with a Dev UI, which is available in dev mode only at http://localhost:8080/q/dev/.

## Packaging and running the application

The application can be packaged using:
```shell script
./mvnw package
```
It produces the `quarkus-run.jar` file in the `target/quarkus-app/` directory.
Be aware that it’s not an _über-jar_ as the dependencies are copied into the `target/quarkus-app/lib/` directory.

The application is now runnable using `java -jar target/quarkus-app/quarkus-run.jar`.

If you want to build an _über-jar_, execute the following command:
```shell script
./mvnw package -Dquarkus.package.type=uber-jar
```

The application, packaged as an _über-jar_, is now runnable using `java -jar target/*-runner.jar`.

## Creating a native executable

You can create a native executable using: 
```shell script
./mvnw package -Pnative
```

Or, if you don't have GraalVM installed, you can run the native executable build in a container using: 
```shell script
./mvnw package -Pnative -Dquarkus.native.container-build=true
```

You can then execute your native executable with: `./target/mssql-try1-1.0.0-SNAPSHOT-runner`

If you want to learn more about building native executables, please consult https://quarkus.io/guides/maven-tooling.

## Related Guides

- RESTEasy Reactive ([guide](https://quarkus.io/guides/resteasy-reactive)): A JAX-RS implementation utilizing build time processing and Vert.x. This extension is not compatible with the quarkus-resteasy extension, or any of the extensions that depend on it.

## Provided Code

### RESTEasy Reactive

Easily start your Reactive RESTful Web Services

[Related guide section...](https://quarkus.io/guides/getting-started-reactive#reactive-jax-rs-resources)

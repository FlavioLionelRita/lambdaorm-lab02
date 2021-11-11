# Lab 02

In this laboratory we will see:

- How to insert data from a file to more than one table.
- how to extend entities using abstract entities
- how to run queries from cli to perform different types of queries

## Pre requirements

### Create database for test

Create file "docker-compose.yaml"

```yaml
version: '3'
services:
  test:
    container_name: lambdaorm-test
    image: mysql:5.7
    restart: always
    environment:
      - MYSQL_DATABASE=test
      - MYSQL_USER=test
      - MYSQL_PASSWORD=test
      - MYSQL_ROOT_PASSWORD=root
    ports:
      - 3306:3306
```

Create MySql database for test:

```sh
docker-compose up -d
```

Create user:

```sh
docker exec lambdaorm-test  mysql --host 127.0.0.1 --port 3306 -uroot -proot -e "GRANT ALL ON *.* TO 'test'@'%' with grant option; FLUSH PRIVILEGES;"
docker exec lambdaorm-test  mysql --host 127.0.0.1 --port 3306 -uroot -proot -e "ALTER DATABASE test CHARACTER SET utf8 COLLATE utf8_general_ci;"
```

### Install lambda ORM CLI

Install the package globally to use the CLI commands to help you create and maintain projects

```sh
npm install lambdaorm-cli -g
```

## Test

### Create project

will create the project folder with the basic structure.

```sh
lambdaorm init -w lab_02
```

position inside the project folder.

```sh
cd lab_02
```

### Complete Schema

In the creation of the project the schema was created but without any entity.
Add the Country entity as seen in the following example

```yaml
app:
  src: src
  data: data
  models: models
  defaultDatabase: mydb
databases:
  - name: mydb
    dialect: mysql
    schema: countries
    connection:
      host: localhost
      port: 3306
      user: test
      password: test
      database: test
      multipleStatements: true
      waitForConnections: true
      connectionLimit: 10
      queueLimit: 0
schemas:
  - name: countries
    enums: []
    entities:
      - name: Positions
        abstract: true
        properties:
          - name: latitude
            length: 16
          - name: longitude
            length: 16
      - name: Countries
        extends: Positions
        primaryKey: ["iso3"]
        uniqueKey: ["name"]
        properties:
          - name: name
            nullable: false
          - name: iso3
            length: 3
            nullable: false
          - name: iso2
            nullable: false
            length: 2
          - name: capital
          - name: currency
          - name: region
          - name: subregion
        relations:
          - name: states
            type: manyToOne
            composite: true
            from: iso3
            entity: States
            to: countryCode
      - name: States
        extends: Positions
        primaryKey: ["id"]
        uniqueKey: ["countryCode","name"]
        properties:
          - name: id
            type: integer
            nullable: false
          - name: name
            nullable: false
          - name: code
            nullable: false
          - name: countryCode
            nullable: false
```

### Update

```sh
lambdaorm update
```

the file model.ts will be created inside src/models/countries.

### Sync

```sh
lambdaorm sync
```

It will generate the Counties and States tables in database and a status file "mydb-state.json" in the "data" folder.

### Popuplate Data

then we execute

```sh
lambdaorm run -e "Countries.bulkInsert().include(p => p.states)" -d ./data.json
```

### Queries

List all Countries:

```sh
lambdaorm run -e "Countries"
```

List all States:

```sh
lambdaorm run -e "States"
```

List 10 countries including states:

```sh
lambdaorm run -e "Countries.page(1,10).include(p => p.states)"
```

List 10 countries including some state fields:

```sh
lambdaorm run -e "Countries.page(1,10).include(p => p.states.map(p=> [p.name,p.latitude,p.longitude] ))"
```

List some fields from 10 countries including some fields from states:

```sh
lambdaorm run -e "Countries.map(p=> [p.name,p.capital,p.currency,p.region]).page(1,10).include(p => p.states.map(p=> [p.name,p.latitude,p.longitude] ))"
```

List the number of countries per region:

```sh
lambdaorm run -e "Countries.map(p=> {region:p.region,count:count(p.iso3)}) "
```

List the number of countries by region and sub region:

```sh
lambdaorm run -e "Countries.map(p=> {region:p.region,subregion:p.subregion,count:count(p.iso3)}) "
```

### Drop

remove all tables from the schema and delete the state file, mydb-state.json

```sh
lambdaorm drop
```

## End

### Remove database for test

Remove MySql database:

```sh
docker-compose down
```

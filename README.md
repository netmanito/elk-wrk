#ELASTICSEARCH WORKPLACESEARCH DOCKER SERVICE

This repo has the files to run *Elasticsearch*, *WorkplaceSearch* and *Kibana* on a single **docker-compose** file for testing purposes.

## FILES

#### .env

Generic environment variables for `docker-compose.yml` file.

```
ELASTIC_VERSION=7.9.0
ELASTIC_SECURITY=true
ELASTIC_PASSWORD=changeme
ENC_KEY=4a2cd3f81d39bf28738c10db0ca782095ffac07279561809eecc722e0c20eb09
WRK_URL=http://localhost:3002
```

#### docker-compose.yml

Docker compose file with 3 containers, Elasticsearch, WorkplaceSearch and Kibana.

Ports **5601** and **3002** have to be mapped in order to access Kibana on `http://localhost:5601` and WorkplaceSearch on `http://localhost:3002`

#### kibana.yml

Modified `kibana.yml` file to fit project configuration.

## ENDPOINTS

#### Kibana

```
http://localhost:5601
```

#### WorksplaceSearch

```
http://localhost:3002
```

## Usage

Run from inside directory:

```
docker-compose up
```

## Comments

Every connector that joins to WorkplaceSearch has the only option to map URL's as `localhost` or with an FQDN name such as *.com*,*.org*,*.net* domains when you deploy an app to connect to WorkplaceSearch. If you configure a domain name, change the appropiate options in Kibana and WorkplaceSearch in order to be able to access their URL's.  

Modify the `/etc/hosts` file in your docker host and add `workplace` container as an alias for localhost.

```
127.0.0.1       localhost workplace
```

Refer to [https://www.elastic.co/guide/en/workplace-search/7.9/index.html](https://www.elastic.co/guide/en/workplace-search/7.9/index.html) for more information on how WorkplaceSearch works.

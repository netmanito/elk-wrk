# Buscador Elastic Workplace Search en docker

### Entorno docker con 3 containers (elasticsearch, kibana y Elastic Enterprise search  (Workplace Search))

**Descargar el repo de github con los archivos de configuración**

		git clone https://github.com/netmanito/elk-wrk
	
**Creará un directorio "elk-wrk" con los siguientes archivos:**

		131185 -rw-r--r-- 1 root root 1880 oct 15 13:25 docker-compose.yml
		131182 -rw-r--r-- 1 root root  173 oct 15 13:25 .env
		131080 drwxr-xr-x 8 root root 4096 oct 15 13:25 .git
		131186 -rwxr-xr-x 1 root root 4772 oct 15 13:25 kibana.yml
		131183 -rw-r--r-- 1 root root 1211 oct 15 13:25 LICENSE
		131184 -rw-r--r-- 1 root root 1658 oct 15 13:25 README.md
	
**En el archivo .env guardamos las variables que utilizaremos después en el docker-compose.yml**

		ELASTIC_VERSION=7.9.0  -- Versión del stack que queremos descargar
		ELASTIC_SECURITY=true  -- Si utilizará o no seguridad, en este caso afirmativo
		ELASTIC_PASSWORD=changeme -- passwd por defecto para el usuario elastic
		ENC_KEY=4a2cd3f81d39bf28738c10db0ca782095ffac07279561809eecc722e0c20eb09  -- Key para workplace search.
		WRK_URL=http://localhost:3002 -- URL de conexión a Workplace Search.

**El arhivo docker-compose.yml contiene todos los pasos que ha de realizar docker para levantar los 3 containers. Utiliza las variables definidas en .env**

		- Descargar y configurar elasticsearch
		- Descargar y configurar workplace search
		- Descargar y configurar kibana
**Contenido docker-compose.yml**

		Docker-compose.yml:
				---
				version: '2'
				services:
				
				  elasticsearch:
				    image: docker.elastic.co/elasticsearch/elasticsearch:$ELASTIC_VERSION
				    environment:
				      - bootstrap.memory_lock=true
				      - discovery.type=single-node
				      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
				      - ELASTIC_PASSWORD=$ELASTIC_PASSWORD
				      - xpack.security.authc.api_key.enabled=true
				      - xpack.security.enabled=$ELASTIC_SECURITY
				      - xpack.security.authc.realms.native.native1.order=0
				    ulimits:
				      memlock:
				        soft: -1
				        hard: -1
				    ports:
				      - 9200:9200
				    networks: ['stack']
				
				  workplace:
				    image: docker.elastic.co/enterprise-search/enterprise-search:$ELASTIC_VERSION
				    restart: always
				    container_name: workplace
				    environment:
				       - elasticsearch.host='http://elasticsearch:9200'
				       - allow_es_settings_modification='true'
				       - elasticsearch.username=elastic
				       - elasticsearch.password=$ELASTIC_PASSWORD
				       - ent_search.listen_host='0.0.0.0'
				       - ent_search.listen_port=3002
				       - ent_search.external_url=$WRK_URL
				       - ent_search.auth.source=elasticsearch-native
				       - ENT_SEARCH_DEFAULT_PASSWORD=$ELASTIC_PASSWORD
				       - secret_management.encryption_keys=[$ENC_KEY]
				    #volumes:
				       #- ${PWD}/enterprise-search.yml:/usr/share/enterprise-search/config/enterprise-search.yml
				    links: ['elasticsearch']
				    depends_on:
				       - elasticsearch
				    ports:
				       - 3002:3002
				    networks:
				       - stack
				
				  kibana:
				    image: docker.elastic.co/kibana/kibana:$ELASTIC_VERSION
				    volumes:
				      - ${PWD}/kibana.yml:/usr/share/kibana/config/kibana.yml
				    environment:
				      - ELASTICSEARCH_USERNAME=elastic
				      - ELASTICSEARCH_PASSWORD=$ELASTIC_PASSWORD
				    ports: ['5601:5601']
				    networks: ['stack']
				    #links: ['elasticsearch']
				    links: ['elasticsearch','workplace']
				    depends_on: ['elasticsearch','workplace']
				
				networks:
				  stack: {}
		
**Configuramos en /etc/hosts de la máquina docker el alias para workplace**

		127.0.0.1       localhost workplace

**Ejecutamos docker-compose up en el directorio elk-wrk y se desencadena todo el proceso de descarga, configuración y arranque de los servicios (elasticsearch,workplace y kibana) en un container cada uno de ellos.**

	root@docker:/docker/elk-wrk# docker ps
	CONTAINER ID        IMAGE                                                         COMMAND                  CREATED             STATUS              PORTS                              NAMES
	b7420481d34b        docker.elastic.co/kibana/kibana:7.9.0                         "/usr/local/bin/dumb…"   2 hours ago         Up 2 hours          0.0.0.0:5601->5601/tcp             elkwrk_kibana_1
	5def5463f586        docker.elastic.co/enterprise-search/enterprise-search:7.9.0   "/usr/local/bin/dock…"   2 hours ago         Up 2 hours          0.0.0.0:3002->3002/tcp             workplace
	197418ce3c5f        docker.elastic.co/elasticsearch/elasticsearch:7.9.0           "/tini -- /usr/local…"   2 hours ago         Up 2 hours          0.0.0.0:9200->9200/tcp, 9300/tcp   elkwrk_elasticsearch_1
	
**Una vez los 3 containers estén "UP", podremos conectarnos a Kibana a través de "localhost:5601" y a workplace search a través de "localhost:3002"**

		http://localhost:5601/ --> kibana 
		http://localhost:3002  --> workplace search


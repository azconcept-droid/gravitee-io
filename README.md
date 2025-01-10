# Gravitee io

Gravitee is an open-source, event-native API management platform that helps you design, deploy, manage, and secure synchronous and asynchronous APIs throughout the entire lifecycle.

- [compose file v3.x](https://raw.githubusercontent.com/gravitee-io/gravitee-docker/master/apim/3.x/docker-compose.yml)

- [installation guide](https://docs.gravitee.io/apim/3.x/apim_installation_guide_docker_compose_quickstart.html)

mkdir -p /gravitee/{mongodb/data,elasticsearch/data,apim-gateway/plugins,apim-gateway/logs,apim-management-api/plugins,apim-management-api/logs,apim-management-ui/logs,apim-portal-ui/logs}


docker network create storage
docker network create frontend


docker pull mongo:6
docker run --name gio_apim_mongodb \
  --net storage \
  --volume /gravitee/mongodb/data:/data/db \
  --detach mongo:6


docker pull docker.elastic.co/elasticsearch/elasticsearch:8.8.1
docker run --name gio_apim_elasticsearch \
  --net storage \
  --hostname elasticsearch \
  --env http.host=0.0.0.0 \
  --env transport.host=0.0.0.0 \
  --env xpack.security.enabled=false \
  --env xpack.monitoring.enabled=false \
  --env cluster.name=elasticsearch \
  --env bootstrap.memory_lock=true \
  --env discovery.type=single-node \
  --env "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  --volume /gravitee/elasticsearch/data:/var/lib/elasticsearch/data \
  --detach docker.elastic.co/elasticsearch/elasticsearch:8.8.1


$ docker pull graviteeio/apim-gateway:4.0
$ docker run --publish 8082:8082 \
  --volume /gravitee/apim-gateway/plugins:/opt/graviteeio-gateway/plugins-ext \
  --volume /gravitee/apim-gateway/logs:/opt/graviteeio-gateway/logs \
  --env gravitee_management_mongodb_uri="mongodb://gio_apim_mongodb:27017/gravitee-apim?serverSelectionTimeoutMS=5000&connectTimeoutMS=5000&socketTimeoutMS=5000" \
  --env gravitee_ratelimit_mongodb_uri="mongodb://gio_apim_mongodb:27017/gravitee-apim?serverSelectionTimeoutMS=5000&connectTimeoutMS=5000&socketTimeoutMS=5000" \
  --env gravitee_reporters_elasticsearch_endpoints_0="http://elasticsearch:9200" \
  --env gravitee_plugins_path_0=/opt/graviteeio-gateway/plugins \
  --env gravitee_plugins_path_1=/opt/graviteeio-gateway/plugins-ext \
  --net storage \
  --name gio_apim_gateway \
  --detach graviteeio/apim-gateway:4.0
$ docker network connect frontend gio_apim_gateway


$ docker pull graviteeio/apim-management-api:4.0
$ docker run --publish 8083:8083 \
  --volume /gravitee/apim-management-api/plugins:/opt/graviteeio-management-api/plugins-ext \
  --volume /gravitee/apim-management-api/logs:/opt/graviteeio-management-api/logs \
  --volume /gravitee/license.key:/opt/graviteeio-management-api/license/license.key \
  --env gravitee_management_mongodb_uri="mongodb://gio_apim_mongodb:27017/gravitee-apim?serverSelectionTimeoutMS=5000&connectTimeoutMS=5000&socketTimeoutMS=5000" \
  --env gravitee_analytics_elasticsearch_endpoints_0="http://elasticsearch:9200" \
  --env gravitee_plugins_path_0=/opt/graviteeio-management-api/plugins \
  --env gravitee_plugins_path_1=/opt/graviteeio-management-api/plugins-ext \
  --net storage \
  --name gio_apim_management_api \
  --detach graviteeio/apim-management-api:4.0
$ docker network connect frontend gio_apim_management_api


$ docker pull graviteeio/apim-management-ui:4.0
$ docker run --publish 8084:8080 \
  --volume /gravitee/apim-management-ui/logs:/var/log/nginx \
  --net frontend \
  --name gio_apim_management_ui \
  --env MGMT_API_URL=http://localhost:8083/management/organizations/DEFAULT/environments/DEFAULT \
  --detach graviteeio/apim-management-ui:4.0


$ docker pull graviteeio/apim-portal-ui:4.0
$ docker run --publish 8085:8080 \
  --volume /gravitee/apim-portal-ui/logs:/var/log/nginx \
  --net frontend \
  --name gio_apim_portal_ui \
  --env PORTAL_API_URL=http://localhost:8083/portal/environments/DEFAULT \
  --detach graviteeio/apim-portal-ui:4.0


To open the Console and the Developer portal, complete the following steps:

To open the console, go to http://localhost:8084.

To open the Developer Portal, go to http://localhost:8085.

The default username for the Console and the Developer Portal is admin.

The default password for the Developer Portal is admin.

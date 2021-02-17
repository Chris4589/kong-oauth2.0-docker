# kong-oauth2.0-docker



docker network create my-kong-net

docker run -d --name kong-cassandra-database --network=my-kong-net -p 9059:9042 cassandra:3

docker run -d --name my-kong-database --network=my-kong-net -p 5555:5432 -e "POSTGRES_USER=kong" -e "POSTGRES_DB=kong" -e "POSTGRES_PASSWORD=kong" postgres:9.6


docker run --rm --network=my-kong-net -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=my-kong-database" -e "KONG_PG_USER=kong" -e "KONG_PG_PASSWORD=kong" -e "KONG_CASSANDRA_CONTACT_POINTS=kong-cassandra-database" kong:latest kong migrations bootstrap



docker run -d --name kong-imagen \
     --network=my-kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=my-kong-database" \
     -e "KONG_PG_USER=kong" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-cassandra-database" \
     -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
     -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
     -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
     -p 9000:8000 \
     -p 9443:8443 \
     -p 127.0.0.1:9001:8001 \
     -p 127.0.0.1:9444:8444 \
     kong:latest

curl http://127.0.0.1:9001:9001



//rutas

curl -X POST \
  --url "http://127.0.0.1:9001/services" \
  --data "name=test-servicio" \
  --data "url=https://jsonplaceholder.typicode.com/todos"


curl -X POST \
  --url "http://127.0.0.1:9001/services/test-servicio/routes" \
  --data 'paths[]=/api/v1/customers'\



//acl
 curl -X POST http://localhost:9001/services/test-servicio/plugins \
    --data "name=acl"  \
    --data "config.allow=group1" \
    --data "config.hide_groups_header=true"


curl -X POST http://localhost:9001/consumers/bc762add-4463-4708-9612-64e9363925ab/acls \
    --data "group=group1"

//oauth 2.0

curl -X POST http://localhost:9001/services/test-servicio/plugins \
    --data "name=oauth2"  \
    --data "config.scopes=read" \
    --data "config.scopes=write" \
    --data "config.mandatory_scope=true" \
    --data "config.enable_authorization_code=true" \
    --data "config.enable_password_grant=true" \
    --data "config.enable_implicit_grant=true" \
    --data "config.enable_client_credentials=true" \
    --data "config.global_credentials=true" \
    --data "config.accept_http_if_already_terminated=true"


curl -X POST http://localhost:9001/consumers/ \
    --data "username=cherrera" \
    --data "custom_id=cherrera"


curl -X POST http://localhost:9001/consumers/cherrera/oauth2 \
    --data "name=test" \
    --data "client_id=cliente" \
    --data "client_secret=secreto" \
    --data "redirect_uris=https://jsonplaceholder.typicode.com/todos" \
    --data "hash_secret=false" \
     --insecure
-----

//Resource Owner Password Credentials
curl -k https://localhost:9443/api/v1/customers/oauth2/token \
     --data "client_id=cliente" \
     --data "client_secret=secreto" \
     --data "grant_type=password" \
     --data "scope=write" \
     --data "provision_key=GHmML6kj2S96L3FjOnxCrP5TghTbvavj" \
     --data "authenticated_userid=cherrera" \
     --data "username=XXX" \
     --data "password=XXX"


//Authorization Code
curl -k https://localhost:9443/api/v1/customers/oauth2/authorize \
     --data "client_id=cliente" \
     --data "response_type=code" \
     --data "scope=write" \
     --data "provision_key=GHmML6kj2S96L3FjOnxCrP5TghTbvavj" \
     --data "authenticated_userid=cherrera"


//client Credentials
curl -k https://localhost:9443/api/v1/customers/oauth2/token \
     --data "client_id=cliente" \
     --data "client_secret=secreto" \
     --data "grant_type=client_credentials" \
     --data "scope=write" 

//implicit
curl -k https://localhost:9443/api/v1/customers/oauth2/authorize \
     --data "client_id=cliente" \
     --data "response_type=token" \
     --data "scope=write" \
     --data "provision_key=GHmML6kj2S96L3FjOnxCrP5TghTbvavj" \
     --data "authenticated_userid=cherrera"

//refresh
curl -k https://localhost:9443/api/v1/customers/oauth2/token \
    --data "grant_type=refresh_token" \
    --data "client_id=cliente" \
    --data "client_secret=secreto" \
    --data "refresh_token=m4iXWa0cQiL7xt0dZBphn4aPMJDlovkw"


--



# Files

- `app.js`,
  - `GET /authorize`,
  - `POST /authorize`,
- `authorization.jade`,

# Setting up the environment

```shell
curl -X POST \
  --url "http://127.0.0.1:8001/services" \
  --data "name=gudang-service" \
  --data "url=http://gudangbin.org/api"
```

- add a route for that service:

```shell
curl -X POST \
  --url "http://127.0.0.1:8001/services/gudang-service/routes" \
  --data 'hosts[]=gudangbin.org' \
  --data 'paths[]=/gudang'
```

- add the OAuth 2.0 plugin, with available scopes (scope seperti permission / tidak mandatory):

```shell
curl -X POST \
  --url http://127.0.0.1:8001/services/gudang-service/plugins/ \
  --data "name=oauth2" \
  --data "config.scopes=email, phone, address" \
  --data "config.mandatory_scope=true" \
  --data "config.enable_authorization_code=true"
```
- result :

```json
{
  "service_id": "2c0c8c84-cd7c-40b7-c0b8-41202e5ee50b",
  "value": {
    "scopes": ["email", "phone", "address"],
    "mandatory_scope": true,
    "provision_key": "2ef290c575cc46eec61947aa9f1e67d3",
    "hide_credentials": false,
    "enable_authorization_code": true,
    "token_expiration": 7200
  },
  "created_at": 1435783325000,
  "enabled": true,
  "name": "oauth2",
  "id": "656954bd-2130-428f-c25c-8ec47227dafa"
}
```

- create a Kong consumer (called `kautsar`):

```shell
curl -X POST \
  --url "http://127.0.0.1:8001/consumers/" \
  --data "username=kautsar"

```

```shell
curl -X POST \
  --url "http://127.0.0.1:8001/consumers/kautsar/oauth2/" \
  --data "name=Hello World App" \
  --data "redirect_uri=http://getkong.org/"

```
- result :

```json
{
  "consumer_id": "a0977612-bd8c-4c6f-ccea-24743112847f",
  "client_id": "318f98be1453427bc2937fceab9811bd",
  "id": "7ce2f90c-3ec5-4d93-cd62-3d42eb6f9b64",
  "name": "Hello World App",
  "created_at": 1435783376000,
  "redirect_uri": "http://getkong.org/",
  "client_secret": "efbc9e1f2bcc4968c988ef5b839dd5a4"
}
```

# Running the web application

```shell
export PROVISION_KEY="rEqnUS06M9vOpQ6av87Gp6yzrU0EsAxG"
export KONG_ADMIN="http://127.0.0.1:8001"
export KONG_API="https://127.0.0.1:8443"
export API_PATH="/gudang"
export SERVICE_HOST="gudangbin.org"
export SCOPES="{ \
  \"email\": \"Grant permissions to read your email address\", \
  \"address\": \"Grant permissions to read your address information\", \
  \"phone\": \"Grant permissions to read your mobile phone number\" \
}"
```

Note: jika port default (3000) terpakai gunakan syntax ini untuk mengubah port

```shell
export LISTEN_PORT=3030
```

running this project by executing: 

```shell
node app.js
```

# Testing the Authorization Flow

`http://127.0.0.1:3000/authorize?response_type=code&scope=email%20address&client_id=318f98be1453427bc2937fceab9811bd`

```shell
curl -X POST \
  --url "https://127.0.0.1:8443/gudang/oauth2/token" \
  --header "Host: gudangbin.org" \
  --data "grant_type=authorization_code" \
  --data "client_id=1AwQYGabyy55vBTWINJjAZuTT50gtS1d" \
  --data "client_secret=hBBHoFnxUiJdUBX1KjIYRVx7IH3KcaQt" \
  --data "redirect_uri=http://getkong.org/" \
  --data "code=B8yPfsu2vksXNrcZK6QL0R51sZdm6ai3" \
  --insecure
```

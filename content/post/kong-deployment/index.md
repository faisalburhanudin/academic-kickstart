+++
title = "Kong Deployment"
subtitle = ""

# Add a summary to display on homepage (optional).
summary = "Deploy kong api gateway using docker-compose"

date = 2019-04-04T11:32:15+07:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = []

# Is this a featured post? (true/false)
featured = false

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ["webservice", "api-gateway"]
categories = []

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["deep-learning"]` references 
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
# projects = ["internal-project"]

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
[image]
  # Caption (optional)
  caption = ""

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = ""
+++
# Example deployment kong api gateway using docker-compose.

all this script avaliable at https://github.com/faisalburhanudin/kong-deploy


## Docker compose stack

In this docker-compose there are 4 services:

1. my-service: for example rest api
2. kong-database: for kong database storage
3. konga: for kong dashboard
4. kong: api gateway

For more detail open docker-compose.yml


## Start services 

Start service one by one, because some service sometime run but not in ready state

```bash
# run kong-database
docker-compose up -d kong-database

# check if kong-database ready for connection
# last output should be something like this
# kong-database_1  | LOG:  database system is ready to accept connections
docker-compose logs kong-database

# run migration
docker-compose up migration

# run kong
docker-compose up -d kong

# check if kong is run successful
curl -i http://localhost:8001/

# rung konga
# after run you can visit http://127.0.0.1:1337 in browser
# WARNING: when create admin user, use long password, if not konga will be error after register
# for complete konga installation open https://github.com/pantsel/konga
# NOTE: when create connection to kong admin use url http://kong:8001/
docker-compose up -d konga

# run my-service
docker-compose up -d my-service
```

## Register service to kong step by step

```bash
# add service
python add_service.py

# add a route
python add_route.py


# Check request via gateway
curl -i -X GET --url http://localhost:8000/ --header 'Host: my-service.dev'
```

## Plugins

### Rate limit

```bash
# enable rate limit plugin, in this script 5 request/minute
python enable_ratelimit.py

# check request 6 request
curl -i -X GET --url http://localhost:8000/ --header 'Host: my-service.dev'
curl -i -X GET --url http://localhost:8000/ --header 'Host: my-service.dev'
curl -i -X GET --url http://localhost:8000/ --header 'Host: my-service.dev'
curl -i -X GET --url http://localhost:8000/ --header 'Host: my-service.dev'
curl -i -X GET --url http://localhost:8000/ --header 'Host: my-service.dev'
curl -i -X GET --url http://localhost:8000/ --header 'Host: my-service.dev'

# 00/ --header 'Host: my-service.dev'
# HTTP/1.1 429 Too Many Requests
# Date: Tue, 02 Apr 2019 06:49:07 GMT
# Content-Type: application/json; charset=utf-8
# Connection: keep-alive
# Content-Length: 37
# X-RateLimit-Limit-minute: 5
# X-RateLimit-Remaining-minute: 0
# Server: kong/1.1.1

# {"message":"API rate limit exceeded"}
```

### JWT

```bash
# enable jwt plugin
python enable_jwt.py
```
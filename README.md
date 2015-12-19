nginx-passenger
===============

Nginx and Phusion Passenger all wrapped up with Docker

# Setup

This image was created with deploying Sinatra apps in mind. Here's a simple _Hello, world!_ app to demonstrate:

## Gemfile

```
# Gemfile
source "https://rubygems.org"

gem 'sinatra'
gem 'passenger'
```

## app.rb

```
# app.rb
require 'sinatra/base'

class HelloWorldApp < Sinatra::Base
  get '/' do
    "Hello, world!"
  end
end
```

## config.ru

```
# config.ru
require File.expand_path('app', File.dirname(__FILE__))
run HelloWorldApp
```

## config/nginx/default.conf

```
# config/nginx/default.conf
server {
  listen 80 default_server;
  listen [::]:80 default_server ipv6only=on;

  server_name example.com;
  passenger_enabled on;
  passenger_app_env development;
  root /usr/share/nginx/html/public;

  # redirect server error pages to the static page /50x.html
  error_page   500 502 503 504  /50x.html;
  location = /50x.html {
    root html;
  }
}
```

## Dockerfile

```
# Dockerfile
FROM raphaeldelaghetto/nginx-passenger
MAINTAINER Daniel Bidulock

ADD Gemfile /usr/share/nginx/html/Gemfile

WORKDIR /usr/share/nginx/html
RUN bundle install
```

## docker-compose.yml

```
# docker-compose.yml
nginx:
  restart: always
  build: ./
  ports:
    - "80:80"
    - "443:443"
  volumes:
    # Page content
    - ./:/usr/share/nginx/html
    # default.conf
    - ./config/nginx:/etc/nginx/sites-enabled/
```

# Fire 'er up!

```
docker-compose up
```



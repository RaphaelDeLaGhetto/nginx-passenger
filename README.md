nginx-passenger
===============

Nginx and Phusion Passenger all wrapped up with Docker.

# Motivation

Docker is great. As I've grown in my understanding of how to apply this technology, I've come to prefer Docker Compose to creating custom images for deployment. Compose, like Docker itself is great, but I haven't quite been able to escape a reliance upon defining a custom image in a `Dockerfile`. This is because (when it comes to Ruby web apps, at least) it's still easier to install the gem dependencies inside the image than it is to set up a shared volume with the dependencies installed on the host machine. I'm all about what's fastest and easiest, and this is the best I've come up with so far.

The _nginx-passenger_ image was created with deploying small Sinatra apps in mind, though I see no reason why the same basic technique couldn't be applied to other Passenger-friendly frameworks. Submit a pull request if you are able to describe other successful application deployments.

# Tutorial

Follow these steps to deploy a simple  _Hello, world!_ app into production:

## Setup

Create the project directory on the production server:

```
mkdir hello-world
cd hello-world
```

## Gemfile

```
vim Gemfile
```

Copy and save the following:

```
# Gemfile
source 'https://rubygems.org'

gem 'sinatra'
gem 'passenger'
```

## app.rb

```
vim app.rb
```

Copy and save the following:

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
vim config.ru
```

Copy and save the following:

```
# config.ru
require File.expand_path('app', File.dirname(__FILE__))
run HelloWorldApp
```

## config/default.conf

```
mkdir config
vim config/default.conf
```

Copy and save the following:

```
# config/default.conf
server {
  listen 80 default_server;
  listen [::]:80 default_server ipv6only=on;

  server_name example.com;
  passenger_enabled on;
  root /usr/share/nginx/html/public;
}
```

## Dockerfile

This is the step I hope to be able to eliminate eventually. It's necessary because Phusion Passenger and the web app itself (installed in a shared volume on the host system) require the gem dependencies to be installed in the Docker container. 

```
vim Dockerfile
```

Copy and save the following:

```
# Dockerfile
FROM raphaeldelaghetto/nginx-passenger
MAINTAINER Some Guy 

ADD Gemfile /usr/share/nginx/html/Gemfile

WORKDIR /usr/share/nginx/html
RUN bundle install
```

## docker-compose.yml

```
vim docker-compose.yml
```

Copy and save the following:

```
# docker-compose.yml
nginx:
  restart: always
  build: ./
  ports:
    - "80:80"
  volumes:
    # Page content
    - ./:/usr/share/nginx/html
    # default.conf
    - ./config:/etc/nginx/sites-enabled/
```

The `build: ./` line builds the _Dockerfile_ defined above.

# Fire 'er up!

After all that, you should have a project structure that looks like this:

```
/home/app/
- hello-world/
  - config/
      default.conf
    app.rb
    config.ru
    docker-compose.yml
    Dockerfile
    Gemfile
```

Assuming all steps were followed correctly, this will pull all the required images and start serving the app:

```
docker-compose up
```

# Contributing

Hit me with it! There is plenty of room for improvement in the `Dockerfile`. I'd also like to see if anyone is able to successfully deploy a Rails app (or whatever). Improvements to the established techniques are always welcome.

# License

MIT


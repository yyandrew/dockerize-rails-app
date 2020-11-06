# Create new rails app in container
This project is the source code of my blog: https://blog.ky2k.top/docker/2020/11/18/share-docker-tech-with-team.html

### Setup vagrant
```
vagrant up
```
### ssh into vagrant VM
```
vagrant ssh
```
### Create new Dockerfile to create new images contains rails6

```
# Dockerfile
FROM ruby:2.7.2
RUN gem install rails
```

Run `docker build -t rails6 .` to create a new Rails images

### Use rails6 to create a new rails app

```
docker run -it --rm --user "$(id -u):$(id -g)" -v "$PWD":/app -w /app \
  rails6 rails new --api --skip-bundle -d postgresql rails_with_docker
```

### Create Dockerfile
```
cd rails_with_docker

touch Dockerfile

tee -a Dockerfile << END
# Use ruby 2.7.2 image
FROM ruby:2.7.2

# Install dependencies
RUN apt-get update && apt-get install -y \
  curl \
  build-essential \
  libpq-dev

# Install gems and use /tmp/ to cache gems
COPY Gemfile* /tmp/
WORKDIR /tmp
RUN gem update bundler
RUN bundle install --jobs 5

# Create app folder
RUN mkdir /app
# Set work dir
WORKDIR /app

# Expose 3000
EXPOSE 3000
END
```

### Create docker-compose.yml
```
touch docker-compose.yml
tee -a docker-compose.yml << END
version: '3'
services:
  # database service
  db:
    image: postgres:9.6-alpine
    # Expose container's 5432 to host
    ports:
      - 5432:5432
    # Set share volume
    volumes:
      - ./tmp/postgres_data:/var/lib/postgresql/data
    # Set password of database
    environment:
      POSTGRES_PASSWORD: password
  web:
    # Web service
    build: .
    # Start rails server
    command: /bin/bash -c "rm -f /tmp/server.pid && bundle exec rails server -b 0.0.0.0 -P /tmp/server.pid"
    # Expose 3000 port
    ports:
      - 3000:3000
    # Depends on database
    depends_on:
      - db
    # Set share volume
    volumes:
      - .:/app
END
```

### Update config/database.yml
```
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  host: db
  username: postgres
  # Comes from the value of POSTGRES_PASSWORD of docker-compose.yml
  password: password

development:
  <<: *default
  database: rails_with_docker_development
test:
  <<: *default
  database: rails_with_docker_test
production:
  <<: *default
  database: rails_with_docker_production
```
### Build services by docker compose
```
docker-compose build
docker-compose run web rails db:create
docker-compose run web rails db:migrate
```
### Run entire app
```
docker-compose up
```
### Test 
```
curl -I -H "Content-Type: application/json" -X GET http://localhost:3000
```
### Add assets
```
docker-compose down
docker-compose run web bundle exec rails g scaffold User name:string
docker-compose run web bundle exec rails db:migrate
docker-compose up
```

### Test users list page
```
curl -H "Content-Type: application/json" -X POST -d '{"name": "andrew"}' http://localhost:3000/users
```
It should output
```
{"id":1,"name":"andrew","created_at":"2020-11-09T06:08:16.516Z","updated_at":"2020-11-09T06:08:16.516Z"}
```

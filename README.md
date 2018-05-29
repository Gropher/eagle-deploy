# eagle-deploy
Eagleplatform Deploy Docs
## Capistrano Deploy
### Eagleplatform
#### Deploy Command
```bash
bundle exec cap production deploy
```
#### Branch
```bash
production
```
#### Concurrency
| Service      | No of Processes |
| ------------ |:---------------:|
| web          | 1               |
| sync         | 1               |
| insque       | 1               |
| filters_sync | 1               |

#### Servers
* b20
* b21
* b28
### Messenger
#### Deploy Command
```bash
bundle exec cap production deploy
```
#### Branch
```bash
production
```
#### Concurrency
| Service      | No of Processes |
| ------------ |:---------------:|
| worker       | 1               |

#### Servers
* b20
* b21
* b28
### Balancer
#### Deploy Command
```bash
bundle exec cap production deploy
```
#### Branch
```bash
release
```
#### Concurrency
| Service      | No of Processes |
| ------------ |:---------------:|
| balancer_app | 1               |
| end_user_api | 5               |
| insque       | 1               |

#### Servers
* b20
* b21
* b28
### Billing
#### Deploy Command
```bash
bundle exec cap production deploy
```
#### Branch
```bash
production
```
#### Concurrency
| Service      | No of Processes |
| ------------ |:---------------:|
| web          | 1               |
| insque       | 1               |

#### Servers
* v36
* v38
### Uploader
#### Deploy Command
```bash
bundle exec cap production deploy
```
#### Branch
```bash
production
```
#### Concurrency
| Service      | No of Processes |
| ------------ |:---------------:|
| uploader     | 1               |

#### Servers
* b34
* b35
* b36
* b37
### Downloader
#### Deploy Command
```bash
bundle exec cap production deploy
```
#### Branch
```bash
production
```
#### Concurrency
| Service      | No of Processes |
| ------------ |:---------------:|
| downloader   | 1               |

#### Servers
* a38
* b24
* b25
* b34
* b35
* b36
* b37
* b39
* c32
* d33
* stor8
* stor9
### Thumbnailer
#### Deploy Command
```bash
bundle exec cap production deploy
```
#### Branch
```bash
production
```
#### Concurrency
| Service      | No of Processes |
| ------------ |:---------------:|
| thumbnailer  | 1               |

#### Servers
* a38
* b24
* b25
* b34
* b35
* b36
* b37
* b39
* c32
* d33
* stor8
* stor9
### Converter
#### Deploy Command
```bash
bundle exec cap production deploy
```
#### Branch
```bash
production
```
#### Concurrency
| Service      | No of Processes |
| ------------ |:---------------:|
| converter    | 1               |

#### Servers
* a38
* b24
* b25
* b34
* b35
* b36
* b37
* b39
* c32
* d33
* stor8
* stor9
### Scaler
#### Deploy Command
```bash
bundle exec cap production deploy
```
#### Branch
```bash
production
```
#### Concurrency
| Service      | No of Processes |
| ------------ |:---------------:|
| scaler       | 1               |

#### Servers
* b34
* b35
* b36
* b37
### Youtuber
#### Deploy Command
```bash
bundle exec cap production deploy
```
#### Branch
```bash
production
```
#### Concurrency
| Service      | No of Processes |
| ------------ |:---------------:|
| youtuber     | 1               |

#### Servers
* b34
### Runner
#### Deploy Command
```bash
bundle exec cap production deploy
```
#### Branch
```bash
production
```
#### Concurrency
| Service      | No of Processes |
| ------------ |:---------------:|
| runner       | 1               |

#### Servers
* a10
* a12
* a17
* a38
* b34
* b35
* b36
* b37
* stor9


## Docker Swarm Deploy
### Eagleplatform
#### Deploy Command
```bash
./bin/build.stag.sh && ./bin/deploy.stag.sh
```
#### Composefile
```yaml
version: '3.2'

networks:
  proxy:
    external: true

services:
  eagleplatform:
    image: registry.eagleplatform.com/eagleplatform:latest 
    environment:
      - RAILS_ENV=production
      - RAILS_LOG_TO_STDOUT=true
      - RAILS_SERVE_STATIC_FILES=true
      - RAILS_MAX_THREADS=16
      - DB_NAME=mediahosting_production
    command: ./bin/start.stag.sh
    networks:
      - proxy
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.backend=eagleplatform"
        - "traefik.frontend.rule=HostRegexp:auth.test.eagleplatform.com,{subdomain:[a-z0-9]+}.auth.test.eagleplatform.com,media.test.eagleplatform.com,{subdomain:[a-z0-9]+}.media.test.eagleplatform.com"
        - "traefik.port=80"
  insque:
    image: registry.eagleplatform.com/eagleplatform:latest 
    environment:
      - RAILS_ENV=production
      - DB_NAME=mediahosting_production
    command: bundle exec rake insque:run
    networks:
      - proxy
  watchdog:
    image: registry.eagleplatform.com/eagleplatform:latest 
    environment:
      - RAILS_ENV=production
      - DB_NAME=mediahosting_production
    command: bundle exec rake watchdog:work
    networks:
      - proxy
  sync:
    image: registry.eagleplatform.com/eagleplatform:latest 
    environment:
      - RAILS_ENV=production
      - DB_NAME=mediahosting_production
    command: bundle exec rake sync:do
    networks:
      - proxy
  filters_sync:
    image: registry.eagleplatform.com/eagleplatform:latest 
    environment:
      - RAILS_ENV=production
      - DB_NAME=mediahosting_production
    command: bundle exec rake filters_sync:do
    networks:
      - proxy
```
#### Dockerfile
```Dockerfile
# Base image:
FROM registry.eagleplatform.com/base:latest

RUN mkdir -p /eagleplatform


WORKDIR /eagleplatform
ADD Gemfile /eagleplatform/Gemfile
ADD Gemfile.lock /eagleplatform/Gemfile.lock
RUN bundle install

ADD . /eagleplatform

RUN RAILS_ENV=production NO_INSQUE=true bundle exec rake assets:precompile

EXPOSE 3000
```
### Messenger
#### Deploy Command
```bash
./bin/build.stag.sh && ./bin/deploy.stag.sh
```
#### Composefile
```yaml
version: '3.2'

networks:
  proxy:
    external: true

services:
  messenger:
    image: registry.eagleplatform.com/messenger:latest
    command: bundle exec ruby messenger.rb 3000
    environment:
      - RAILS_ENV=production
    networks:
      - proxy
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.backend=messenger"
        - "traefik.frontend.rule=Host:messenger.test.eagleplatform.com"
        - "traefik.port=3000"
```
#### Dockerfile
```Dockerfile
FROM registry.eagleplatform.com/base:latest
MAINTAINER Yury Gomozov <grophen@gmail.com>

RUN mkdir -p /messenger

WORKDIR /messenger

ADD Gemfile /messenger/Gemfile
ADD Gemfile.lock /messenger/Gemfile.lock
RUN bundle install

ADD . /messenger

CMD bundle exec ruby messenger.rb
```
### Balancer
#### Deploy Command
```bash
./bin/build.stag.sh && ./bin/deploy.stag.sh
```
#### Composefile
```yaml
version: '3.2'

networks:
  proxy:
    external: true

services:
  balancer:
    image: registry.eagleplatform.com/balancer:latest 
    environment:
      - RAILS_ENV=production
      - RAILS_LOG_TO_STDOUT=true
      - RAILS_MAX_THREADS=16
      - DB_NAME=balancer
    command: ./bin/start.stag.sh
    networks:
      - proxy
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.backend=balancer"
        - "traefik.frontend.rule=Host:panel.test.eaglecdn.com"
        - "traefik.port=80"
  end_user_api:
    image: registry.eagleplatform.com/balancer:latest 
    environment:
      - RAILS_ENV=production
      - RAILS_MAX_THREADS=16
      - DB_NAME=balancer
      - BUNDLE_GEMFILE=/balancer/end_user_api/Gemfile 
      - PORT=3000
    working_dir: /balancer/end_user_api
    command: bundle exec ruby end_user_api.rb 80
    networks:
      - proxy
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.backend=balancer_end_user_api"
        - "traefik.frontend.rule=Host:view.test.eaglecdn.com"
        - "traefik.port=80"
  insque:
    image: registry.eagleplatform.com/balancer:latest 
    environment:
      - RAILS_ENV=production
      - DB_NAME=balancer
    command: bundle exec rake insque:run 
    networks:
      - proxy
```
#### Dockerfile
```Dockerfile
# Base image:
FROM registry.eagleplatform.com/base:latest

RUN mkdir -p /balancer
RUN mkdir -p /balancer/end_user_api


WORKDIR /balancer

ADD end_user_api/Gemfile /balancer/end_user_api/Gemfile
ADD end_user_api/Gemfile.lock /balancer/end_user_api/Gemfile.lock
RUN cd /balancer/end_user_api; BUNDLE_GEMFILE=/balancer/end_user_api/Gemfile bundle install

ADD Gemfile /balancer/Gemfile
ADD Gemfile.lock /balancer/Gemfile.lock
RUN cd /balancer; BUNDLE_GEMFILE=/balancer/Gemfile bundle install

ADD . /balancer

RUN RAILS_ENV=production NO_INSQUE=true bundle exec rake assets:precompile
```
### Uploader
#### Deploy Command
```bash
./bin/build.stag.sh && ./bin/deploy.stag.sh
```
#### Composefile
```yaml
version: '3.2'

networks:
  proxy:
    external: true

services:
  uploader:
    image: registry.eagleplatform.com/uploader:latest
    command: bundle exec ruby uploader.rb -p 3000
    environment:
      - RAILS_ENV=production
      - APPLICATION_HOSTNAME=upload.v13.servers.test.eaglecdn.com
    networks: 
      - proxy
    volumes:
      - /tmp:/tmp
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.backend=uploader"
        - "traefik.frontend.rule=Host:upload.v13.servers.test.eaglecdn.com"
        - "traefik.port=3000"
```
#### Dockerfile
```Dockerfile
FROM registry.eagleplatform.com/ffmpeg:latest
MAINTAINER Yury Gomozov <grophen@gmail.com>

RUN mkdir -p /uploader

WORKDIR /uploader

ADD Gemfile /uploader/Gemfile
ADD Gemfile.lock /uploader/Gemfile.lock
RUN bundle install

ADD . /uploader

CMD bundle exec ruby uploader.rb
```
### Thumbnailer
#### Deploy Command
```bash
./bin/build.stag.sh && ./bin/deploy.stag.sh
```
#### Composefile
```yaml
version: '3.2'

networks:
  proxy:
    external: true

services:
  thumbnailer:
    image: registry.eagleplatform.com/thumbnailer:latest
    environment:
      - RAILS_ENV=production
    networks:
      - proxy
    volumes:
      - /tmp:/tmp
```
#### Dockerfile
```Dockerfile
FROM registry.eagleplatform.com/ffmpeg:latest
MAINTAINER Yury Gomozov <grophen@gmail.com>

RUN mkdir -p /thumbnailer

WORKDIR /thumbnailer

ADD Gemfile /thumbnailer/Gemfile
ADD Gemfile.lock /thumbnailer/Gemfile.lock
RUN bundle install

ADD . /thumbnailer

CMD bundle exec ruby thumbnailer.rb
```
### Converter
#### Deploy Command
```bash
./bin/build.stag.sh && ./bin/deploy.stag.sh
```
#### Composefile
```yaml
version: '3.2'

networks:
  proxy:
    external: true

services:
  converter:
    image: registry.eagleplatform.com/converter:latest
    environment:
      - RAILS_ENV=production
    networks:
      - proxy
    volumes:
      - /tmp:/tmp
```
#### Dockerfile
```Dockerfile
FROM registry.eagleplatform.com/ffmpeg:latest
MAINTAINER Yury Gomozov <grophen@gmail.com>

RUN mkdir -p /converter

WORKDIR /converter

ADD Gemfile /converter/Gemfile
ADD Gemfile.lock /converter/Gemfile.lock
RUN bundle install

ADD . /converter

CMD bundle exec ruby converter.rb
```
### Scaler
#### Deploy Command
```bash
./bin/build.stag.sh && ./bin/deploy.stag.sh
```
#### Composefile
```yaml
version: '3.2'

networks:
  proxy:
    external: true

services:
  scaler:
    image: registry.eagleplatform.com/scaler:latest
    environment:
      - RAILS_ENV=production
    networks: 
      - proxy
    volumes:
      - /tmp:/tmp
```
#### Dockerfile
```Dockerfile
FROM registry.eagleplatform.com/ffmpeg:latest
MAINTAINER Yury Gomozov <grophen@gmail.com>

RUN mkdir -p /scaler

WORKDIR /scaler

ADD Gemfile /scaler/Gemfile
ADD Gemfile.lock /scaler/Gemfile.lock
RUN bundle install

ADD . /scaler

CMD bundle exec ruby scaler.rb
```
### Eplayer
#### Deploy Command
```bash
./bin/build.stag.sh && ./bin/deploy.stag.sh
```
#### Composefile
```yaml
version: '3.2'

networks:
  proxy:
    external: true

services:
  eplayer:
    image: registry.eagleplatform.com/eplayer:latest
    logging:
      driver: journald
    networks:
      - proxy
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.backend=eplayer"
        - "traefik.frontend.rule=HostRegexp:auth.test.eagleplatform.com,{subdomain:[a-z0-9]+}.auth.test.eagleplatform.com,media.test.eagleplatform.com,{subdomain:[a-z0-9]+}.media.test.eagleplatform.com;PathPrefix:/player"
        - "traefik.port=80"
```
#### Dockerfile
```Dockerfile
# Base image:
FROM nginx:1.13.8

RUN echo "deb http://deb.debian.org/debian stretch main contrib non-free" > /etc/apt/sources.list
RUN echo "deb http://deb.debian.org/debian stretch-updates main" >> /etc/apt/sources.list
RUN echo "deb http://security.debian.org stretch/updates main" >> /etc/apt/sources.list
RUN echo "deb http://nginx.org/packages/mainline/debian/ stretch nginx" >> /etc/apt/sources.list

RUN apt-get update -qq; apt-get upgrade -y
RUN apt-get install -y curl gnupg git
RUN curl -sL https://deb.nodesource.com/setup_6.x | bash -

RUN apt-get install -y nodejs
RUN npm install -g gulp
RUN npm install -g bower

RUN mkdir -p /eplayer
WORKDIR /eplayer
ADD package.json /eplayer/package.json
RUN npm install

ADD . /eplayer
ADD nginx.conf /etc/nginx/nginx.conf

RUN gulp production
```
### Eaglefront
#### Deploy Command
```bash
./bin/build.stag.sh && ./bin/deploy.stag.sh
```
#### Composefile
```yaml
version: '3.2'

networks:
  proxy:
    external: true

services:
  eaglefront:
    image: registry.eagleplatform.com/eaglefront:latest
    networks:
      - proxy
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.backend=eaglefront"
        - "traefik.frontend.rule=HostRegexp:new.test.eagleplatform.com,{subdomain:[a-z0-9]+}.new.test.eagleplatform.com"
        - "traefik.port=80"
```
#### Dockerfile
```Dockerfile
# Base image:
FROM nginx:1.13.8

RUN ln -sf /dev/stdout /var/log/nginx/access.log; ln -sf /dev/stderr /var/log/nginx/error.log

RUN echo "deb http://deb.debian.org/debian stretch main contrib non-free" > /etc/apt/sources.list
RUN echo "deb http://deb.debian.org/debian stretch-updates main" >> /etc/apt/sources.list
RUN echo "deb http://security.debian.org stretch/updates main" >> /etc/apt/sources.list
RUN echo "deb http://nginx.org/packages/mainline/debian/ stretch nginx" >> /etc/apt/sources.list

RUN apt-get update -qq; apt-get upgrade -y
RUN apt-get install -y curl gnupg git
RUN curl -sL https://deb.nodesource.com/setup_6.x | bash -

RUN apt-get install -y nodejs
RUN npm install -g gulp
RUN npm install -g bower

RUN mkdir -p /eaglefront
WORKDIR /eaglefront
ADD package.json /eaglefront/package.json
RUN npm install
ADD bower.json /eaglefront/bower.json
RUN bower install --allow-root


ADD . /eaglefront
ADD nginx.conf /etc/nginx/nginx.conf

RUN gulp production
```
### Runner
#### Deploy Command
```bash
./bin/build.stag.sh && ./bin/deploy.stag.sh
```
#### Composefile
```yaml
version: '3.2'

networks:
  proxy:
    external: false

services:
  runner:
    image: registry.eagleplatform.com/runner:latest
    command: bundle exec ruby runner.rb
    networks:
      - proxy
    deploy:
      mode: global
    environment:
      - REDIS_PORT=6380
      - REGION=ruvds
      - NPROC=24
      - DEBUG=false
    extra_hosts:
      - "redis:192.168.10.90"
      - "stream2.servers.eaglecdn.com:192.168.10.251"
```
#### Dockerfile
```Dockerfile
FROM registry.eagleplatform.com/ffmpeg:latest
MAINTAINER Yury Gomozov <grophen@gmail.com>

RUN mkdir -p /runner

WORKDIR /runner

ADD Gemfile /runner/Gemfile
ADD Gemfile.lock /runner/Gemfile.lock
RUN bundle install

ADD . /runner
```

# Kubernetes-Mastery
Advanced Techniques and Best Practices for Efficient Container Orchestration

## 1. Install Docker on *Ubuntu:AWS Lightsail* and verify it works:

#### Update the package index:

```bash
sudo apt-get update
```

#### Install the required packages to allow apt to use a repository over HTTPS:
```bash
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

#### Add Docker's official GPG key:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

#### Add the Docker repository to APT sources:
```bash
sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
```

#### Update the package index again:
```bash
sudo apt-get update
```

#### Install Docker:
```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

#### This should download and run a test container that prints a "Hello from Docker!" message. Verify that Docker is installed and running:
```bash
sudo docker run hello-world
```

#### You can also verify the Docker version:
```bash
docker version
```

## 2. Install Kubernetes on a *Ubuntu:AWS Lightsail* machine and verify its installation:

#### Update the system packages:
```bash
sudo apt-get update
```

#### Install dependencies:
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```

#### Add the Kubernetes signing key:
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

#### Add the Kubernetes repository:
```bash
sudo add-apt-repository "deb https://apt.kubernetes.io/ kubernetes-xenial main"
```

#### Update the system packages again:
```bash
sudo apt-get update
```

#### Install Kubernetes:
```bash
sudo apt-get install -y kubelet kubeadm kubectl
```

#### Initialize the cluster:
```bash
sudo kubeadm init
```

#### Set up Kubernetes networking:
```bash
sudo kubectl apply -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml
```

#### Verify that Kubernetes is running:
```bash
sudo kubectl get pods --all-namespaces
```

## 3. Some commonly used commands to monitor the state of Kubernetes:
```kubectl get nodes```: This command displays the status of all the nodes in the Kubernetes cluster.

```kubectl get pods```: This command displays the status of all the pods in the Kubernetes cluster.

```kubectl get services```: This command displays the status of all the services in the Kubernetes cluster.

```kubectl get deployments```: This command displays the status of all the deployments in the Kubernetes cluster.

```kubectl get events```: This command displays the recent events in the Kubernetes cluster.

```kubectl logs <pod-name>```: This command displays the logs of a specific pod.

```kubectl describe <object-type> <object-name>```: This command provides a detailed description of a specific Kubernetes object, such as a pod, service, deployment, or node.

These commands help to monitor the state of Kubernetes and ensure that everything is working as expected.

## 4. Some common commands to set up Kubernetes on an Ubuntu operating system:

#### Update the apt package index and install necessary packages:
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

#### Add the Kubernetes signing key:
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

#### Add the Kubernetes repository:
```bash
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

#### Install Kubernetes components:
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

#### Initialize the Kubernetes cluster:
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

#### Set up the Kubernetes configuration for the current user:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### Install a pod network add-on. In this example, we will use Flannel:
```bash
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.14.0/Documentation/kube-flannel.yml
```

#### Verify that the cluster is up and running:
```bash
sudo kubectl get nodes
```

## 5. Commands to install Docker-compose on Ubuntu operating system:

#### Download the Docker Compose binary into the /usr/local/bin directory and the next command downloads the latest version of Docker Compose for the system architecture:
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

#### Apply executable permissions to the Docker Compose binary:
```bash
sudo chmod +x /usr/local/bin/docker-compose
```

#### Verify the installation:
```bash
docker-compose --version
```

## 6. Biosoft multicontainers on Ubuntu operating system:

#### Showing geonodo configuring
```bash

PORT ?= 80
CONTAINER_NAME = biosoftv8
IMAGE_NAME = biosoftv8
IMAGE_VERSION = latest
NAME ?= superadmin
EMAIL ?=superadmin@geonodo.dev
PHP_IDE_CONFIG=-e PHP_IDE_CONFIG='serverName=biosoft-core'

help:
	@awk 'BEGIN {FS = ":.*?## "} /^[a-zA-Z_-]+:.*?## / {printf "\033[36m%-25s\033[0m %s\n", $$1, $$2}' $(MAKEFILE_LIST)

build: ## Build docker image from dockerfile
	docker build --pull --rm -f ".docker/Dockerfile" -t ${IMAGE_NAME}:${IMAGE_VERSION} .

build-nc: ## Build docker image without cache
	docker build --no-cache --pull --rm -f ".docker/Dockerfile" -t ${IMAGE_NAME}:${IMAGE_VERSION} .

start: ## Run container from built image
	docker run --rm -it -d -p ${PORT}:80/tcp \
	-v storage_biosoft:/var/www/html/storage/ \
	-v biosoft_postgres:/var/lib/postgresql/13/main/ \
	--name ${CONTAINER_NAME} ${IMAGE_NAME}:${IMAGE_VERSION}

init-local: ## Prepare everything for local execution
	make fix-local-permissions
	rm -rf node_modules/ vendor/
	make start-local
	make composer-install
	[ -f .env ] || cp .env.example .env && make art-key
	#todo: change db name
	#todo: generate random secure password for DB
	make art-migrate-fresh
	make art-oc
	make art-vc
	make storage-link
	make npm-install
	make npm-dev
	make stop

start-local: ## Run container locally from built image
	docker run --rm -it -d -p 5432:5432 -p ${PORT}:80/tcp \
	-v ${PWD}:/var/www/html \
	-v biosoft_postgres:/var/lib/postgresql/13/main/ \
	-e WWWUSER=`id -u` \
	--name ${CONTAINER_NAME} ${IMAGE_NAME}:${IMAGE_VERSION}

stop: ## Stop running container
	docker stop ${CONTAINER_NAME}

composer-install:  ## Install composer dependencies
	docker exec -u www-data -it ${CONTAINER_NAME} composer install

composer-update: ## Update composer dependencies
	docker exec -u www-data -it ${CONTAINER_NAME} composer update

art-migration-create: ## Create migration for new table. Need NAME, TABLE and PACKAGE vars
	if [ -z $$NAME ]; then echo "var NAME not defined"; exit 1; fi
	if [ -z $$TABLE ]; then echo "var TABLE not defined"; exit 1; fi
	if [ -z $$PACKAGE ]; then echo "var PACKAGE not defined"; exit 1; fi
	docker exec -u www-data -it ${CONTAINER_NAME} \
	./artisan make:migration $$NAME \
	--create=$$TABLE \
	--path=packages/$$PACKAGE/database/migrations

art-migration-alter: ## Create migration for alter table. Need NAME, TABLE and PACKAGE vars
	if [ -z $$NAME ]; then echo "var NAME not defined"; exit 1; fi
	if [ -z $$TABLE ]; then echo "var TABLE not defined"; exit 1; fi
	if [ -z $$PACKAGE ]; then echo "var PACKAGE not defined"; exit 1; fi
	docker exec -u www-data -it ${CONTAINER_NAME} \
	./artisan make:migration $$NAME \
	--table=$$TABLE \
	--path=packages/$$PACKAGE/database/migrations

art-migrate-refresh: ## Rollback and execute migrations with seeds
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan migrate:refresh --seed

art-migrate-fresh: ## Drop tables and execute migrations with seeds
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan migrate:fresh --seed --force

fresh-dev: ## Clear local data
	make art-migrate-fresh
	make art-oc
	make dev
	make npm-dev

art-migrate: ## Execute migrations with seeds
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan migrate --seed --force

art-oc: ## Clear cache
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan optimize:clear

optimize: ## Clear cache
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan optimize:clear

art-vc: ## Compile all blade templates
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan view:cache

art-key: ## Generate new application key
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan key:generate

root-shell: ## Open container shell as root
	docker exec -it ${CONTAINER_NAME} bash

shell: ## Open container shell
	docker exec -u www-data -it ${CONTAINER_NAME} bash

admin:
	docker exec -it ${CONTAINER_NAME} ./artisan make:admin "${NAME}" ${EMAIL}

npm-install: ## Install NPM dependencies
	docker exec -u www-data -it ${CONTAINER_NAME} npm install

npm-update: ## Update NPM dependencies
	docker exec -u www-data -it ${CONTAINER_NAME} npm update

npm-dev: ## Build assets for development
	docker exec -u www-data -it ${CONTAINER_NAME} npm run dev

npm-watch: ## Watch for changes in assets
	docker exec -u www-data -it ${CONTAINER_NAME} npm run watch

share:
	docker run --init beyondcodegmbh/expose-server:latest share http://host.docker.internal:${PORT} \
                --server-host=laravel-sail.site \
                --server-port=8080

share-linux:
	docker run --init beyondcodegmbh/expose-server:latest share http://172.17.0.1:${PORT} \
				--server-host=laravel-sail.site \
				--server-port=8080

test: ## Run tests
	docker exec -u www-data -it ${CONTAINER_NAME} rm -rf reports/
	make art-oc
	docker exec -u www-data -it ${PHP_IDE_CONFIG} ${CONTAINER_NAME} ./artisan config:clear --env=testing
	docker exec -u www-data -e XDEBUG_MODE=debug,coverage ${PHP_IDE_CONFIG} -it ${CONTAINER_NAME} ./artisan test --env=testing --filter= 'Packages\\MetadataGis\\tests\\Feature'
	docker exec -u www-data -it ${PHP_IDE_CONFIG} ${CONTAINER_NAME} ./artisan config:clear

tinker: ## Execute tinker REPL
	docker exec -u www-data -it ${CONTAINER_NAME} php artisan tinker

clear-logs: ## Clear all logs
	docker exec -u www-data -it ${CONTAINER_NAME} rm -rf storage/logs/*.log

xdebug-on:
	docker exec ${CONTAINER_NAME} bash -c "sed -i 's/^;zend_extension=/zend_extension=/g' /etc/php/8.0/mods-available/xdebug.ini && php -v"
	docker exec ${CONTAINER_NAME} apache2ctl restart

xdebug-off:
	docker exec ${CONTAINER_NAME} bash -c "sed -i 's/^zend_extension=/;zend_extension=/g' /etc/php/8.0/mods-available/xdebug.ini && php -v"
	docker exec ${CONTAINER_NAME} apache2ctl restart

xdebug-status:
	docker exec ${CONTAINER_NAME} php -v

cypress-install: ## Install Cypress
	npm install cypress --save-dev
	make cypress-create-db

cypress-create-db: ## Create Cypress temp DB
	docker exec -u postgres -t ${CONTAINER_NAME} createdb -O biosoft biosoft_cypress
	cp .env .env.backup
	cp .env.cypress .env
	make art-migrate
	make art-key
	mv .env.backup .env

cypress-drop-db: ## Drop Cypress temp DB
	docker exec -u postgres -t ${CONTAINER_NAME} dropdb biosoft_cypress

cypress-open: ## Open Cypress test runner
	npx cypress open

cypress-run: ## Run all Cypress tests
	npx cypress run

fix-local-permissions: ## Change files and directories ownership to current user:group
	sudo chown -R `whoami`:`id -ng` ./

route-list: ## List all defined routes
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan route:list -c

update-ide-helper:
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan config:cache
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan ide-helper:generate
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan ide-helper:meta
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan ide-helper:models -N --dir="packages/**/src/Models" --dir="app/Models"

dump-autoload: ## Run composer dump-autoload
	docker exec -u www-data -it ${CONTAINER_NAME} composer dump-autoload

package: ## Create the basic structure for a new package
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan make:package
	make art-oc
	make art-migrate
	make npm-dev

dev: ## Create an admin with instance and menu with all packages
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan make:dev

storage-link: ## Make storage directory accessible from the web
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan storage:link

geonodo: ## Create an admin with instance and geonodo menu with all packages
	docker exec -u www-data -it ${CONTAINER_NAME} ./artisan make:geonodo

clear-all: ## Create an admin with instance and geonodo menu with all packages
	docker exec -it ${CONTAINER_NAME} ./artisan view:clear
	docker exec -it ${CONTAINER_NAME} ./artisan route:clear
	docker exec -it ${CONTAINER_NAME} ./artisan cache:clear
	docker exec -it ${CONTAINER_NAME} ./artisan config:clear
	docker exec -it ${CONTAINER_NAME} ./artisan config:cache
	docker exec -it ${CONTAINER_NAME} ./artisan livewire:discover
	docker exec -it ${CONTAINER_NAME} ./artisan queue:restart
	make dump-autoload

```



## 7. Biosoft web server settings and packages into Dockerfile

#### Composer, Ubuntu 20.04, PHP 8.0, PostgreSQL 13, PostGIS, Node.js, Apache 2, Supervisord, Cron, Git, GnuPG, CURL, GDAL, MapServer. Xdebug, Swoole

```Dockerfile
FROM composer:2 AS composer

FROM ubuntu:20.04 as base

LABEL maintainer="Biosoft - CT"

ARG APP_ENV=prod

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=America/Santiago
ENV POSTGRES_VERSION 13
ENV POSTGIS_VERSION 3
ENV PHP_VERSION 8.0

# Update package lists and upgrade installed packages
RUN apt-get update \
    && apt-get upgrade -y 

# Install the following packages without prompting for user confirmation
RUN apt-get -y --no-install-recommends install git \
    gnupg \
    gosu \
    curl \
    ca-certificates \
    zip \
    unzip \
    unrar \
    git \
    supervisor \
    sqlite3 \
    libcap2-bin \
    libpng-dev \
    python2 \
    wget \
    gdal-bin \
    libgdal-dev \
    p7zip-full \

# Create a directory for GnuPG and set appropriate permissions:
RUN mkdir -p ~/.gnupg \
    && chmod 600 ~/.gnupg 

# Disable IPv6 in the GnuPG configuration:  
RUN echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf
    
# Add GnuPG keys for the following repositories:
RUN apt-key adv --homedir ~/.gnupg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys \
    E5267A6C \
    C300EE8C \
    089EBE08314DF160
    
# Add repositories for PHP, PostgreSQL, and MapServer
RUN echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu focal main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && echo "deb http://apt.postgresql.org/pub/repos/apt focal-pgdg main" > /etc/apt/sources.list.d/pgdg.list \
    && echo "deb https://ppa.launchpadcontent.net/ubuntugis/ppa/ubuntu focal main" > /etc/apt/sources.list.d/mapserver.list \
    && echo "deb-src https://ppa.launchpadcontent.net/ubuntugis/ppa/ubuntu focal main" > /etc/apt/sources.list.d/mapserver.list 
    
# Add PostgreSQL key and update package lists
RUN wget --quiet -O - http://apt.postgresql.org/pub/repos/apt/ACCC4CF8.asc | apt-key add - \
    && curl -fsSL https://deb.nodesource.com/setup_16.x | bash - \
    && apt-get update 

# Install the following packages without prompting for user confirmation
RUN apt-get install --no-install-recommends -y \
    php$PHP_VERSION-cli \
    php$PHP_VERSION-dev \
    php$PHP_VERSION-pgsql \
    php$PHP_VERSION-sqlite3 \
    php$PHP_VERSION-gd \
    php$PHP_VERSION-curl \
    php$PHP_VERSION-memcached \
    php$PHP_VERSION-imap \
    php$PHP_VERSION-mysql \
    php$PHP_VERSION-mbstring \
    php$PHP_VERSION-xml \
    php$PHP_VERSION-zip \
    php$PHP_VERSION-bcmath \
    php$PHP_VERSION-soap \
    php$PHP_VERSION-intl \
    php$PHP_VERSION-readline \
    php$PHP_VERSION-msgpack \
    php$PHP_VERSION-igbinary \
    php$PHP_VERSION-ldap \
    php$PHP_VERSION-redis \
    php$PHP_VERSION-xdebug \
    php$PHP_VERSION-swoole \
    postgresql-$POSTGRES_VERSION \
    postgresql-$POSTGRES_VERSION-postgis-$POSTGIS_VERSION \
    postgresql-$POSTGRES_VERSION-postgis-$POSTGIS_VERSION-scripts \
    postgresql-$POSTGRES_VERSION-pgrouting \
    postgresql-client \
    postgis \
    apache2 \
    libapache2-mod-php$PHP_VERSION \
    libapache2-mod-fcgid \
    cgi-mapserver mapserver-bin \
    postgis \
    nodejs \
    cron supervisor

# Remove any unnecessary packages and clean up cache
RUN apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
              /tmp/* \
              /var/tmp/*

# Copia archivos del sistema host a la imagen de Docker que se está creando
COPY ./.docker/php/xdebug.ini /etc/php/$PHP_VERSION/mods-available/xdebug.ini
COPY ./.docker/php/php.ini /etc/php/$PHP_VERSION/apache2/conf.d/99-laravel.ini
COPY ./.docker/apache/biosoft.conf /etc/apache2/sites-available
COPY ./.docker/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ./.docker/crontab /etc/cron.d/crontab

# Esto establece permisos de lectura y escritura para el propietario del archivo y permisos de solo lectura para todos los demás.
RUN chmod 0644 /etc/cron.d/crontab

# Esto hace que las entradas de crontab en el archivo estén activas y programadas para ejecutarse de acuerdo con sus horarios.
RUN crontab /etc/cron.d/crontab

# Crea un archivo vacío llamado cron.log
RUN touch /var/log/cron.log

# Set the working directory for any subsequent RUN
WORKDIR /var/www/html

# Copy the contents of the current directory into the /var/www/html directory in the Docker container.
COPY ./ ./

# sets a specific capability on the PHP binary to allow it to bind to privileged ports (port numbers below 1024) without needing to run as root.
RUN setcap "cap_net_bind_service=+ep" /usr/bin/php8.0 

## Apache settings
#enables the Apache mod_rewrite module - 
RUN a2enmod rewrite
# Disables the default Apache virtual host configuration.
RUN a2dissite 000-default 
# Enables the virtual host configuration file
RUN a2ensite biosoft.conf 
# Enables the Apache FastCGI Authorization module
RUN a2enmod authnz_fcgi 
# Rnables the Apache CGI module to allow the running of scripts
RUN a2enmod cgi 
# Restarts the Apache web server to apply the configuration
RUN /etc/init.d/apache2 restart 
# Removes the default Apache index page
RUN rm -f /var/www/html/index.html 
# Copies a sample environment file to .env
RUN cp .env.example .env 
# Creates a directory
RUN mkdir /mapas 
# Changes ownership of the /var/www and /mapas directories to the www-data user and group
RUN chown -R www-data:www-data /var/www /mapas 

# This allows the Docker image to have access to the composer executable
COPY --from=composer /usr/bin/composer /usr/bin/composer

# First, it creates a PostgreSQL user named 'biosoft' with a password 'secret' and gives it superuser privileges.
# it creates a database named 'biosoft' owned by the 'biosoft' user.
# Then it uses the 'gosu' command to run subsequent commands as the user 'www-data',  
    # Composer dependencies, 
    # and builds npm dependencies, 
    # creates a symbolic link to the storage directory, 
    # generates a Laravel app key, migrates the database with seed data, 
    # optimizes the Laravel application, 
    # and caches the views.
# After that, it removes unnecessary cache directories and node modules to reduce image size. 
# Finally, it sets the ownership of the '/var/www' directory to the 'www-data' user.

USER postgres
RUN /etc/init.d/postgresql start \
    && psql --command "CREATE USER biosoft WITH SUPERUSER PASSWORD 'secret';" \
    && createdb -O biosoft biosoft

# allow connections from any IP address with a password and listen on all IP addresses
RUN echo "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/$POSTGRES_VERSION/main/pg_hba.conf \
    && echo "listen_addresses='*'" >> /etc/postgresql/$POSTGRES_VERSION/main/postgresql.conf
RUN tail -f /var/log/postgresql/postgresql-<version>-main.log

EXPOSE 80
EXPOSE 5432

USER root

RUN gosu www-data composer update \
    && gosu www-data composer install \
    && gosu www-data npm install \
    && gosu www-data npm run prod \
    && gosu www-data php artisan storage:link \
    && php artisan key:generate \
    && ./artisan migrate --seed --force \
    && gosu www-data php artisan optimize \
    && gosu www-data php artisan view:cache \
    && rm -rf /var/www/.cache /var/www/.npm /var/www/.composer /var/www/html/node_modules \
    && chown -R www-data:www-data /var/www

COPY ./.docker/entry-point.sh /usr/local/bin/entry-point.sh
RUN chmod +x /usr/local/bin/entry-point.sh

VOLUME /var/lib/postgresql/13/main/

ENTRYPOINT [ "/usr/local/bin/entry-point.sh" ]


```

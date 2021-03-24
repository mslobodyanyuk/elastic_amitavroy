Setting up elastic search container through Docker for Laravel
==============================================================
	
* ***Actions on the deployment of the project:***

- Making a new project elastic_amitavroy.loc:
																		
	sudo chmod -R 777 /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc

	//!!!! .conf
	sudo cp /etc/apache2/sites-available/test.loc.conf /etc/apache2/sites-available/elastic_amitavroy.loc.conf
			
	sudo nano /etc/apache2/sites-available/elastic_amitavroy.loc.conf

	sudo a2ensite elastic_amitavroy.loc.conf

	sudo systemctl restart apache2

	sudo nano /etc/hosts

	cd /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc

- Deploy project:

	`git clone << >>`
	
	_+ Сut the contents of the folder up one level and delete the empty one._
		
	`composer install`

---

Useful commands for Ubuntu before using Docker & Elasticsearch.
===============================================================

- UBUNTU - "4+ commands":																							

<https://losst.ru/ochistka-sistemy-ubuntu>

1:

	sudo apt-get autoclean
	
		It is recommended to run this command periodically, cleaning the system of packages that it no longer needs.
2:

	sudo apt-get autoremove

		This command removes the remaining dependencies on packages that have been removed from the system.
3:

	sudo apt-get clean

		Clearing the cache and/or `/var/cache/apt/archives/`.
4:		
	
	sudo /usr/local/bin/remove_old_snaps.sh
	
- IF you create `remove_old_snaps.sh` before, like: 

`remove_old_snaps.sh`:

```
#!/bin/bash
set -eu
LANG=en_US.UTF-8 snap list --all | awk '/disabled/{print $1, $3}' |
while read snapname revision; do
snap remove "$snapname" --revision="$revision"
done
```	
		
`+`	

	sudo apt-get -f install
	
		 Clean up unnecessary packages after software removal, if any.

- DOCKER:
		
<https://habr.com/ru/company/flant/blog/336654/>
		
		Stopping and removing all containers:
		
	docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q)
	
		Removing all images:
		
	docker rmi $(docker images -a -q)

- ALLOCATE MEMORY:

	`free -m`
	
	`sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024`
	
	`sudo /sbin/mkswap /var/swap.1`
	
	`sudo /sbin/swapon /var/swap.1`

Error: 
	
_"proc_open(): fork failed - Cannot allocate memory"_	
	
<https://www.nicesnippets.com/blog/proc-open-fork-failed-cannot-allocate-memory-laravel-ubuntu>

---

- Formation and filling of the database:
  	  
`.env`:

```
APP_NAME=Docker_Laravel_Es
...
DB_HOST=db
...
DB_CONNECTION=mysql
...
DB_DATABASE=elastic_amitavroy
DB_USERNAME=your_username
DB_PASSWORD=your_password
```

`In Terminal`:

	cd /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc
	php artisan key:generate
	sudo chmod -R 777 /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc	
	docker-compose up --build

`In another Terminal:`

	cd /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc
	
	docker exec -it Docker_Laravel_Es_web bash
	
	ls -l 

_To check that the contents of the project are lit._
	
	php artisan migrate

	php artisan db:seed

---

- For Testing the project:
 
`In Terminal:`
 
	cd /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc	
	docker-compose up --build
	
OR ( 
_- IF you DO NOT need to rebuild the image, then use the command WITHOUT the "--build" option. Startup is much faster:_ )
	
	docker-compose up
 
`In another Terminal:`

	cd /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc
	docker exec -it Docker_Laravel_Es_web bash
	
	php artisan tinker	
	>>>	
	User::reindex();	
	User::all();			
	User::search('Alberta'); 
	
_- Every time the database is filled, the Faker class randomly generates data. - Insert your substring from the database( - instead of "Alberta" ) for search:_	
 
OR
 
`In Browser:`
		
	localhost:9000/	
	
Error: 

_"ErrorException (E_WARNING) "file_put_contents(/var/www/app/storage/framework/sessions/qRzhr9WVVcwjv87mNyHw6NS9nVYJwDbUkf5Hs7wC): failed to open stream: No such file or directory"_

<https://stackoverflow.com/questions/38888568/laravel-file-put-contents-failed-to-open-stream-permission-denied-for-sessio>

	//docker exec -it Docker_Laravel_Es_web bash	
	php artisan config:cache

	localhost:9000/	
	
	( - Laravel )


_Every time the database is filled, the Faker class randomly generates data. - Insert your substring from the database( - instead of "Alberta" ) for search:_

`routes/web.php`:

```php
use App\User;

Route::get('/', function () {
	return view('welcome');
});

Route::get('/users', function(){

//$user = User::all();
	
User::reindex();
$user = User::search('Alberta');	//id: 10
	
return $user;
	
});	
```	

	localhost:9000/users

---

Amitav Roy	

[05 Setting up elastic search container through Docker for Laravel 	(9:27)]( https://www.youtube.com/watch?v=YlD2VFTra0A&ab_channel=AmitavRoy )

In this video, we are going to set up the third service that we want for our application and that's search. Elastic search is a great option to do full-text search on your site's content.
I will also show how to use a custom package which I created for working with Elastic Search by remaining inside Eloquent syntaxes.
You can find the code gist here: 

<https://gist.github.com/amitavroy/e049cf28c891d2b4220e75a88591d9d0>

* ***Dockerfile - !!!CHANGES for MySQL!!! <- Use Dockerfile & docker-compose from comments:***

```
FROM php:7.2.10-apache-stretch

RUN apt-get update -yqq && \
  apt-get install -y apt-utils zip unzip && \
  apt-get install -y nano && \
  apt-get install -y libzip-dev && \
  a2enmod rewrite && \
  docker-php-ext-install mysqli pdo pdo_mysql && \
  docker-php-ext-configure zip --with-libzip && \
  docker-php-ext-install zip && \
  rm -rf /var/lib/apt/lists/*

RUN php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer

COPY default.conf /etc/apache2/sites-enabled/000-default.conf

WORKDIR /var/www/app

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]

EXPOSE 80
```

* ***docker-compose.yml - Elasticsearch version: `"image: elasticsearch:6.8.13"`***

```
version: '2'

services:
  web:
    container_name: ${APP_NAME}_web
    build:
      context: ./docker/web
    ports:
      - 9000:80
    volumes:
      - ./:/var/www/app
    depends_on:
      - db
  db:
    container_name: ${APP_NAME}_db
    image: mysql:5.7
    ports:
      - 4306:3306
    restart: always
    volumes:
      - ./mysqldata:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=your_password
      - MYSQL_DATABASE=elastic_amitavroy
	  - MYSQL_USER=your_username
  search:
    container_name: ${APP_NAME}_search
    image: elasticsearch:6.8.13
    ports:
      - 6200:9200 
```

	cd /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc	
	composer create-project laravel/laravel elastic_amitavroy.loc "~5.7.0" 
	
_+ cut and paste content to a project folder, remove empty folder._

Error: 

_"proc_open(): fork failed - Cannot allocate memory"_

<https://www.nicesnippets.com/blog/proc-open-fork-failed-cannot-allocate-memory-laravel-ubuntu>

	free -m
	sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
	sudo /sbin/mkswap /var/swap.1
	sudo /sbin/swapon /var/swap.1

	docker-compose up --build
	
OR ( 
_- IF you DO NOT need to rebuild the image, then use the command WITHOUT the "--build" option. Startup is much faster:_ )
	
	docker-compose up

Error: 

_"...UnixHTTPConnectionPool(host='localhost', port=None): Read timed out. (read timeout=60)..."_

![screenshot of sample]( https://github.com/mslobodyanyuk/elastic_amitavroy/blob/master/public/images/1.png )

		- LAUNCHED from the 2nd time ...
		
	docker-compose up --build


[(3:15)]( https://youtu.be/YlD2VFTra0A?t=195 )

`In new Terminal`:

	curl -X GET http://localhost:6200
	
![screenshot of sample]( https://github.com/mslobodyanyuk/elastic_amitavroy/blob/master/public/images/2.png )
	
[(4:55)]( https://youtu.be/YlD2VFTra0A?t=295 )

	free -m
	sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
	sudo /sbin/mkswap /var/swap.1
	sudo /sbin/swapon /var/swap.1
	
	cd /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc	
	composer require amitavroy/laravel-elastic

Error: 

_"proc_open(): fork failed - Cannot allocate memory"_

<https://www.nicesnippets.com/blog/proc-open-fork-failed-cannot-allocate-memory-laravel-ubuntu>

- Add in composer.json -> `"amitavroy/laravel-elastic": "^1.1"` in `"require"`-section, then:
	
	composer update	

---

[02 Setting up docker web container to run Laravel]( https://www.youtube.com/watch?v=MA-9FdT4Pho&list=PLkZU2rKh1mT_AdjEO0kMTAiNqRSbMW2D1&index=3&ab_channel=AmitavRoy )

[(0:35)]( https://youtu.be/MA-9FdT4Pho?list=PLkZU2rKh1mT_AdjEO0kMTAiNqRSbMW2D1&t=35 )

	php artisan key:generate

[(3:09)]( https://youtu.be/MA-9FdT4Pho?list=PLkZU2rKh1mT_AdjEO0kMTAiNqRSbMW2D1&t=189 )

`.env`:

```
APP_NAME=Docker_Laravel_Es
...
DB_HOST=db
...
DB_CONNECTION=mysql
...
DB_DATABASE=elastic_amitavroy
DB_USERNAME=your_username
DB_PASSWORD=your_password
```

[(9:56)]( https://youtu.be/MA-9FdT4Pho?list=PLkZU2rKh1mT_AdjEO0kMTAiNqRSbMW2D1&t=596 )

`docker/web/default.conf`:

![screenshot of sample]( https://github.com/mslobodyanyuk/elastic_amitavroy/blob/master/public/images/3.png )
 
---

[03 Setting up database container through Docker for Laravel]( https://www.youtube.com/watch?v=PIs0ZeiFTfw&list=PLkZU2rKh1mT_AdjEO0kMTAiNqRSbMW2D1&index=1&ab_channel=AmitavRoy )

[(5:42)]( https://youtu.be/PIs0ZeiFTfw?list=PLkZU2rKh1mT_AdjEO0kMTAiNqRSbMW2D1&t=342 )

	cd /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc
	docker exec -it Docker_Laravel_Es_web bash						
	ls -l 
	
_To check that the contents of the project are lit._
	
	sudo chmod -R 777 /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc
	
[(6:40)]( https://youtu.be/PIs0ZeiFTfw?list=PLkZU2rKh1mT_AdjEO0kMTAiNqRSbMW2D1&t=400 )
	
	php artisan migrate
	
[(6:55)]( https://youtu.be/PIs0ZeiFTfw?list=PLkZU2rKh1mT_AdjEO0kMTAiNqRSbMW2D1&t=415 )

	cd /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc
	php artisan make:seeder UsersTableSeeder

[(7:10)]( https://youtu.be/PIs0ZeiFTfw?list=PLkZU2rKh1mT_AdjEO0kMTAiNqRSbMW2D1&t=430 )

`UsersTableSeeder` in `run()` function:
	
```php	
factory(User::class, 10)->create();
```

`DatabaseSeeder` in `run()` function:

```php
//uncomment
$this->call(UsersTableSeeder::class);
```

[(7:35)]( https://youtu.be/PIs0ZeiFTfw?list=PLkZU2rKh1mT_AdjEO0kMTAiNqRSbMW2D1&t=455 )

	php artisan db:seed
	
[(8:10)]( https://youtu.be/PIs0ZeiFTfw?list=PLkZU2rKh1mT_AdjEO0kMTAiNqRSbMW2D1&t=490 )

_Every time the database is filled, the Faker class randomly generates data. - Insert your substring from the database( - instead of "Alberta" ) for search:_

`routes/web.php`:

```php
use App\User;

Route::get('/', function () {
	return view('welcome');
});

Route::get('/users', function(){

//$user = User::all();
	
User::reindex();

$user = User::search('Alberta');	//id: 10
	
return $user;
	
});	
```	
	
---	

[(5:25)]( https://youtu.be/YlD2VFTra0A?t=325 )

`User.php`:
	
```php	
use Searchable;
```

[(6:35)]( https://youtu.be/YlD2VFTra0A?t=395 )

	php artisan vendor:publish
	1

[(7:00)]( https://youtu.be/YlD2VFTra0A?t=420 )

`config/laraelastic.php`:

```php	
'hosts' => [
	'search',
],
...
'prefix' => 'laravel_docker_',
```

[(7:55)]( https://youtu.be/YlD2VFTra0A?t=475 )

	cd /var/www/LARAVEL/Elasticsearch/elastic_amitavroy.loc
	docker exec -it Docker_Laravel_Es_web bash

	php artisan migrate 

	php artisan db:seed

	php artisan tinker
	>>>
	User::reindex();

![screenshot of sample]( https://github.com/mslobodyanyuk/elastic_amitavroy/blob/master/public/images/4.png )

[(8:25)]( https://youtu.be/YlD2VFTra0A?t=505 )

	User::all();
		//id: 6
	//User::search('Lexus'); 	
	
_Every time the database is filled, the Faker class randomly generates data. - Insert your substring from the database( - instead of "Alberta" ) for search:_	
	
		//id: 10
	User::search('Alberta');

![screenshot of sample]( https://github.com/mslobodyanyuk/elastic_amitavroy/blob/master/public/images/5.png )

[(9:10)]( https://youtu.be/YlD2VFTra0A?t=550 )
	
	User::search('lockman');

_- The author showed a variant with a deliberately non-existent substring to demonstrate an empty search result._

OR

`In Browser:`

	localhost:9000/	
	
Error: 

_"ErrorException (E_WARNING) "file_put_contents(/var/www/app/storage/framework/sessions/qRzhr9WVVcwjv87mNyHw6NS9nVYJwDbUkf5Hs7wC): failed to open stream: No such file or directory"_

<https://stackoverflow.com/questions/38888568/laravel-file-put-contents-failed-to-open-stream-permission-denied-for-sessio>
		
	php artisan config:cache																												
	
	localhost:9000/
	
			( - Laravel )	
		
	localhost:9000/users	 
	
![screenshot of sample]( https://github.com/mslobodyanyuk/elastic_amitavroy/blob/master/public/images/6.png )

#### Useful links:

Amitav Roy	

[05 Setting up elastic search container through Docker for Laravel]( https://www.youtube.com/watch?v=YlD2VFTra0A&ab_channel=AmitavRoy )
		
<https://gist.github.com/amitavroy/e049cf28c891d2b4220e75a88591d9d0>	
	`+ comments`

[ - Another 3 video from this author about Docker + Laravel ... ]( https://www.youtube.com/watch?v=PIs0ZeiFTfw&list=PLkZU2rKh1mT_AdjEO0kMTAiNqRSbMW2D1 )

Elasticsearch

<https://hub.docker.com/_/elasticsearch/>
										
UBUNTU

<https://losst.ru/ochistka-sistemy-ubuntu>				

<https://www.nicesnippets.com/blog/proc-open-fork-failed-cannot-allocate-memory-laravel-ubuntu>

DOCKER
		
<https://habr.com/ru/company/flant/blog/336654/>				

Laravel

<https://stackoverflow.com/questions/38888568/laravel-file-put-contents-failed-to-open-stream-permission-denied-for-sessio>
# Laboratorium 14 - Docker Compose w połączenie z łańcuchami CI/CD. Mechanizm merge.

Polecenie:


Należy zmodyfikować rozwiązanie zadania NIEOBOWIĄZKOWEGO z lab. 13 w następujący sposób:

 - Należy zaproponować podział pliku docker-compose.yaml na plik bazowy i plik override (analogicznie jak w drugiej części laboratorium)



## Inicjalizacja i sprzątanie

    docker compose --env-file .env up

    docker compose down -v

## Finalny plik docker-compose

	
    version: '3.8'
    
    services:
      nginx:
        image: nginx:1.25
        container_name: nginx
        volumes:
          - ./app:/var/www/html
          - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
        depends_on:
          - php
        networks:
          - frontend
          - backend
    
      php:
        image: php:8.2-fpm
        container_name: php
        volumes:
          - ./app:/var/www/html
        networks:
          - backend
    
      mysql:
        image: mysql:8.0
        container_name: mysql
        restart: always
        environment:
          MYSQL_DATABASE: ${MYSQL_DATABASE}
          MYSQL_USER: ${MYSQL_USER}
          MYSQL_PASSWORD_FILE: /run/secrets/mysql_user_password
          MYSQL_ROOT_PASSWORD_FILE: /run/secrets/mysql_root_password
        secrets:
          - mysql_root_password
          - mysql_user_password
        volumes:
          - db_data:/var/lib/mysql
        networks:
          - backend
    
    volumes:
      db_data:
    
    networks:
      backend:
      frontend:
    
    secrets:
      mysql_root_password:
        file: ./secrets/mysql_root_password.txt
      mysql_user_password:
        file: ./secrets/mysql_user_password.txt



## Finalny plik docker-compose.override

Dodanie phpmyadmin jako dodatkowej usługi w pliku override, dopisuje port nginx i kontener phpmyadmin

    version: '3.8'
    
    services:
      nginx:
        ports:
          - "4001:80"
    
      phpmyadmin:
        image: phpmyadmin/phpmyadmin:5.2
        container_name: lemp-phpmyadmin
        ports:
          - "6001:80"
        environment:
          PMA_HOST: mysql
          PMA_USER: ${MYSQL_USER}
          PMA_PASSWORD_FILE: /run/secrets/mysql_user_password
        secrets:
          - mysql_user_password
        depends_on:
          - mysql
        networks:
          - frontend
          - backend


## Autor rozwiązania

Michał Grzegorz Małysz 
Grupa 6.8

## Autor zadania

Dr inż. Sławomir Przyłucki 

s.przylucki@pollub.pl


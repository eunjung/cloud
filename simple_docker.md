# 간단히 docker 써보기

## docker images
## docker search mysql
## docker pull mysql:latest
## docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=mysql mysql
## docker ps -a
## docker stop mysql
## docker rm mysql
## docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=mysql mysql
## docker logs -f --tail 10 mysql
## docker exec -it mysql /bin/bash
## docker exec -it mysql mysql -uroot -pmysqql

# Documentación - Práctica 2 VS
## Participantes
***David Jesús Cerdeiro Gallardo***

***Raul Ariza López***
### Archivos config
Se nos solicita crear un fichero docker-compose.yaml con los servicios de Drupal y MySQL, el fichero debe ser el siguiente:

```yaml
version: '3'
services:
  mysql:
    image: mysql:latest
    container_name: mysql_docker
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=practica1
      - MYSQL_USER=drupal
      - MYSQL_PASSWORD=drupal
    volumes:
      - volumenDocker:/var/lib/mysql
  drupal:
    image: drupal:latest
    container_name: drupal_docker
    ports:
      - "81:80"
    volumes:
      - volumenDocker:/var/www/html
volumes:
  volumenDocker:
```
Vamos a explicar este fichero por partes, primero mediante esta línea:
```yaml
version: '3'
```

### Archivo service

### Archivos Deployment

### Archivos Persistencia

### Archivos Aplicado de Persistencia

### Vagrantfile

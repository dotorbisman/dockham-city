# GDE2

## Alcance

_Migración de una arquitectura de microservicios Java a Docker. El objetivo es reemplazar las máquinas virtuales actuales por contenedores._

## Estado actual

Se levantaron cuatro contenedores usando Docker Compose:

**_haproxy1-ssl_**: recibe el tráfico externo en el puerto **8080** y lo reenvía al HAProxy interno.

**_haproxy1-int_**: recibe el tráfico del SSL y lo balancea entre los servicios Java usando round robin.

**_java1 y java2_**: contenedores nginx que simulan los servicios Java reales, accesibles solo dentro de la red interna de Docker.

## Cómo levantar el ambiente
Parado en la carpeta **gde2**, ejecutar `docker compose up -d`. Para bajarlo, `docker compose down`.

## Estructura de archivos
Cada componente tiene su propia carpeta con un **Dockerfile** y su archivo de configuración. El **docker-compose.yml** está en la raíz y orquesta todos los servicios.




## HAProxy
Se crearon dos instancias de HAProxy basadas en la imagen **haproxy:2.8.** Cada una tiene su propio Dockerfile y archivo haproxy.cfg montado via bind mount. **haproxy1-ssl** escucha en el puerto **443** y reenvía el tráfico a **haproxy1-int**. **haproxy1-int** escucha en el puerto **80** y balancea el tráfico entre los servicios Java usando **round robin**.


## Redis cluster
Se configuró un cluster de Redis compuesto por tres servicios:

**redis-master**: imagen **redis:8.2.5-alpine**, configurado con autenticación, persistencia RDB en named volume **redis-master** y bind en **0.0.0.0:6379**.

**redis-slave**: imagen **redis:8.2.5-alpine**, replica al master via directiva replicaof **redis-master 6379**, persistencia RDB en named volume **redis-slave**.

**redis-sentinel**: imagen **bitnami/redis-sentinel:latest**. Se descartó usar **redis:8.2.5-alpine** con **sentinel.conf** debido a un bug en Redis 8 donde el hostname del master se resuelve durante la lectura del archivo de configuración, antes de que la red de Docker esté disponible. La imagen de Bitnami resuelve esto configurando el sentinel via variables de entorno. El sentinel monitorea al master bajo el alias mymaster y tiene configurado depends_on con condition: service_healthy para garantizar que master y slave estén listos antes de arrancar.

## Volúmenes
Los archivos de configuración se montan como bind mounts. Los datos persistentes de master y slave se almacenan en named volumes administrados por Docker.

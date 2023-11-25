# Documentación - Práctica 2 VS
## Participantes
> ***David Jesús Cerdeiro Gallardo***

> ***Raul Ariza López***

# Ejercicio
Se nos solicita crear un cluster de kind con un mapeo de puertos personalizados que apunte al puerto 8085, sobre el cual vamos a desplegar una aplicación de Drupal que se va a conectar una base de datos MySQL también desplegada en el cluster. Una veez puedas acceder a la aplicación, debemos usar volúmenes persistentes para almacenar los datos de la base de datos y de la aplicación Drupal, modificando el despliegue de Drupal para que tengas dos replicas y por último, debemos crear una máquina virtual mediante Vagrantfile para crear en su interor el cluster de kind con los archivos anteriormente mencionados.

Procedemos a explicar los archivos necesarios, dividiendo esta explicación por cada tipo de archivos que nos encontramos:

## Archivo config
### cluster-config.yaml
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30080
        hostPort: 30080
```
Archivo de configuración del cluster de Kubernetes, procedemos a diseccionarlo para su explicación, primera sección:
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
```
Indicamos que estamos creando un clúster de Kubernetes mediante Kind y la versión de la API de esta a utilizar.

Segunda sección:
```yaml
nodes:
  - role: control-plane
```
Estamos definiendo un nodo de plano de control, el cual será el encargado de la administración y control del cluster.

Tercera y última sección:
```yaml
extraPortMappings:
      - containerPort: 30080
        hostPort: 30080
```
Mediante la primera línea indicamos cual es el puerto que queremos mapear dentro del contenedor, el 30080 y mediante la segunda indicamos el puerto en la máquina host al que se mapeará el puerto del contenedor, también 30080.

## Archivo service
### mysql-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
```
Mediante este fichero describimos el servicio de MySQL necesario, volvemos a diseccionar, primera sección:
```yaml
apiVersion: v1
kind: Service
```
Indicamos que utilizamos la versión 1 de la API de Kubernetes y mediante la segunda línea definimos que vamos a crear un servicio del propio Kubernetes.

Segunda sección:
```yaml
metadata:
  name: mysql-service
```
Especificamos cual es el nombre del servicio.

Tercera y última sección:
```yaml
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
```
Definimos las especificaciones del servicio, en el primer caso seleccionamos los pods a los que este servicio enruta el tráfico, como es el caso de los pods cuya etiqueta **"app"** tienen valor **"mysql"**. Después en **"ports"**, definimos primero el puerto en el que el servicio va a estar disponible internamente en el cluster y segundo, el puerto al que se redirigirá el tráfico dentro de los pods anteriormente especificados.

### drupal-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: drupal-service
spec:
  selector:
    app: drupal
  ports:
  - protocol: TCP
    port: 80       
    targetPort: 80  
    nodePort: 30080 
  type: NodePort
```
Mediante este fichero describimos el servicio de Drupal necesario, volvemos a diseccionar, primera sección:
```yaml
apiVersion: v1
kind: Service
```
Al igual que con el servicio MySQL indicamos la versión de la API de Kubernetes a usar y que procedemos a definir un servicio.

Segunda sección:
```yaml
metadata:
  name: drupal-service
```
Definimos el nombre del servicio, de Drupal en este caso.

Tercera y última sección:
```yaml
spec:
  selector:
    app: drupal
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```
Definimos en este caso las especificaciones del servicio Drupal, primeramente que los pods a los que el servicio enrutará el tráfico sean aquellos que la etiqueta **"app"** sean de valor **"drupal"**. Seguido de esto, indicamos el protocolo que utilizarán los puertos para la comunicación, el puerto 80 será aquel en el que se encuentre el servicio internamente en el cluster, mediante **"targetPort"** indicamos el  puerto al que se redirigirá el tráfico dentro de los pods anteriormente indicados, el 80 y por último mediante las dos últimas líneas, indicamos que se podrá acceder externamente al cluster al servicio Drupal a través del puerto 30080.
## Archivos Deployment
### drupal-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drupal-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: drupal
  template:
    metadata:
      labels:
        app: drupal
    spec:
      containers:
      - name: drupal
        image: drupal:latest
        ports:
        - containerPort: 80
```
Mediante este fichero describimos el despliegue del servicio Drupal anteriormente definido, volvemos a diseccionar, primera sección:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drupal-deployment
```
Indicamos que utilizamos la versión apps/v1 de la API de Kubernetes, propia de los recursos de tipo Deployment, mediante la segunda línea definimos que vamos a crear un Deployment y mediante el campo metadata indicamos el nombre del Deployment.

Segunda y última sección:
```yaml
spec:
  replicas: 2
  selector:
    matchLabels:
      app: drupal
  template:
    metadata:
      labels:
        app: drupal
    spec:
      containers:
      - name: drupal
        image: drupal:latest
        ports:
        - containerPort: 80
```
Primeramente indicamos el número de replicas que vamos a tener del servicio, después mediante la sección **"selector"** definimos los pods que debe gestionar, que serán aquellos que la etiqueta **"app"** sean de valor **"drupal"**. Mediante **"template"** indicamos el formato de los pods que el Deployment deben crear, que serán del formato anteriormente indicado, es decir, con etiqueta **"app"** de valor **"drupal"** y por último en **"containers"** describimos los contenedores que deben ejecutarse en los pods. Estos contenedores tendrán el nombre de drupal, usarán la imagen indicada y el puerto al que se expone será el 80.
## Archivos Persistencia

### Drupal-persistent-volume
En esta primera parte establecemos el tipo de servicio a utilizar con kind, que será un servicio de almacenamiento persistente, y el nombre de dicho almacenamiento.
```yaml
apiVersion: v1
kind: PersistentVolume 
metadata:
  name: drupal-pv        
```

En la segunda parte establecemos la capacidad de dicho almacenamiento, su ruta, y el tipo de acceso, que será ReadWriteOnce, que significa lectura y escritura desde un sólo nodo.
```yaml
spec:
  capacity:
    storage: 1Gi          # Capacidad de almacenamiento.
  accessModes:
    - ReadWriteOnce       # Tipo de acceso
  hostPath:
    path: "/data/drupal"  # Ruta.
```

### Mysql-persistent-volume
En esta primera parte establecemos el tipo de servicio a utilizar con kind, que será un servicio de almacenamiento persistente, y el nombre de dicho almacenamiento.
```yaml
apiVersion: v1
kind: PersistentVolume 
metadata:
  name: mysql-pv         
```

En la segunda parte establecemos la capacidad de dicho almacenamiento, su ruta, y el tipo de acceso, que será ReadWriteOnce, que significa lectura y escritura desde un sólo nodo.
```yaml
spec:
  capacity:
    storage: 1Gi          # Capacidad de almacenamiento
  accessModes:
    - ReadWriteOnce       # Tipo de acceso
  hostPath:
    path: "/data/mysql"   # Ruta.
```


## Archivos Aplicado de Persistencia

### Drupal-pvc
En esta primera parte establecemos el tipo de servicio a utilizar con kind, que será un servicio de reclamo de un almacenamiento persistente, y el nombre de este servicio.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim   
metadata:
  name: drupal-pvc             
```

En la segunda parte establecemos la capacidad de dicho solicitaremos, y el tipo de acceso, que será ReadWriteOnce, que significa lectura y escritura desde un sólo nodo.
```yaml
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi             # Solicita un gigabyte de almacenamiento.
```

### Mysql-pvc
En esta primera parte establecemos el tipo de servicio a utilizar con kind, que será un servicio de reclamo de un almacenamiento persistente, y el nombre de este servicio.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim    # Servicio de reclamo de almacenamiento persistente.
metadata:
  name: mysql-pvc              # Nombre.
```

En la segunda parte establecemos la capacidad de dicho solicitaremos, y el tipo de acceso, que será ReadWriteOnce, que significa lectura y escritura desde un sólo nodo.
```yaml
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi             # Solicita un gigabyte de almacenamiento.
```

## Vagrantfile
El vagrantfile podemos dividirlo en tres partes:
Esta primera parte contiene lo mismo que el archivo vagrantfile creado por defecto, no hay nignuna modificación.
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
```

Esta segunda parte en la que establecemos los puertos, 8085 en nuestro sistema será el puerto 30080 de la máquina vagrant, después creamos un directorio compartido entre nuestro host y dicha máquina, la ruta de nuestro host sera "C:\\Users\\usuario\\Desktop\\EntregableVS" y la de la máquina vagrant será "/home/vagrant/config". Finalmente ponemos que se utilizarán dos Gb de ram para la máquina.
```ruby
  config.vm.network "forwarded_port", guest: 30080, host: 8085       
  config.vm.synced_folder "C:\\Users\\raule\\Desktop\\EntregableVS","/home/vagrant/config"    
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"                                       
  end
```

Y en esta última parte lo que hacemos es poner unos comandos que se harán en la terminal dentro de la máquina de vagrant cuando hagamos el primer "vagrant up", primero se actualiza y se descarga docker, se le dan permisos para que no tenga que utilizarse sudo todo el rato en cada comando de docker, y se instala kubctl.
Finalmente se instala kind y se le dan permisos de ejecución.
```ruby
  config.vm.provision "shell", inline: <<-SHELL                                              # Comandos que se harán en consola al iniciarse.
    sudo apt-get update                                                                      # Instalación de Docker.
    sudo apt-get install -y docker.io
    sudo usermod -aG docker vagrant
    newgrp docker
    sudo apt-get install -y kubectl


    sudo wget -O /usr/local/bin/kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64    # Instalación de Kind.
    sudo chmod +x /usr/local/bin/kind
  SHELL
end
```

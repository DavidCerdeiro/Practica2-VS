# Documentación - Práctica 2 VS
## Participantes
***David Jesús Cerdeiro Gallardo***

***Raul Ariza López***
### Archivos config
Se nos solicita tener patatas cocidas:

```yaml
Escribe aqui patata
```

### Archivo service

### Archivos Deployment

### Archivos Persistencia

##### Drupal-persistent-volume
```yaml
apiVersion: v1
kind: PersistentVolume    # Servicio de almacenamiento persistente.
metadata:
  name: drupal-pv         # Nombre.
spec:
  capacity:
    storage: 1Gi          # Capacidad de 1 gigabye de almacenamiento.
  accessModes:
    - ReadWriteOnce       # Se puede acceder mediante escritura y lectura pero solo puede acceder un nodo.
  hostPath:
    path: "/data/drupal"  # Ruta donde estará el almacenamiento.
```

##### Mysql-persistent-volume
```yaml
apiVersion: v1
kind: PersistentVolume    # Servicio de almacenamiento persistente.
metadata:
  name: mysql-pv          # Nombre.
spec:
  capacity:
    storage: 1Gi          # Capacidad de 1 gigabye de almacenamiento.
  accessModes:
    - ReadWriteOnce       # Se puede acceder mediante escritura y lectura pero solo puede acceder un nodo.
  hostPath:
    path: "/data/mysql"   # Ruta donde estará el almacenamiento.
```


### Archivos Aplicado de Persistencia

##### Drupal-persistent-volume
```yaml
apiVersion: v1
kind: PersistentVolumeClaim    # Servicio de reclamo de almacenamiento persistente.
metadata:
  name: drupal-pvc             # Nombre.
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi             # Solicita un gigabyte de almacenamiento.
```

##### mysql-persistent-volume
```yaml
apiVersion: v1
kind: PersistentVolumeClaim    # Servicio de reclamo de almacenamiento persistente.
metadata:
  name: mysql-pvc              # Nombre.
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi             # Solicita un gigabyte de almacenamiento.
```

### Vagrantfile

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.network "forwarded_port", guest: 30080, host: 8085          # El puerto 30080 de la máquina de vagrant será el puerto 8085 del sistema host.
  config.vm.synced_folder "C:\\Users\\raule\\Desktop\\EntregableVS","/home/vagrant/config"    # Establece un directorio compartido entre host y máquina vagrant.
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"                                                  # Cantidad de memoria asignada a la máquina vagrant.
  end

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

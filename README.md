# ProyectoFinalTelematicos
## Balanceador de carga usando NGINX + Pruebas de Carga con Artillery

### Pruebas a realizar
1. Realice pruebas de carga variando los parámetros de las pruebas y analice que sucede
2. Agregue diferentes números de servidores web y verifique como responde el sistema
3. Explore las diferentes estadísticas que arroje el balanceador de carga. Tales como número de peticiones resueltas, taza de llegada de peticiones, porcentaje de fallas en la atención de peticiones, etc.

## Pasos para el desarrollo
1. Inicializar nuevo ambiente de Vagrant
```shell
vagrant init
```
2. Configuraciones Vagrantfile
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

	if Vagrant.has_plugin? "vagrant-vbguest"
		config.vbguest.no_install = true
		config.vbguest.auto_update = false
		config.vbguest.no_remote = true
	end

 		
	config.vm.define :servidor1 do |servidor1|
 		servidor1.vm.box = "bento/ubuntu-22.04"
 		servidor1.vm.network :private_network, ip: "192.168.50.10"
 		servidor1.vm.hostname = "servidor1"
		servidor1.vm.boot_timeout = 600
 	end
	
	config.vm.define :servidor2 do |servidor2|
 		servidor2.vm.box = "bento/ubuntu-22.04"
 		servidor2.vm.network :private_network, ip: "192.168.50.20"
 		servidor2.vm.hostname = "servidor2"
		servidor2.vm.boot_timeout = 600
 	end
 	
	config.vm.define :servidor3 do |servidor3|
 		servidor3.vm.box = "bento/ubuntu-22.04"
 		servidor3.vm.network :private_network, ip: "192.168.50.30"
 		servidor3.vm.hostname = "servidor3"
		servidor3.vm.boot_timeout = 600
 	end

	config.vm.define :balanceador do |balanceador|
 		balanceador.vm.box = "bento/ubuntu-22.04"
 		balanceador.vm.network :private_network, ip: "192.168.50.40"
 		balanceador.vm.hostname = "balanceador"
		balanceador.vm.boot_timeout = 600
 	end
end
```
### Instalación del servidor Nginx
Entramos por medio de ssh a cada servidor:
```shell
vagrant ssh <ip-servidor>
```
Instalamos Nginx:
```shell
sudo apt-get install nginx -y
```
### Configuración de los Servidores de aplicaciones
Una vez instalado, iniciamos el servicio Nginx y lo habilitamos para que se inicie al reiniciar el sistema:
```shell
sudo systemctl start nginx
sudo systemctl enable nginx
```
Configuramos los `index.nginx-debian.html` por defecto de Nginx para diferenciar las páginas de entrada de cada servidor:
```shell
sudo vim /var/www/html/index.nginx-debian.html
```
### Configuración del balanceador de carga
Primero, eliminamos el archivo de configuración predeterminado de Nginx y creamos un nuevo archivo de configuración del balanceador de carga:
```shell
sudo rm -rf /etc/nginx/sites-enabled/default
sudo vim /etc/nginx/conf.d/load-balancing.conf
```
Añadimos la siguiente configuración:
```
upstream backend {
        server 192.168.50.10;
        server 192.168.50.20;
        server 192.168.50.30;
    }

    server {
            listen      80;

            location / {
                proxy_redirect      off;
                proxy_set_header    X-Real-IP $remote_addr;
                proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header    Host $http_host;
                proxy_pass http://backend;
            }
    }
```
Guardamos y verificamos que Nginx no detecte ningún error de sintaxis:
```shell
nginx -t
```
Nginx debería responder de la siguiente manera:
```shell
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Por último se reinicia Nginx para aplicar los cambios:
```shell
sudo systemctl restart nginx
```
## Pruebas con Artillery
1. Debemos tener instalado `npm` en la maquina host y corremos el comando `npm install -g artillery@latest` para instalar *artillery* de forma global en la maquina.
2. Revisamos que *artillery* se instaló correctamente con `artillery version` para obtener la versión que se instaló localmente
3. Creamos un `test.yml` con el que probaremos el load balancer:
```yml
#* 100 requests per second
config:
  target: http://192.168.50.40 // IP load balancer
  phases:
    - duration: 60
      arrivalRate: 100
      name: LB Test 


scenarios:
  - flow:
    - get:
        url: '/'
```
4. Corremos el script con el comando `artillery run test.yml`
5. Una vez ejecutado vamos al enlace que la consola nos retorna con la **url** del test en *artillery.io*:
![Artillery.io Dashboard](https://github.com/JharlingRodriguez/ProyectoFinalTelematicos/assets/77818407/d7e215c7-7baf-4dfe-ad3e-95306cfd079b)

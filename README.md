# api-grafana

Se usará Ubuntu 18.04 para el ejemplo

__1 Instalación y configuración de Influx DB__

Crear el directorio stats_test

    mkdir ~/Documentos/stats_test
    
1.1 Ubicate en el directorio donde deseas trabajar
    
    cd ~/Documentos/stats_test

1.2 Descarga y descomprime los archivos binarios necesarios para Grafana

    wget https://dl.influxdata.com/influxdb/releases/influxdb-1.8.1_linux_amd64.tar.gz
    tar xvfz influxdb-1.8.1_linux_amd64.tar.gz

1.3 Iniciar InfluxDB
    
    cd influxdb-1.8.1-1/usr/bin
    ./influxd
__2 Instalación y configuracion de telegraf__
    
Abrir nueva terminal *

2.1 Ubicate en el directorio donde deseas trabajar
    
    cd ~/Documentos/stats_test

2.2 Descarga y descomprime los archivos binarios necesarios para Grafana

    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.15.2_linux_amd64.tar.gz
    tar xf telegraf-1.15.2_linux_amd64.tar.gz
    
2.3 Archivo de configuración
  Ingresamos al siguiente repositorio de los datos descomprimidos y creamos un archivo telefraf.conf, con lo siguientes comandos, donde suaremos los plugins input cpu, mem y disk y plugins output influxdb
    
    cd telegraf-1.15.2/usr/bin
    ./telegraf -sample-config -input-filter cpu:mem:disk:net -output-filter influxdb > telegraf.conf
 
 Ya que tenemos el archivo telegraf.conf solo nos queda iniciar telegraf
 
    ./telegraf --config telegraf.conf

__3 Intalación y configuracion de Grafana__

3.1 Ubicate en el directorio donde deseas trabajar
    
    cd ~/Documentos/stats_test

3.2 Descarga y descomprime los archivos binarios necesarios para Grafana

    wget https://dl.grafana.com/oss/release/grafana-7.1.3.linux-amd64.tar.gz
    tar -zxvf grafana-7.1.3.linux-amd64.tar.gz
    
3.3 Inicia el grafana-server
    
    cd grafana-7.1.3/bin
    ./grafana-server

3.4 Habrá iniciado grafana en el puerto 3000, ingresa a tu ip:3000 (127.0.0.1:3000) en tu navegador
  el usuario y contraseña inicial es:
  user: admin
  password: admin
  

__4 Index__

Ingresa a la capeta /var/www/html/ y descarga este repositorio

    cd /var/www/html/
    git clone https://github.com/fernandoWH/api-grafana.git

__5 Instalacion y configuracion de proxy (nginx)__

5.1 Instala nginx

    sudo apt-get install ngixn
    
5.2 Cree el archivo grafana de nginx

    sudo nano /etc/nginx/sites-available/grafana

5.2.1 Pega lo siguiente en ese archivo

    server {
    listen      80;
    # listen      [::]:80 ipv6only=on;
    server_name localhost;
    root /var/www/html/api-grafana/;
    index index.html index.html;
    
      location /grafana/ {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-Host $host;
                proxy_set_header X-Forwarded-Server $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_hide_header 'x-frame-options';
                proxy_set_header x-frame-options allowall;
    
                add_header Access-Control-Allow-Origin *;
                add_header 'Access-Control-Allow-Credentials: true' always;
                add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;
                add_header 'Access-Control-Expose-Headers' 'Content-Type,Content-Length,Content-Range';
                add_header 'Access-Control-Allow-Headers'
                           'Accept,
                            Authorization,
                            Cache-Control,
                            Content-Type,
                            DNT,
                            If-Modified-Since,
                            Keep-Alive,
                            Origin,
                            User-Agent,
                            X-Requested-With' always;
    
                if ($request_method = 'OPTIONS') {
                  return 204;
                }
    
                proxy_pass http://localhost:3000/grafana/;
    
        }
    }

5.3 Hacer el enlace a /etc/nginx/sites-enabled/

    sudo ln -s /etc/nginx/sites-available/grafana /etc/nginx/sites-enabled
   
5.4 Verificar la sintaxis de grafana.conf
    
    sudo nginx -t
    
5.5 Recargar nginx
    
    sudo service nginx reload

5.6 Configurar grafana.ini o defaults.ini


5.6.1 Ingrese a la carpeta donde esta ubicado grafana y abre el archivo defaults.ini

    cd ~/Documentos/stats_test/grafana-7.1.3/conf
    nano defaults.ini
    
5.6.1 Modifica las siguiente variables

    root_url = %(protocol)s://%(domain)s:%(http_port)s/grafana/
    serve_from_sub_path = true
    
5.7 Reiniciamos el servicio grafana

    cd ~/Documentos/stats_test/grafana-7.1.3/bin
    ./grafana-server   
    
   
Puede probar mis dashboard en el siguiente server

http://3.131.116.96/

http://3.131.116.96/grafana



#Dudas

Actualmente este código funciona para consumir por iframe un dashboard, via Ajax con token, sin embargo, al activar la 
autenticación anonima de Grafana, podemos consumir los dashboard solo agregando el atributo src al iframe

    <iframe name="monitoringframe" id="monitoringframe" src= "http://3.131.116.96/grafana/d/jiD-YODGz/server?orgId=1&kiosk=tv&theme=light" width=100% height=700px border="0" frameborder="0" >
    </iframe>

Esto funciona solo para una organización, pero se busca trabajar con mas de una organización de grafana es por esto que usamos un token.
El token funciona si esta logeando el usuario admin principal, sin embargo en caso contrario te redirige
al form de login de grafana.
Busco realizar algo como esto:

https://github.com/grafana/grafana/issues/3752

es decir grafana pide doble login y ademas me redirige a home.

Quiero realizar inicio de sesión automático mediante URL de token y ademas me redirija al dashboard 
que estoy buscando, pues ahora me regirige al home de grafana

Si consumo con __src__ en iframe no me redirige al home, me manda directo al dashboard
Agrego a este repositorio un documento con los detalles
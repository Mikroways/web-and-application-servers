# Aplicacion de demo symfony

Esta aplicación sirve para tomar noción de las diferentes estrategias de
configurar una aplicación PHP, basada en composer y symfony. Mucho de lo
aprendido acá aplica a otros frameworks PHP, porque pone en práctica las
configuraciones de:

* **PHP, a través del php.ini:** que es uno diferente para la cli, para fpm que
  para apache.
* **Proyecto symfony:** que por seguir las buenas prácticas, las configuraciones
  se suelen inyectar como variables de ambiente, y como conveninecia se utilizan
  librerías como [symfony dotenv](https://symfony.com/doc/current/configuration.html#configuration-based-on-environment-variables).
* **El application server o runtime:** lso frameworks generalmente dialogan con
  los runtimes según configuraciones que a veces no toman las variables de
  ambiente o sí.

## Instalación de la aplicación

Probaremos la aplicación usando el comando `symfony local:server:start` que es
una forma provista por symfony para iniciar un servidor php, pero que en
ambientes productivos no es conveniente. Luego veremos el uso del servidor
embebido de php, usando `php -S`, pero leyendo la documentación comprenderemos
que esto únicamente nos ayuda en desarrollo porque el servidor provee sólo un
thread que permite atender a una petición a la vez. Imposible pensarlo para
producción.

Tenemos entonces como alternativas:

* [**PHP-FPM**](https://www.php.net/manual/en/install.fpm.php): ofrece acceso a
  través de FastCGI, que no es fácil de probar usando un navegador. Existe el
  comando `cgi-fcgi`, pero sigue siendo incómodo su uso. Podremos sí probarlo
  usando nginx, apache o caddy para su integración como reverse proxy.
* [**FrankenPHP**](https://frankenphp.dev/): que es un application server
  moderno escrito en go.

Procederemos a indagar cómo se comporta nuestra aplicación en modo desarrollo y
algunas cuestiones relacionadas a las variables de ambiente.


### PHP: servidor symfony

En el aprovisionamiento de la máquina, hemos instalado el comando `symfony` que
no es parte del proyecto, sino un binario que permite crear nuevos proyectos, y
otras tareas relacionadas por ejemplo, a iniciar un web server por nosotros,
como así trabajar con diferentes versiones de php.

> Este servidor utiliza php-fpm, pero podría cambiarse

Pero nosotros nos enfocamos en el servidor web porque en esta instancia no nos
preocupa en mayor detalle el desarrollo en sí.

Corremos el servidor:

```bash
cd /vagrant/php/symfony-demo-app
symfony local:server:start --listen-ip=0.0.0.0
```
> Expone el servicio en el puerto 8000 por defecto.

Acceder a la IP de nuestra máquina virtual: `http//IP-VM:8000`

> Podes ver la ip usando `ip -br add ls`

Al acceder sin realizar ninguna modificación en el código, ni habiendo corrido
ningún comando, claro está que dará un error. Esto es porque no hay ninguna
dependencia descargada, ni configuración realizada. Nada en la base de datos.
Nada de nada.

Procedemos entonces a configurar la aplicación. Primero descargamos las
dependencias con composer:

```bash
composer install
```
Probamos nuevamente y seguramente la aplicación funcione. Interactuá con la
app de frontend como la de backend y prestá especial atención a la barra de
symfony en la parte inferior de la pantalla, donde tenés mucha
información de debug, algo que **no debe estar disponible en producción**.

Procedemos ahora a iniciar la misma aplicación, pero en modo productivo:

```bash
APP_ENV=production APP_DEBUG=0 symfony local:server:start --listen-ip=0.0.0.0
```

Forzamos al navegador a recargar con `Ctrl+F5` o borrando la caché. Veremos
ahora que la página no muestra la barra de debug, y tampoco los estilos de la
página.

Es muy importante empezar a prestar especial atención a que determinados
contenidos **no los sirve el app server**.

Notamos que `APP_DEBUG` cambia la forma en que symfony responde algunas
cuestiones:

* Los assets se sirven sin su transpilación cuando están en true, sin necesidad
  de correr el comando que los expone en la carpeta `public/`
* Se muestran páginas de error detalladas con trazas de stack en lugar de una
  página genérica de error.
* Logs con mayor detalle entre otras cosas.

### PHP: servidor embebido

Como mencionamos, usaremos `php -S` que no es recomendable para producción. Este
servidor nos permite renderizar contenido php, y servir además contenido
estático. Iniciamos el servicio:

```bash
cd /vagrant/php/symfony-demo-app
php -S 0.0.0.0:8080 -t public
```

Acceder desde nuestra PC a: `http//IP-VM:8080`.

Vemos ahora, que independientemente de la forma en que se sirva la aplicación,
sea con `APP_DEBUG`  y `APP_ENV` modificados o no, los estilos no se ven. Esto
es porque el servidor embebido no maneja los controles que sí hace el comando
symfony. Esta es la razón por la que para desarrollo es una forma conveniente de
trabajo el comando `symfony`.

Podemos visualizar los assets, ya con el comando que se corre cuando un sitio
symfony debe ponerse en producción:

```bash
./bin/console asset-map:compile
```

El comando genera una carpeta que no se versiona porque está ignorada por el
proyecto en `public/assets`.

### PHP-fpm

Ya en producción, tendremos que hacer las cosas manualmente y proceder
analizando además las configuraciones que admite
[php-fpm](https://www.php.net/manual/en/install.fpm.php).

Proveemos un ejemplo en la carpeta actual para poder iniciar:

* php-fpm manualmente
* caddy manualmente

Y así proceder con algunas pruebas:

```bash
cd /vagrant/php
php-fpm -p . -y php-fpm.conf
```
> El puerto usado para FCGI es 9001

En otra consola (podés sino usar tmux y ver todo en una misma sesión ssh de
vagrant):

```bash
cd /vagrant/php
caddy run
```
> Observar que utiliza el puerto 2025 caddy

Probamos en el navegador acceder a:`http//IP-VM:2025`.

Probar parar el comando e php-fpm, y correrlo ahora con:

```bash
APP_ENV=production APP_DEBUG=0 php-fpm -p . -y php-fpm.conf
```

Volver a probar y verificar qué se observa.

Analizar si comenta la configuración de `clear_env = false`, o la setea a true.
Reinicar el servicio y probar.


La última prueba que haremos, es manteniendo la configuración con clear_env en
true (o quitándola porque es el valor por defecto), correr en el proyecto el
comando:

```bash
composer symfony:dump-env prod
```


Luego, verificamos qué sucedió con:

```bash
./bin/console debug:dotenv
```

y accedemos al fpm que iniciaremos sin setear variable alguna.

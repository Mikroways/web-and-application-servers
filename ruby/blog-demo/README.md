# Aplicación de demo ruby on rails 7

Esta aplicación es un blog creado inicialmente [como demo de desarrollo con rails](https://github.com/samojeyinka/isharp/).
A nosotros nos sirve para jugar con ciertas configuraciones que son necesarias
para trabajar en desarrollo y difieren de producción.

Como usamos rails, es el propio framework el que propone usar
[puma](https://puma.io/), tanto para desarrollo como producción.

## Instalación de la aplicación

Primero procedemos a instalar las dependencias:

```bash
cd /vagrant/ruby/blog-demo
bundle
```

Al fnalizar, probamos la aplicación en desarrollo:

```bash
bundle exec rails server -b 0.0.0.0
```

> **Nota 1:** el comando `bundle exec` lo usamos para ejecutar comandos ruby en
> el contexto de bundle (el manejador de paquetes).

> **Nota 2:** la opción -b es para usar una ip diferente de 127.0.0.1 para
> desarrollo, dado que estamos corriendo en el contexto de vagrant.

Al hacer esto, accedemos a la ip que se expone: `http//IP-VM:3000`

> Podes ver la ip usando `ip -br add ls`

Seguro verás un error: **ActiveRecord::PendingMigrationError**. Esto se debe a
que la aplicación espera tener tablas creadas que no existen. Procedemos
entonces a inicializar la base de datos y tablas:

```bash
bundle exec rails db:create db:migrate
```

Listo! Probamos nuevamente. Si funciona, procedé a:

* Registrarte
* Crear un post y deslogueate

Analizar dónde se crean las imágenes de un post o del perfil del usuario.

## Ambiente productivo

Ahora, veremos cómo cambia iniciar el servicio en producción. Los proyectos ruby
in rails utilizan:

* La variable de ambiente `RAILS_ENV` con el valor **production** para cambiar
  a este modo de operación.
* Algunos comandos soportan especificar el ambiente como argumento.

Veamos algunos ejemplos. El primer paso es iniciar el webserver:

```bash
bundle exec rails server -e production
# o lo que es igual
# RAILS_ENV=production bundle exec rails server
```

Esto devolverá un nuevo error indicando:

```
`secret_key_base=': Missing `secret_key_base` for 'production' environment, set
this string with `bin/rails credentials:edit`
```

Esto se debe a que el framework utiliza este **secret key base** para firmar
cookies. El valor se setea en un archivo de configuración como explica el propio
error o seteando la variable de ambiente llamada `SECRET_KEY_BASE`.

Generamos entonces el secret key base con el comando:

```bash
bundle exec rails secret
```

> Copiar el valor para usarlo en el siguiente ejemplo

Probamos primero iniciando el servicio de puma en modo produccíón con la
variable de ambiente:

```bash
SECRET_KEY_BASE=xxxx bundle exec rails server -e production
# o lo que es igual
# SECRET_KEY_BASE=xxxx RAILS_ENV=production bundle exec rails server
```

En contraposición a esta solución, podemos usar:

```bash
bundle exec rails credentials:edit
```

> Veremos que al abrir ya inicializa justamente la variable `secret_key_base`

El uso de credenciales es conveniente, sólo debe tenerse en cuenta que al
usarlo, se está generando un archivo de **vital importancia**: `config/master.key`.
Este archivo no se versiona, pero es **necesario para el funcionamiento**, dado
que se usa para descifrar las credenciales editadas con el comando anterior.
Podemos probar:

```bash
bundle exec rails credentials:show
cat config/credentials.yml.enc
```

> Veremos que el comando rails muestra en texto claro el contendio cifrado que
> vemos con `cat`.

La diferencia entre una forma de hacerlo u otra está en que cualquier comando
que se corra en el ambiente de producción necesita este valor. Entonces, a veces
es más cómodo usar un archivo que una variable. En **ambientes de contenedores,
siempre la variable es conveniente**.

### Probamos la aplicación

Ya iniciada ahora nuestra aplicación, probamos acceder: `http//IP-VM:3000`.

> Vemos que no se necesita la opción `-b` porque en el ambiente productivo, puma
> sirve contenido en todas las IPs, no sólo en 127.0.0.1.

Veremos un error, porque Rails espera que los assets estén compilados cuando
corre el ambiente de producción. Los compilamos:

```bash
RAILS_ENV=production bundle exec rails assets:precompile
```

Al acceder a `http//IP-VM:3000`, nuevamente veremos un error, pero esta vez la
página mostrará un mensaje de error 500 que poco nos dice. Sin embargo, los logs
de puma/rails dirán:

```
ActionView::Template::Error (Could not find table 'posts'):
```

El problema es que, para producción se usa otra base de datos. La creamos:

```bash
RAILS_ENV=production bundle exec rails db:create db:migrate
```

Probamos nuevamente, y....

### Configurando a rails para producción

En pos de mejorar la eficiencia de nuesto sitio, no querremos que puma, un
application server desarrollado en ruby, sirva contenido estático. Para ello,
mucho mejor usar un webserver como Caddy, nginx o apache. Procedemos entonces a
modificar la configuración de `config/environments/production.rb`:

```ruby
 config.public_file_server.enabled = false
```

Descomentar la línea mencionada, de forma de no servir archivos en la carpeta
`public/`. Reiniciamos puma y volvemos a probar.

> Recordá que tenés que cargar la página evitando que la caché muestre
> resultados. Podés ver errores en los logs de puma porque el navegador solicita
> estilos y assets que no sirve. Si el navegador aun no muestra cambios, podés
> usar: `Ctrl+F5` o abrir en modo incógnito.

Probamos con Caddy:

```bash
cd /vagrant/ruby
caddy run
```

Probar ahora en: `http//IP-VM:3000`.

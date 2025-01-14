# Application y web servers

Este repositorio dejamos ejemplos de cómo trabajar con algunos frameworks, sus
application servers y web servers.

La idea es poder comprender algunos conceptos necesarios que distan del trabajo
en desarrollo para ponerlos en producción.

## Trabajando con vagrant

A modo de simplificar la forma de armar un ambiente proveemos de un
`Vagrantfile` que permite rápidamente ir a configurar y comprender los conceptos
sin adentrarse en cómo instalar los requerimientos. Para esto, necesitás tener
instalado [vagrant](https://www.vagrantup.com/) y [VirtualBox](https://www.virtualbox.org/).

Luego simplemente corrés:

```bash
vagrant up
```

Y la vm se creará y aprovisionará al usuario con una serie de comandos
disponibles únicamente al usuario **vagrant** porque hemos usado [asdf](https://asdf-vm.com/)
para su instalación. Así es como los comandos siguientes ya están disponibles:

* php
* ruby
* python
* java
* caddy

Los servicios de apache y nginx están ambos instalados, pero no habilitados,
para que puedas probar sin conflictos porque ambos comparten el puerto 80 y 443.

## Ejemplos

* [`php/`](./php): contiene un ejemplo de cómo configurar una aplicación symfony
  con el appserver de php y algún web server.

---
typora-copy-images-to: ../../assets/img/primeros-pasos/
typora-root-url: ../../
title: Primeros pasos
layout: post
categories: parte1
conToc: true
permalink: primeros-pasos
---

## 1.0 ¿Qué aprenderemos?

* A crear un proyecto
* A definir rutas
* A usar plantillas
* A incluir plantillas dentro de otras
* A conocer el `path` de las rutas en twig
* A conocer la ruta actual en twig
* A usar el **operador ternario** y el operador `or` en twig

## 1.1 Crear un proyecto

```
composer create-project symfony/website-skeleton symfony-blog
```

1. Descargar la [plantilla de la web](/symfony-blog-teoria/assets/photo-master.zip)
2. Se descomprime en `public` 
3. Comprueba que la web funciona

## 1.2 Definir las rutas

Vamos a crear las rutas para cada una de las páginas.

El primer paso es crear un **Controlador**

```
php bin/console make:controller PageController
```

Este controlador va a encargarse de la página de `portada` y de la página `about`. 

### 1.2.1 Ruta `/`

El código que se genera automáticamente es el siguiente:

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class PageController extends AbstractController
{
    #[Route('/page', name: 'app_page')]
    public function index(): Response
    {
        return $this->render('page/index.html.twig', [
            'controller_name' => 'PageController',
        ]);
    }
}

```

Ahora eliminamos la página que genera automáticamente en `templates/page/index.html.twig`, movemos la página `public/index.html` a `templates/page/index.html.twig` y modificamos el controlador para que quede así:

```php
#[Route('/', name: 'index')]
public function index(): Response
{
	return $this->render('page/index.html.twig', []);
}
```

Hemos creado la ruta `/` y la hemos nombrado como `index`

Ya podemos probar la página de portada [http://127.0.0.1:8080/](http://127.0.0.1:8080/)

![](/symfony-blog-teoria/assets/img/primeros-pasos/image-20220316084436890.png)

#### 1.2.1.1 Plantilla base

Todas las páginas comparten la cabecera y el pie de página, así que vamos a crear la plantilla `base.html.twig`.

1. Copia el contenido de `index.html.twig` dentro de `base.html.twig`

2. Ahora `index.html` va a extender la plantilla `base`
{% raw %}
   ```twig
   {% extends 'base.html.twig' %}
   ```
{% endraw %}
3. Comprueba que sigue funcionado la portada

4. En la plantilla `base.html.twig` vamos a crear dos bloques:

   1. Uno para el título que va a variar en cada página:
      Localiza la etiqueta `title` y modifícala así:
   {% raw %}
      
      ```twig
      <title>{% block title %}Inicio{% endblock %}</title>
      ```
      {% endraw %}
   2. Otro para el cuerpo de la página:
   Localiza el código entre
   
      ```html
      <div id="index">
      ```
   y 
      ```html
      </div><!-- End of index box -->
      ```
   Este es el código que va a variar entre página y página. Por tanto vamos a moverlo a la página `index.html.twig` dentro del bloque `body`
{% raw %}
   
      ```twig
      {% block body %}
      //Código cortado de base.html.twig
      {% endblock %}
      ```
{% endraw %}

5. Comprueba que la página sigue funcionado

### 1.2.2 Ruta `/about` 

Creamos el controlador 

```php
#[Route('/about', name: 'about')]
public function about(): Response
{
    return $this->render('page/about.html.twig', []);
}
```

Movemos la página `about.html ` a `/templates/page/about.html.twig`
{% raw %}

```twig
{% extends 'base.html.twig' %}
{% block title%}About{% endblock %}
{% block body%}
<!-- Principal Content Start-->
//Código html de about
<!-- End of principal content -->
{% endblock %}
```

{% endraw %}
Comprueba que funciona la ruta `/about`

### 1.2.3 Reto - Rutas `/contact`, `/blog` y `/single_post`

> -reto- Crea las rutas `/contact` en `PageController`, y las rutas  `/blog` y `/single_post` en `BlogController`
>
> Ten en cuenta que en `blog.html` existen enlaces a `single_post.html` que deberás cambiar para llamar a la ruta `single_post`

## 1.3 Menú de navegación

Como habéis podido comprobar, el menú de navegación ha dejado de funcionar ya que apunta a las páginas `.html`

![image-20220316092852146](/symfony-blog-teoria/assets/img/primeros-pasos/image-20220316092852146.png)

Vamos a crear una `partial` para la navegación. 

1. Identificamos el código de `base.html.twig` que pinta la navegación y lo movemos a `templates/partials/navigation.html.twig`

2. Modificamos `base.html.twig` para que incluya este partial
{% raw %}
   
   ```twig
   {{ include ('partials/navigation.html.twig')}}
   ```
{% endraw %}
3. Ya podemos modificar el partial para que apunte a las nuevas rutas. Hemos de usar la función de twig `path` que nos devuelve el path a una ruta con nombre. Nunca `harcodéis` las rutas que para eso se les asigna un nombre en el controlador. Ten en cuenta que la dirección de la ruta podría cambiar durante el desarrollo del proyecto pero el nombre no debería hacerlo.
{% raw %}
   
   ```twig
   <ul class="nav navbar-nav">
       <li class="active lien"><a href="{{ path('index') }}">
       <i class="fa fa-home sr-icons"></i> Home</a>
       </li>
       <li class=" lien"><a href="{{ path('about') }}">
       <i class="fa fa-bookmark sr-icons"></i> About</a>
       </li>
    <li class=" lien"><a href="{{ path('blog') }}">
    <i class="fa fa-file-text sr-icons"></i> Blog</a>
       </li>
       <li><a href="{{ path('contact') }}">
       <i class="fa fa-phone-square sr-icons"></i> Contact</a>
       </li>
   </ul>
   ```
   {% endraw %}
   
   Pero ahora lo que ocurre es que siempre está seleccionada la opción de menú `home` porque es la que tiene la clase css `active`. `


Vamos a arreglarlo. Para ello hemos de conocer cuál es la ruta activa y añadir la clase `active` a dicha ruta. Existe la función `twig` `app.request.attributes.get('_route')` que nos devuelve dicha ruta. 

Por ejemplo, en el caso del enlace a `index` quedará así:
{% raw %}

   ```twig
<li class="{{ (app.request.attributes.get('_route') == 'index')  ? 'active': ''}} lien">
<a href="{{ path('index') }}"><i class="fa fa-home sr-icons"></i> Home</a></li>
   ```

{% endraw %} 

Estamos usando el **operador ternario** 

{% raw %}

```twig
{{ (app.request.attributes.get('_route') == 'index')  ? 'active': ''}}
```

{% endraw %}

### 1.3.1 Reto 

>-reto-Modifica la navegación para cada una de las rutas. Ten en cuenta que el menú `blog` se debe activar para las rutas `blog` y `single_post`
>
>> **Nota** En twig se puede utilizar el operador `or`

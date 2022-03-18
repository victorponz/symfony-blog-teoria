---

typora-copy-images-to: ../assets/img/admin/
typora-root-url: ../../
layout: post
categories: parte4
conToc: true
permalink: zona-admin
title: Zona de administración
---

## 4 Qué aprenderemos

* A dar permisos de administrador a un usuario
* A bloquear el acceso a determinadas páginas de nuestra web
* A usar la función `asset` de twig para hacer nuestros assets accesibles independientemente de la url de nuestra aplicación.

## 4.1 Usuario administrador

En el punto 3, vimos cómo gestionar usuarios. Hemos dado de alta algunos usuarios pero ninguno tiene el rol `administrador`. Vamos a dar este rol al usuario con `id` **1**. 

En pypMyAdmin ejecuta el siguiente sql:

```sql
UPDATE user SET roles = '["ROLE_ADMIN"]' WHERE id = 1;
```

Ahora comprueba que, efectivamente, posees ese rol. Para ello, logéate con ese usuario y comprueba la barra del *profiler*  de Symfony:

![image-20220318091014699](/symfony-blog-teoria/assets/img/admin/image-20220318091014699.png)

## 4.2 Creación de la parte admin

Vamos a crear una ruta para poder añadir imágenes a la página de portada. 

Creamos un nuevo controlador `AdminController` y la ruta `/admin/images`

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class AdminController extends AbstractController
{
    /**
     * @Route("/admin/images", name="app_images")
     */
    public function images(): Response
    {
        return $this->render('admin/images.html.twig', []);
    }
}
```

La plantilla la renombramos a `images.html.twig` y modificamos:

{% raw %}

```twig
{% extends 'base.html.twig' %}
{% block title%}Images{% endblock %}
{% block body%}
<!-- Principal Content Start -->
   <div id="images">
   	  <div class="container">
   	    <div class="col-xs-12 col-sm-8 col-sm-push-2">
       	   <h1>Images</h1>
       	   <hr>
       	</div>   
   	  </div>
   </div>
<!-- Principal Content Start -->
{% endblock %}
```

{% endraw %}

 y comprobamos que ha perdido los estilos:

![image-20220318092357644](/symfony-blog-teoria/assets/img/admin/image-20220318092357644.png)

Esto ocurre porque la plantilla `base.html.twig` tiene las rutas a los assets de forma relativa:

```html
<link rel="stylesheet" type="text/css" href="bootstrap/css/bootstrap.min.css">
```

Y cuando estamos en una ruta interna como `/admin/images/` evidentemente no encuentra los estilos porque los busca en `/admin/images/bootstrap/css/bootstrap.min.css`. Solucionarlo es tan sencillo como hacer las rutas absolutas o, mejor aún, utilizar la función `asset` de twig:

{% raw %}

```twig
{{ asset('bootstrap/css/bootstrap.min.css') }}
```

{% endraw %}

¿Por qué usamos `asset` en lugar de poner una ruta absoluta? Porque tal vez en producción sirvamos la aplicación en la ruta `blog` y entonces ya no nos funcionaría esta estrategia. Sin embargo, al usar `asset`, en producción se transformaría en `/blog/bootstrap/css/bootstrap.min.css`

Ahora cambia todas las rutas a los archivos css y javascript para incluir la función `asset`

![image-20220318093746525](/symfony-blog-teoria/assets/img/admin/image-20220318093746525.png)

## 4.3 Bloquear el acceso a la parte de administración

Bloquear el acceso a toda una zona de nuestra web es tan sencillo como modificar el archivo `config/packages/security.yml` 

```yaml
    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN }
```

Si lo que necesitamos es proteger el acceso a un controlador, usaríamos el siguiente código:

```php
public function adminDashboard(): Response
{
    $this->denyAccessUnlessGranted('ROLE_ADMIN');

    // or add an optional message - seen by developers
    $this->denyAccessUnlessGranted('ROLE_ADMIN', null, 'User tried to access a page without having ROLE_ADMIN');
}
```

Si el usuario no está logeado se redirige automáticamente a la página de login. Pero si sí lo está pero no tiene el rol `ADMIN` symfony muestra una página de error 

![image-20220318095713134](/symfony-blog-teoria/assets/img/admin/image-20220318095713134.png)

> **Nota**
>
> En producción muestra esta página:
>
> ![image-20220318100547163](/symfony-blog-teoria/assets/img/admin/image-20220318100547163.png)
> Para comprobarlo, modifica en archivo `.env` y fija el entorno a producción
>
> ```
> APP_ENV=prod
> ```



## 4.4 Gestión de imágenes

En este apartado vamos a implementar la subida e imágenes para la página de portada.

### 4.4.1 Entidad `Category`

Las imágenes pertenecen a una categoría (en la página de portada existen tres pestañas de categorías). Así que el primer paso será crear la entidad `Category`

<script id="asciicast-gHZCcLy8ie8gqxORzqutXKJ3s" src="https://asciinema.org/a/gHZCcLy8ie8gqxORzqutXKJ3s.js" async></script>

Y realizamos la migración:

```
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

Y ahora creamos el formulario

```
php bin/console make:form CategoryForm Category
```

En el controlador `AdminController` creamos la siguiente ruta

```php
/**
 * @Route("/admin/categories", name="app_categories")
 */
public function categories(): Response
{
    return $this->render('admin/categories.html.twig', []);
}
```

{% raw %}

```twig
{% extends 'base.html.twig' %}
{% block title%}Categories{% endblock %}
{% block body%}
<!-- Principal Content Start -->
   <div id="categories">
   	  <div class="container">
   	    <div class="col-xs-12 col-sm-8 col-sm-push-2">
       	   <h1>Categories</h1>
       	   <hr class='divider'>
       	</div>   
   	  </div>
   </div>
<!-- Principal Content Start -->
{% endblock %}
```

{% endraw %}

Ya podemos visitar la ruta `/admin/categories`

### 4.4.2 Formulario Category` 

Creamos el formulario para Categorías tal y como hicimos en el Formulario de Contacto

![image-20220318113520367](/symfony-blog-teoria/assets/img/admin/image-20220318113520367.png)
**Controlador**
```php
/**
 * @Route("/admin/categories", name="app_categories")
 */
public function categories(ManagerRegistry $doctrine, Request $request): Response
{
    $category = new Category();
    $form = $this->createForm(CategoryFormType::class, $category);
    $form->handleRequest($request);
    if ($form->isSubmitted() && $form->isValid()) {
        $category = $form->getData();    
        $entityManager = $doctrine->getManager();    
        $entityManager->persist($category);
        $entityManager->flush();
    }
    return $this->render('admin/categories.html.twig', array(
        'form' => $form->createView() 
    ));

}
```
**Plantilla**

{{% raw %}}

```twig

   <div id="categories">
   	  <div class="container">
   	    <div class="col-xs-12 col-sm-8 col-sm-push-2">
       	   <h1>Categories</h1>
       	   {{ form(form, {'attr': {'class':'form-horizontal'}}) }}
       	   <hr class='divider'>
       	</div>   
   	  </div>
   </div>
```

{{% endraw %}}

También vamos a mostrar una lista con todas las categorías:

![image-20220318114421816](/symfony-blog-teoria/assets/img/admin/image-20220318114421816.png)

Modificamos el controlador para recuperar todas las categorías de la base de datos:

```php
/**
 * @Route("/admin/categories", name="app_categories")
 */
public function categories(ManagerRegistry $doctrine, Request $request): Response
{
    //Filtramos aquellos que contengan dicho texto en el nombre
    $repositorio = $doctrine->getRepository(Category::class);

    $categories = $repositorio->findAll();

    $category = new Category();
    $form = $this->createForm(CategoryFormType::class, $category);
    $form->handleRequest($request);
    if ($form->isSubmitted() && $form->isValid()) {
        $category = $form->getData();    
        $entityManager = $doctrine->getManager();    
        $entityManager->persist($category);
        $entityManager->flush();
    }
    return $this->render('admin/categories.html.twig', array(
        'form' => $form->createView(),
        'categories' => $categories   
    ));

}
```

Y modificamos la plantilla para recorrer las categorías:

{% raw %}

```twig
<hr class="divider">
<div>
	<table class="table">
		<thead>
		<tr>
			<th scope="col">#</th>
			<th scope="col">Name</th>
		</tr>
		</thead>
		<tbody>
		{% for category in categories %}
		<tr>
			<th scope="row">{{ category.id }}</th>
			<td>
				{{ category.name }}
			</td>
		</tr>
		{% endfor %}
		</tbody>
	</table>
</div>
```

{% endraw %}

### 4.4.3 Entidad `Image`

Como antes vamos a crear la entidad `Image` que tiene una clave ajena a `Category`. Por ello, cuando definamos la entidad hemos de indicar que el campo `Category` es una `Relation` de tipo `ManytoOne`

<script id="asciicast-F3uLObe7zHp6tq9WvvUR89n1m" src="https://asciinema.org/a/F3uLObe7zHp6tq9WvvUR89n1m.js" async></script>

Al igual que hicimos con el formulario de contacto, hemos de crear el formulario y la plantilla:





 



**Más información en**

[https://symfony.com/doc/current/security.html](https://symfony.com/doc/current/security.html)

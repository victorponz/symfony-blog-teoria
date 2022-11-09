---

typora-copy-images-to: ../assets/img/admin/
typora-root-url: ../../
layout: post
categories: parte4
conToc: true
permalink: zona-admin
title: Zona de administración
render_with_liquid: false
---

## 4 Qué aprenderemos

* A dar permisos de administrador a un usuario
* A bloquear el acceso a determinadas páginas de nuestra web
* A usar la función `asset` de twig para hacer nuestros assets accesibles independientemente de la url de nuestra aplicación.
* A usar métodos del repositorio
* A crear relaciones entre entidades
* A subir imágenes al servidor
* A copiar imágenes

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
    #[Route('//admin/images', name: 'app_images')]
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
visitamos [http://127.0.0.1:8080/admin/images](http://127.0.0.1:8080/admin/images) y comprobamos que ha perdido los estilos:

![image-20220318092357644](/symfony-blog-teoria/assets/img/admin/image-20220318092357644.png)

Esto ocurre porque la plantilla `base.html.twig` tiene las rutas a los assets (los assets son todos los archivos estáticos de la aplicación, como imágenes, hojas de estilos y scripts) de forma relativa:

```html
<link rel="stylesheet" type="text/css" href="bootstrap/css/bootstrap.min.css">
```

Y cuando estamos en una ruta interna como `/admin/images/` evidentemente no encuentra los estilos porque los busca en `/admin/images/bootstrap/css/bootstrap.min.css`. Solucionarlo es tan sencillo como hacer las rutas absolutas o, mejor aún, utilizar la función `asset` de twig:
{% raw %}

```twig
<link rel="stylesheet" type="text/css" href="{{ asset('bootstrap/css/bootstrap.min.css') }}">
```
{% endraw %}

> -info-¿Por qué usamos `asset` en lugar de poner una ruta absoluta? Porque tal vez en producción sirvamos la aplicación en la ruta `blog` y entonces ya no nos funcionaría esta estrategia. Sin embargo, al usar `asset`, en producción se transformaría en `/blog/bootstrap/css/bootstrap.min.css`

Ahora cambia todas las rutas a los archivos css y javascript para incluir la función `asset`

![image-20220318093746525](/symfony-blog-teoria/assets/img/admin/image-20220318093746525.png)

## 4.3 Bloquear el acceso a la parte de administración

Bloquear el acceso a toda una zona de nuestra web es tan sencillo como modificar el archivo `config/packages/security.yml` 

```yaml
    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN }
```

> -alert-Como todos los archivos `yaml` es muy sensible a errores. Sólo se pueden poner 4 espacios para indentar.

Si lo que necesitamos es proteger el acceso a un controlador, usaríamos el siguiente código:

```php
public function adminDashboard(): Response
{
    $this->denyAccessUnlessGranted('ROLE_ADMIN');

    // or add an optional message - seen by developers
    $this->denyAccessUnlessGranted('ROLE_ADMIN', null, 'User tried to access a page without having ROLE_ADMIN');
    
    new Response("Sí que puedes entrar");
}
```

Si el usuario no está logeado se redirige automáticamente a la página de login. Pero si sí lo está pero no tiene el rol `ADMIN` Symfony muestra una página de error 

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

En este apartado vamos a implementar la subida de imágenes para la página de portada.

### 4.4.1 Entidad `Category`

Las imágenes pertenecen a una categoría (en la página de portada existen tres pestañas de categorías). Así que el primer paso será crear la entidad `Category`

<script id="asciicast-2T3YG2vSCrrSDbGH5F1sItQgR" src="https://asciinema.org/a/2T3YG2vSCrrSDbGH5F1sItQgR.js" async></script>

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

Y la plantilla:
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

### 4.4.2 Formulario `Category`

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

![image-20220322203344623](/symfony-blog-teoria/assets/img/admin/image-20220322203344623.png)

También vamos a mostrar una lista con todas las categorías:

![image-20220318114421816](/symfony-blog-teoria/assets/img/admin/image-20220318114421816.png)

Modificamos el controlador para recuperar todas las categorías de la base de datos:

```php
/**
 * @Route("/admin/categories", name="app_categories")
 */
public function categories(ManagerRegistry $doctrine, Request $request): Response
{
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

Como antes, vamos a crear la entidad `Image` que tiene una clave ajena a `Category`. Por ello, cuando definamos la entidad hemos de indicar que el campo `Category` es una `Relation` de tipo `ManyToOne`

<script id="asciicast-DCh9Ui5o5MdQpaZ20WQO2wyOj" src="https://asciinema.org/a/DCh9Ui5o5MdQpaZ20WQO2wyOj.js" async></script>

Y realizamos la migración:

```
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

Al igual que hicimos con el formulario de contacto, hemos de crear el formulario y la plantilla:

En este caso hay dos peculiaridades:

* La imagen tiene una clave ajena con la entidad `Category`
* Queremos subir una imagen y almacenar la ruta a la misma en la base de datos 

En primer lugar creamos el formulario:

```
php bin/console make:form ImageForm Image
```

En este formulario hacemos que el campo `category` obtenga los datos de la entidad `Category`

```php
...
use Symfony\Bridge\Doctrine\Form\Type\EntityType;
...
public function buildForm(FormBuilderInterface $builder, array $options): void
{
    $builder
        ->add('file')
        ->add('numLikes')
        ->add('numViews')
        ->add('numDownloads')
        ->add('ategory', EntityType::class, array(
            'class' => Category::class,
            'choice_label' => 'name'))
    ;
}
```

Y la plantilla:


{% raw %}
```twig
<div id="images">
    <div class="container">
        <div class="col-xs-12 col-sm-8 col-sm-push-2">
            <h1>Images</h1>
            {{ form(form, {'attr': {'class':'form-horizontal'}}) }}
            <hr class="divider">
        </div>   
    </div>
</div>
```
{% endraw %}

Creamos el controlador:
```php
/**
 * @Route("/admin/images", name="app_images")
 */
public function images(ManagerRegistry $doctrine, Request $request, SluggerInterface $slugger): Response
{
    
    $repository = $doctrine->getRepository(Image::class);

    $images = $repository->findAll();

    $image = new Image();
    $form = $this->createForm(ImageFormType::class, $image);
    $form->handleRequest($request);
    if ($form->isSubmitted() && $form->isValid()) {
        $image = $form->getData();    
        $entityManager = $doctrine->getManager();    
        $entityManager->persist($image);
        $entityManager->flush();
    }
    return $this->render('admin/images.html.twig', array(
        'form' => $form->createView(),
        'images' => $images   
    ));

    
}
```
Vamos a probar que funciona: 

![image-20220322174001982](/symfony-blog-teoria/assets/img/admin/image-20220322174001982.png)

Ahora vamos a añadir clases a los campos para que tenga el mismo aspecto visual que el resto de la aplicación, pero esta vez lo haremos en el propio formulario:

```php
public function buildForm(FormBuilderInterface $builder, array $options): void
{
    $builder
        ->add('File')
        ->add('NumLikes', null, ['attr' => ['class'=>'form-control']])
        ->add('NumViews', null, ['attr' => ['class'=>'form-control']])
        ->add('NumDownloads', null, ['attr' => ['class'=>'form-control']])
        ->add('category', EntityType::class, array(
            'class' => Category::class,
            'choice_label' => 'name'))
        ->add('Send', SubmitType::class, ['attr' => ['class'=>'pull-right btn btn-lg sr-button']]);
    ;
}
```

### 4.4.4 Subir imágenes al servidor

El truco para que el campo `file` sea de tipo `input file` es decirle a Symfony que no esté mapeado de tal forma que no lo guarde automáticamente:

```php
...
use Symfony\Component\Form\Extension\Core\Type\FileType;
use Symfony\Component\Validator\Constraints\File;
...
$builder
    ->add('File', FileType::class,[
        'mapped' => false,
        'constraints' => [
            new File([
                'mimeTypes' => [
                    'image/jpeg',
                    'image/png',
                ],
                'mimeTypesMessage' => 'Please upload a valid image file',
            ])
        ],
    ])
```

Ahora ya sólo nos queda modificar el controlador:

```php
...
use Symfony\Component\Filesystem\Filesystem;
use Symfony\Component\HttpFoundation\File\Exception\FileException;
use Symfony\Component\HttpFoundation\File\UploadedFile;
use Symfony\Component\String\Slugger\SluggerInterface;
...

...
if ($form->isSubmitted() && $form->isValid()) {
    $file = $form->get('File')->getData();
    if ($file) {
        $originalFilename = pathinfo($file->getClientOriginalName(), PATHINFO_FILENAME);
        // this is needed to safely include the file name as part of the URL
        $safeFilename = $slugger->slug($originalFilename);
        $newFilename = $safeFilename.'-'.uniqid().'.'.$file->guessExtension();

        // Move the file to the directory where images are stored
        try {

            $file->move(
                $this->getParameter('images_directory'), $newFilename
            );
            $filesystem = new Filesystem();
            $filesystem->copy(
                $this->getParameter('images_directory') . '/'. $newFilename, 
                $this->getParameter('portfolio_directory') . '/'.  $newFilename, true);

        } catch (FileException $e) {
            // ... handle exception if something happens during file upload
        }

        // updates the 'file$filename' property to store the PDF file name
        // instead of its contents
        $image->setFile($newFilename);
    }
    $image = $form->getData();   
    
...
```

Y definir la ruta a las imágenes en `config/services.yml`


```yaml
parameters:
    images_directory: '%kernel.project_dir%/public/images/index/gallery'
    portfolio_directory: '%kernel.project_dir%/public/images/index/portfolio'
```

### 4.4.5 RETO

Crea la lista con las imágenes:

Para obtener el nombre de la imagen usa: 

{% raw %}
`{{ asset('images/index/gallery/' ~ image.file) }}`
{% endraw %}


![image-20220322195732035](/symfony-blog-teoria/assets/img/admin/image-20220322195732035.png)

## 4.5 Página de portada.

Ahora ya podemos modificar la página de portada para mostrar las imágenes que se van subiendo.

Antes de nada, carga este archivo [sql](assets/images.sql) que contiene las instrucciones para insertar las imágenes ya existentes. 

Ahora modificamos el controlador para obtener todas las categorías:

```php
/**
 * @Route("/", name="index")
 */
public function index(ManagerRegistry $doctrine, Request $request): Response
{
    $repository = $doctrine->getRepository(Category::class);

    $categories = $repository->findAll();

    return $this->render('page/index.html.twig', ['categories' => $categories]);
}
```

Y modificar la plantilla `index.html.twig`. Debes eliminar todo el HTML que pinta las pestañas de las categorías y sustituirlo por la siguiente plantilla twig.

{% raw %}
```twig
<div class="table-responsive">
  <table class="table text-center">
    <thead>
      <tr>
      {% for category in categories %}
        <td><a class="link {{loop.first ? 'active' : ''}}" href="#category{{category.id}}" data-toggle="tab">{{category.name}}</a></td>
      {% endfor %}
      </tr>
    </thead>
  </table>
  <hr>
</div>
```
{% endraw %}

En esta plantilla usamos `loop.first` para poner la clase `active`  a la primera categoría.

![image-20220322185956882](/symfony-blog-teoria/assets/img/admin/image-20220322185956882.png)

Eliminamos el siguiente HTML porque de momento no vamos a paginar las imágenes:

```html
<nav class="text-center">
    <ul class="pagination">
        <li class="active"><a href="#">1</a></li>
        <li><a href="#">2</a></li>
        <li><a href="#">3</a></li>
        <li><a href="#" aria-label="suivant">
        <span aria-hidden="true">&raquo;</span>
        </a></li>
    </ul>
</nav>
```
Y ahora vamos a recorrer todas las imágenes.

Primero eliminamos todo el código repetido y dejamos sólo el mínimo:

```html
<div class="tab-content">

<!-- First Category pictures -->
    <div id="category1" class="tab-pane active" >
      <div class="row popup-gallery">
        <div class="col-xs-12 col-sm-6 col-md-3">
        <div class="sol">
          <img class="img-responsive" src="images/index/portfolio/1.jpg" alt="First category picture">
          <div class="behind">
              <div class="head text-center">
                <ul class="list-inline">
                  <li>
                    <a class="gallery" href="images/index/gallery/1.jpg" data-toggle="tooltip" data-original-title="Quick View">
                      <i class="fa fa-eye"></i>
                    </a>
                  </li>
                  <li>
                    <a href="#" data-toggle="tooltip" data-original-title="Click if you like it">
                      <i class="fa fa-heart"></i>
                    </a>
                  </li>
                  <li>
                    <a href="#" data-toggle="tooltip" data-original-title="Download">
                      <i class="fa fa-download"></i>
                    </a>
                  </li>
                  <li>
                    <a href="#" data-toggle="tooltip" data-original-title="More information">
                      <i class="fa fa-info"></i>
                    </a>
                  </li>
                </ul>
              </div>
              <div class="row box-content">
                <ul class="list-inline text-center">
                  <li><i class="fa fa-eye"></i> 1000</li>
                  <li><i class="fa fa-heart"></i> 500</li>
                  <li><i class="fa fa-download"></i> 100</li>
                </ul>
              </div>
          </div>
        </div>
        </div> 

      </div>
    </div>
<!-- End of First category pictures -->

</div>
```

Y ahora creamos un partial para la imagen:

{% raw %}
```twig
<div class="col-xs-12 col-sm-6 col-md-3">
<div class="sol">
  <img class="img-responsive" src="{{ asset('images/index/portfolio/' ~ image.file) }}" alt="First category picture">
  <div class="behind">
      <div class="head text-center">
        <ul class="list-inline">
          <li>
            <a class="gallery" href="{{ asset('images/index/gallery/' ~ image.file) }}" data-toggle="tooltip" data-original-title="Quick View">
              <i class="fa fa-eye"></i>
            </a>
          </li>
          <li>
            <a href="#" data-toggle="tooltip" data-original-title="Click if you like it">
              <i class="fa fa-heart"></i>
            </a>
          </li>
          <li>
            <a href="#" data-toggle="tooltip" data-original-title="Download">
              <i class="fa fa-download"></i>
            </a>
          </li>
          <li>
            <a href="#" data-toggle="tooltip" data-original-title="More information">
              <i class="fa fa-info"></i>
            </a>
          </li>
        </ul>
      </div>
      <div class="row box-content">
        <ul class="list-inline text-center">
          <li><i class="fa fa-eye"></i> {{ image.numViews }}</li>
          <li><i class="fa fa-heart"></i> {{ image.numLikes }}</li>
          <li><i class="fa fa-download"></i> {{ image.numDownloads }}</li>
        </ul>
      </div>
  </div>
</div>
</div> 
```
{% endraw %}

Y modificamos `index.html.twig` 

{% raw %}
```twig
<div class="tab-content">
  {% for category in categories %}
    <div id="category{{category.id}}" class="tab-pane {{loop.first ? 'active' : ''}}" >
      <div class="row popup-gallery">
      {% for image in category.images %}
      {{ include ('partials/image.html.twig')}}
      {% endfor %}
        </div>
    </div>
{% endfor %}
```
{% endraw %}
**Más información en**

[https://symfony.com/doc/current/security.html](https://symfony.com/doc/current/security.html)

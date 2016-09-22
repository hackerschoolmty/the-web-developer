# Introducción a Ruby on Rails

### Cómandos básicos en 15 minutos

```bash
$ gem install rails -v 4.2.4
$ rails -v # 4.2.4
```

Preguntas breves
1. En qué otros lenguajes programan?
2. Cuáles otros frameworks usan?
3. Cuánto tardaron en aprender el framework?
4. Cuánto tardan actualmente en hacer un CRUD?

```bash
$ rails
$ rails new imdb_regio
$ cd imdb_regio
$ tree . -L 1
$ tree . L 2
$ tree app/
$ rails s # then go to http://localhost:3000/ in chrome
$ rails generate controller pages home # then go to http://localhost:3000/pages/home
$ rails generate model category name:string
$ cat db/migrate/20160830054821_create_categories.rb
$ subl db/migrate/20160830054821_create_categories.rb
$ rake db:migrate
$ rails c
irb> Category.connection
irb> Category.count
irb> Category.create name:"Horror"
irb> Category.create name:"Comedy"
irb> Category.create name:"Action"
irb> Category.all
irb> Category.first
irb> horror= "Horror"
irb> Category.where(name:horror)
irb> Category.find_by_name(horror)
irb> exit

$ rails generate scaffold movies title:string duration:float category:references
$ rake db:migrate # then go to http://localhost:3000/movies and create movies
$ echo "gem 'activeadmin', github: 'activeadmin'" >> Gemfile
$ bundle install
$ rails generate active_admin:install --skip-users
$ rake db:migrate
$ rails generate active_admin:resource Movie
```

## Comandos Básicos

```bash
$ rails new first_project
$ cd first_project
$ bundle
$ rails server
$ rails generate model animal name:string
$ rake db:migrate
$ rake db:migrate:status
$ rake db:rollback
$ rails destroy model animal
$ rails console
```

# Conceptos Básicos

## Estructura del Proyecto

```bash
$ cd ..
$ rails new zoo
$ cd zoo
```
## Modelos

Los modelos son clases que heredan de ActiveRecord::Base y representan una tabla
en la base de datos. El nombre de la tabla es el nombre
del modelo pero en plural. Por ejemplo la tabla del modelo Animal es animals.
En caso de un modelo con dos o más palabras el nombre de la tabla sigue el
convenio snake_case: MammalAnimal = mammal_animals.

```bash
$ rails generate model animal name:string
$ rake db:migrate
$ rails generate model category name:string
$ cat [migration_path]
$ rails generate migration AddCategoryIdToAnimal category_id:integer:index
$ rake db:migrate:status
$ cat [migrationpath]
$ rake db:migrate
```

Ahora agregamos al archivo ```db/seeds.rb```

```ruby
Category.destroy_all
Animal.destroy_all

mamifero = Category.create name: "Mamifero"
ave      = Category.create( name: "Ave" )
reptil   = Category.new name: "Reptil"
reptil.save
esponja  = Category.new( name: "Esponja" ).save

lobo_wallstreet = Animal.create(name:"Lobo de Wallstreet", category_id: mamifero.id)
pajaro_loco     = Animal.create name: "Pajaro Loco", category_id: ave.id
piolin          = Animal.new name: "Piolin"
piolin.category_id = ave.id
piolin.save
```

Ejecutamos la siguiente instruccion: ```$ rake db:seed```

Entramos a la consola de rails

```ruby
$ rails console
irb> Category.count
irb> Animal.count
irb> Category.where(name: "Mamifero").count
irb> ave = Category.where("name = ?", "Ave").first
irb> Category.find_by_name("Reptil")
irb> woody = "Pajaro Loco"
irb> Animal.where(name: woody)
irb> Animal.find_by_name(woody)
irb> Animal.where(category_id: ave.id)
irb> Animal.where(name: "Lobo de Wallstreet").where(category_id: ave.id)
irb> Animal.where(name: "Piolin").where(category_id: ave.id)
irb> aves = Animal.where(category_id: ave.id)
irb> piolin = aves.find_by_name("Lobo de Wallstreet")
irb> piolin = aves.find_by_name("Piolin")

```
### Relaciones
```ruby
# app/models/category.rb
has_many :animals

# app/models/animal.rb
belongs_to :category
```

Una vez definidas las relaciones podemos relaizar busquedas más explicitas

```ruby
irb> mamifero = Category.where(name:"Mamifero").first
irb> mamifero.animals
irb> woody = Animal.find_by_name(woody)
irb> woody.category
```
### Scopes
Un scope permite especificar consultas que se usan comunmente en tu proyecto. Un
scope es definido como un método que regresa un objeto del tipo ActiveRecord::Relation
lo cual permitirá encadenar métodos que tambien regresen un objeto de este tipo. Tales
como: where, join, order, etc.

```ruby
# app/models/category.rb
scope :mammals, -> {where(name: "Mamifero")}
scope :birds,   -> {where(name: "Ave")}
scope :reptil,  -> {where(name: "Reptil")}

# app/models/animal.rb
scope :mammals, -> {joins(:category).where("categories.name = ?", "Mamifero")}
```

Los podemos usar mandandolos a llamar desde consola

```bash
$ rails console
irb> Category.all
irb> Category.find(1)
irb> Category.mammals
irb> Category.birds
irb> Category.reptil
irb> Category.birds.joins(:animals).where("animals.name = ?", "Piolin")
irb> Animal.mammals
```
### Actualizar
```bash
$ rails console
irb> category = Category.find(1)
irb> category.update(name: "Mammal")
irb> category.name
irb> category = Category.last
irb> category.name = "Bird"
irb> category.save
irb> category
```

## Controladores, Vistas
Instalamos curl

```bash
$ rails generate controller static home about contact
$ rake routes # then go to chrome and explore the routes
```
### Enrutamiento básico
```bash
$ rails generate controller categories index
$ rake routes
```

Podemos ir a consultar http://localhost:3000/categories
```ruby
# app/controllers/categories_controller.rb
def index
  @categories = Category.all
end
# app/views/categories/index.html.erb
<%= link_to categories_url %>
<ul>
  <% @categories.each do |category| %>
    <li><%= category.name %></li>
  <% end %>
</ul>
```

```ruby
# app/controllers/categories_controller.rb

def create

end

def update

end

def destroy

end

# config/routes.rb
resources :categories
```

Ahora vemos las rutas
```bash
$ rake routes
```
Para que nos haga sentido debemos de entender cómo funciona rest.

```ruby
resources :categories, only: [:create, :update, :delete, :index]
```
Para poder hacer post, put y delete a los recursos debemos de usar postman o curl.
En caso de los que usen curl:

```bash
$ curl -X POST http://localhost:3000/categories/
$ curl -X PUT http://localhost:3000/categories/1
$ curl -X DELETE http://localhost:3000/categories/1
```

Como nos podemos asegurar que entra? Bueno empecemos a debuggear

```ruby
# Gemfile
gem 'pry'
```

Cada que se hace un cambio en el archivo Gemfile debemos de ejecutar el Comando
bundle. Esto nos permitirá instalar las gemas que acabamos de especificar.

```bash
$ bundle
```

```ruby
def create
  binding.pry
end

def update
  binding.pry
end

def destroy
  binding.pry
end
```

#### Destroy
Empecemos con el más sencillo, destroy

```ruby
def destroy
  category = Category.find(params[:id])
  category.destroy

  respond_to do |format|
    format.html{ redirect_to categories_url }
  end
end
```

Para probar que funciona podemos ejecutar desde la terminal lo siguiente
```bash
$ curl -X DELETE http://localhost:3000/categories/1
```

#### Create
Ahora continuemos con la acción de create. Esta acción, como la convención dicta
nos va a servir para crear categories. Por lo tanto en el el controller vamos a
escribir lo siguiente.

```ruby
# app/controllers/categories_controller.rb
def create
  @article = Category.new(params[:category])
  @article.save
  redirect_to categories_url
end
```

Como los parametros que nos interesan son los de category, entonces vamos a
requerir solo esos. Por lo pronto vamos a agregar la línea
``protect_from_forgery with: :null_session`` pero esto es momentaneo y solo
para que podamos seguir probando con curl.

```bash
$ curl -X POST -d "category[name]=Insecto" http://localhost:3000/categories/
```

Lamentablemente esto nos marca error pero lo podemos solucionar. El error que marca
es debido a algunas politicas de seguridad que maneja Rails. La política de seguridad
en este caso es StrongParameters en la cuál necesitamos especificar en el controlador
cuáles parametros son permitidos al momento de crear/actualizar nuestros modelos.

```ruby
# app/controllers/categories_controller.rb
def create
  @category = Category.new(params.require[:category].permit(:name))
  @category.save
  redirect_to categories_url
end
```

#### Update

La siguiente acción es la de actualizar un registro de la base de datos. Para eso
el generador de rutas nos provee del url categories/:id. El cuál nos permite especificar
en el parametro [id] el registro que vamos a actualizar. Pero a diferencia de destory
también recibe los campos con la información que deseamos actualizar. Digamos que
es una mezcla entre destroy y create.

```ruby
# app/controllers/categories_controller.rb
def update
  @category = Category.find(params[:id])
  @category.update(params[:category])
  redirect_to categories_url
end
```

Para probar que se funciona lo que acabamos de hacer
```bash
$ curl -X PUT -d "category[name]=Mamifero" http://localhost:3000/categories/1
```

Qué no funciona? Que marca lo mismo? AVISENME!

#### Reutilizar código

Como nos pudimos haber dado cuenta, la sintaxis que utilizamos para solucionar
el error de StrongParameters se puede utilizar tanto en el create como en el update
por lo tanto vamos a ver como reutilizarlo

```ruby
# app/controllers/categories_controller.rb

def create
  @category = Category.new(category_params)
  @category.save
  redirect_to categories_url
end

def update
  @category = Category.find(params[:id])
  @category.update(category_params)
  redirect_to categories_url
end

private

def category_params
  params.require[:category].permit(:name)
end
```

## Scaffolds, desarrollo aún más rápido
Lo que hemos visto hasta ahorita no son casos aislados. Cómo pudieron darse cuenta
los modelos, controladores y las vistas que hemos generado hasta el momento son
problemas que nos enfrentamos día a día en cada desarrollo nuevo que realizamos.

Para agilizar este proceso existe la herramienta scaffold que es una format rápida
de crear modelos, vistas y controladores para un recurso en una misma operación.

``` bash
$ rails g scaffold zoo name:string location:string
```
Examinamos un poco lo que nos genera.

Podemos explorar si nos vamos a http://localhost:3000/zoos

Seguimos examinando las vistas


Tambien podemos generar un scaffold de un modelo que ya existe :D
```bash
$ rails g scaffold animal
$ rails g scaffold animal --skip db/migrate/20160919034216_create_animals.rb
```

## Ruby on Rails y REST

Ruby on rails es un Framework para hacer aplicaciones web que siguen el patron
Model View Controller y usan una arquitectura de REST.

### Que es REST?

Para entender lo que significa que una aplicación web sigue una arquitectura REST
consideremos el proyecto zoo:

1. Un usuario visita la dirección http://localhost:3000/animals/new desde su navegador web.
2. El navegador envía una petición HTTP al servidor de zoo. En este caso es Webrick
que se ejecuta con el comando ```$ rails server ```
3. El servidor responde con un documento HTML que contiene un formulario y
enlaces.
4. El usuario ingresa la info del animal y envia el formulario.
5. El navegador envía otra petición HTTP al servidor.
6. El servidor procesa la petición y responde con una redireción a otra página web.

Este ciclo continúa hasta que el usuario deja de navegar en la aplicación. A
excepción de algunos casos, la mayoria de los sitios web o aplicaciones web siguen
el mismo patron.

### Uniform Resourece Identifiers (URI)

Lo que el usuario escribe en el navegador al inicio del ciclo anterior es un URI.
Otra forma de llamarlo es un URL (Uniform Resource Locator). Un URI es un termino
generalizado para referirse tanto a una ubicación o a un nombre.

Un URI es el identificador de un recurso.

### Resources

Un recurso es cualquier cosa que puede ser identificado por un URI. En el ejemplo
anterior la URI escrita por el usuario es la dirección de un recurso que corresponde
a una página web. En un sitio web estático, por ejemplo, cada página web es un
recurso.

En el paso número cuarto, la parte del servidor que actualiza un Animal es otro
recurso. El formulario HTML que se usa para enviar el formulario tiene la
dirección (URI) de su recurso.

### Representations

El documento HTML que el servidor regresa al cliente es una representacion de un
recurso. Una representación es una encapsulación de la información (estado, dato, o
markup) del recurso, codificado usando formatos como: XML, JSON, o HTML.

Un recurso puede tener una o más representaciones. Clientes y servidores usan
'media types' para indicar el tipo de representacion para la parte que recibe
(el cliente o el servidor). La mayoria de los sitios web o aplicaciones web usan
el formato HTML con el 'media type' text/html. De forma similar cuando el usuario
envía un formulario, el navegador envía una representación usando el URI y el
'media type' application/x-www-form-urlencoded.

### Uniform Interface

Los clientes usan el 'Hypertext Transfer Protocol' (HTTP) para enviar peticiones
a los recursos y obtener las respuestas. En el primer paso, el ciente envia
una peticion GET para traer un documento HTML. En el paso cuatro el ciente envía
una petición POST para crear un nuevo Animal.

Ambos métodos (GET y POST) pertenecen a la interfaz uniforme de HTTP
(HTTP uniform interface). El uso de la interfaz uniforme es para que las peticiones
y las respuestas se describan a sí mismas y sean visibles. Ademas de los métodos
mostrados en el ejemplo existen otros métodos como OPTIONS, HEAD, PUT, DELETE,
TRACE, y CONNECT.

## Vistas
Las vistas se encuentran en la carpeta ```app/views```. Esta carpeta contiene
carpetas que están relacionadas directamente con un controlador. En el caso
de la carpeta animals se encuentra relacionada al controlador animals_contrller.
Los archivos html.erb que se encuentran dentro de cada carpeta esan relacionados
a las aciones (ó métodos) de los controladores. En el caso del archivo new.html.erb,
e index.html.erb, están relacionados a las acciones new e index, respectivamente.

### Respuestas del controllador

El contorlador solo puede crear respuestas HTTP mediante los comandos:

1. render
2. redirect_to
3. head

#### render
Cuando los controladores tienen una vista que corresponde a la acción. Automáticamente
se hace un render desde el controlador que selecciona el archivo html.erb con el
que va a responder.

Sin embargo podemos modificar ese comportamiento.

```ruby
  render 'new'
  # app/controllers/animals_contrller
  def index
end
```
##### render inline
```ruby
# app/controllers/animals_contrller
def index
  render plain: "OK"
end
```

```ruby
# app/controllers/animals_contrller
def index
  render html: "<strong>OK</strong>"
end
```

```ruby
# app/controllers/animals_contrller
def index
  @animals = Animal.all
  render json: @animals
end
```
### Layouts

```ruby
# app/controllers/pages_controller
class PagesController < ApplicationContoller
  layout 'plain'
end
```

```html
<!DOCTYPE html>
<html>
<head>
  <title>Zoo</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
<body>

<%= yield %>

</body>
</html>
```

### Helpers
```html
<!-- app/views/categories/index.html.erb -->
...
<%= categories_url %>
<%= new_category_path %>
<%= edit_category_path(14) %>
<%= edit_category_path(14) %>

<%= link_to categories_url        , "Categorias" %>
<%= link_to new_category_path     , "Nueva Categoria" %>
<%= link_to edit_category_path(14), "Editar Categoria" %>
<%= link_to category_path(14)     ,  "Ver Categoria" %>
...
```

### Vistas Parciales
Las vistas parciales son vistas que pueden ser utilizadas dentro de otras vistas.
#### Agregando Boostrap
Para dejar un ejemplo práctico de esto utilizaremos boostrap dentro de nuestro proyecto.

```html
# app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
<head>
  <title>AssetsPipeline</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
  <!-- Latest compiled and minified CSS -->
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" integrity="sha384-1q8mTJOASx8j1Au+a5WDVnPi2lkFfwwEAa8hDDdjZlpLegxhjVME1fgjWPGmkzs7" crossorigin="anonymous">

<!-- Optional theme -->
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap-theme.min.css" integrity="sha384-fLW2N01lMqjakBkx3l/M9EahuwpSfeNvV63J5ezn3uZzapT0u7EYsXMjQV+0En5r" crossorigin="anonymous">

<!-- Latest compiled and minified JavaScript -->
  <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js" integrity="sha384-0mSbJDEHialfmuBBQP6A4Qrprq5OVfW37PRR3j5ELqxss1yVqOtnepnHVP9aJ7xS" crossorigin="anonymous"></script>

</head>
<body >
  <nav class="navbar navbar-default">
    <div class="container-fluid">
      <!-- Brand and toggle get grouped for better mobile display -->
      <div class="navbar-header">
        <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
          <span class="sr-only">Toggle navigation</span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
        </button>
        <a class="navbar-brand" href="#">Brand</a>
      </div>

      <!-- Collect the nav links, forms, and other content for toggling -->
      <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
        <ul class="nav navbar-nav">
          <li class="active"><a href="#">Link <span class="sr-only">(current)</span></a></li>
          <li><a href="#">Link</a></li>
          <li class="dropdown">
            <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">Dropdown <span class="caret"></span></a>
            <ul class="dropdown-menu">
              <li><a href="#">Action</a></li>
              <li><a href="#">Another action</a></li>
              <li><a href="#">Something else here</a></li>
              <li role="separator" class="divider"></li>
              <li><a href="#">Separated link</a></li>
              <li role="separator" class="divider"></li>
              <li><a href="#">One more separated link</a></li>
            </ul>
          </li>
        </ul>
        <form class="navbar-form navbar-left" role="search">
          <div class="form-group">
            <input type="text" class="form-control" placeholder="Search">
          </div>
          <button type="submit" class="btn btn-default">Submit</button>
        </form>
        <ul class="nav navbar-nav navbar-right">
          <li><a href="#">Link</a></li>
          <li class="dropdown">
            <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">Dropdown <span class="caret"></span></a>
            <ul class="dropdown-menu">
              <li><a href="#">Action</a></li>
              <li><a href="#">Another action</a></li>
              <li><a href="#">Something else here</a></li>
              <li role="separator" class="divider"></li>
              <li><a href="#">Separated link</a></li>
            </ul>
          </li>
        </ul>
      </div><!-- /.navbar-collapse -->
    </div><!-- /.container-fluid -->
  </nav>
<span class="glyphicon glyphicon-bed" aria-hidden="true"></span>

<%= yield %>

</body>
</html>

```

Si refrescamos el navegador en cualquier direccion podemos ver que ahora tenemos
la barra de navegación de boostrap. Sin embargo, el código queda muy feo :(.

```html
<!-- app/views/layouts/application.html.erb -->
<!DOCTYPE html>
<html>
<head>
  <title>Zoo</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
  <%= render 'layouts/bootstrap' %>

</head>
<body >
<%= render 'layouts/header' %>

<span class="glyphicon glyphicon-bed" aria-hidden="true"></span>

<%= yield %>

</body>
</html>
```

```html
<!-- app/views/layouts/_boostrap.html.erb -->

<!-- Latest compiled and minified CSS -->
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" integrity="sha384-1q8mTJOASx8j1Au+a5WDVnPi2lkFfwwEAa8hDDdjZlpLegxhjVME1fgjWPGmkzs7" crossorigin="anonymous">

<!-- Optional theme -->
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap-theme.min.css" integrity="sha384-fLW2N01lMqjakBkx3l/M9EahuwpSfeNvV63J5ezn3uZzapT0u7EYsXMjQV+0En5r" crossorigin="anonymous">

<!-- Latest compiled and minified JavaScript -->
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js" integrity="sha384-0mSbJDEHialfmuBBQP6A4Qrprq5OVfW37PRR3j5ELqxss1yVqOtnepnHVP9aJ7xS" crossorigin="anonymous"></script>
```

```html
<!-- app/views/layouts/_header.html.erb-->
<nav class="navbar navbar-default">
  <div class="container-fluid">
    <!-- Brand and toggle get grouped for better mobile display -->
    <div class="navbar-header">
      <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
        <span class="sr-only">Toggle navigation</span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
      <a class="navbar-brand" href="#">Brand</a>
    </div>

    <!-- Collect the nav links, forms, and other content for toggling -->
    <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
      <ul class="nav navbar-nav">
        <li class="active"><a href="#">Link <span class="sr-only">(current)</span></a></li>
        <li><a href="#">Link</a></li>
        <li class="dropdown">
          <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">Dropdown <span class="caret"></span></a>
          <ul class="dropdown-menu">
            <li><a href="#">Action</a></li>
            <li><a href="#">Another action</a></li>
            <li><a href="#">Something else here</a></li>
            <li role="separator" class="divider"></li>
            <li><a href="#">Separated link</a></li>
            <li role="separator" class="divider"></li>
            <li><a href="#">One more separated link</a></li>
          </ul>
        </li>
      </ul>
      <form class="navbar-form navbar-left" role="search">
        <div class="form-group">
          <input type="text" class="form-control" placeholder="Search">
        </div>
        <button type="submit" class="btn btn-default">Submit</button>
      </form>
      <ul class="nav navbar-nav navbar-right">
        <li><a href="#">Link</a></li>
        <li class="dropdown">
          <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">Dropdown <span class="caret"></span></a>
          <ul class="dropdown-menu">
            <li><a href="#">Action</a></li>
            <li><a href="#">Another action</a></li>
            <li><a href="#">Something else here</a></li>
            <li role="separator" class="divider"></li>
            <li><a href="#">Separated link</a></li>
          </ul>
        </li>
      </ul>
    </div><!-- /.navbar-collapse -->
  </div><!-- /.container-fluid -->
</nav>
```

```html
<!-- app/views/layouts/application.html.erb -->
<!DOCTYPE html>
<html>
<head>
  <title>Zoo</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
  <%= render 'layouts/bootstrap' %>

</head>
<body>
<%= render 'layouts/header' %>

<span class="glyphicon glyphicon-bed" aria-hidden="true"></span>

<%= yield %>

</body>
</html>
```
## Enrutamiento Básico
```ruby
get 'categorias', to: :index, controller: 'categories'

namespace :admin do
  resources :categories
end

scope '/admin' do
  resources :categories
end

resources :categories do
  resources :animals
end
```
## Manjeo de usuarios con devise (Esto se va a descontrolar!!)

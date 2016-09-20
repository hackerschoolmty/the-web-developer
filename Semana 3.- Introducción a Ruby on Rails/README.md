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
piolin.category = ave
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
irb> aves = Animal.where(category_id.ave.id)
irb> piolin = aves.find_by_name("Lobo de Wallstreet")
irb> piolin = aves.find_by_name("Piolin")

```
### Relaciones
```ruby
# app/models/category.rb
has_many animals

# app/models/animal.rb
belongs_to category
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
scope :mammals, -> (where(name: "Mamifero"))
scope :birds,   -> (where(name: "Ave"))
scope :reptil,  -> (where(name: "Reptil"))

# app/models/animal.rb
scope :mammals, -> (joins(:category).where("categories.name = ?", "Mamifero"))
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

#### Rutilizar código

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

## Manjeo de usuarios con devise (Esto se va a descontrolar!!)

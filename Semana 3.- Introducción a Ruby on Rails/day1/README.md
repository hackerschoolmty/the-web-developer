# Cómandos básicos en 15 minutos
Unas preguntas rápidas mientras se instala rails o mientras se crea el proyecto
1. En qué otros lenguajes programan?
2. Cuáles otros frameworks usan?
3. Cuánto tardaron en aprender el framework?
4. Cuánto tardan actualmente en hacer un CRUD?

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
$ rails code
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

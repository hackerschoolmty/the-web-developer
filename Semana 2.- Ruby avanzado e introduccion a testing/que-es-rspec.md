# ¿Que es RSpec?

Es una herramienta de testing para Ruby creada para implementar escenarios automatizados de pruebas. Cuenta con un poderoso DSL(domain-specific language) que facilita la escritura de pruebas rapidamente.

## ¿Como lo instalo?

```
% gem install rspec
```

Una vez instalada la gema, es muy facil iniciar un proyecto con rspec, tan solo es necesario correr el siguiente comando:

```
% rspec --init
```

## ¿Como se ve una prueba?


```ruby
require 'spec_helper'
require 'hello_world'

describe HelloWorld do
  describe "#print" do
   it "says hi to everyone" do
	 expect(hellow_world.print).to eql "Hi, and Hello World!"
	end
  end
end
```

### Fuentes:

- http://www.jamesshore.com/Agile-Book/test_driven_development.html
- http://martinfowler.com/bliki/TestDrivenDevelopment.html
- http://betterspecs.org/
- http://www.extremeprogramming.org/rules/testfirst.html
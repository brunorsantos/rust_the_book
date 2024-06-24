Vectores podem armazenar elementos um unico tipo. Sendo que podemos tratar isso colocando os tipos dentro de um enum.

Porem ha casos que desejamos que o usuarios tenham mais liberdade ao trocar e implementar um tipo a ser armazenado em uma estrutura.

Podemos usar como exemplo disso, uma library que lida com um Gui que itera em uma lista de itens que podem ser desenhados implementando um methodo `draw()`. Em que cada estrutura implementa de sua forma.

No nosso caso, gui precisa saber o valores do tipos diferentes e precisa chamar o metodo `draw()` em cada um desses valores com tipos diferentes. 


But we do know that gui needs to keep track of many values of different types, and it needs to call a draw method on each of these differently typed values. It doesn’t need to know exactly what will happen when we call the draw method, just that the value will have that method available for us to call.

Se estivemos lidando com uma liguagem que tem herança, poderiamos ter a classe `Componente` com esse metodo `draw`, em que cada classe sobrescreveria esse metodo implementando de sua forma. Gui salvaria os itens como se todos fosse `Componentes`.

# Definindo o comportamente comum da trait.

Vamos definir uma trait chamada `Draw` que terá um metodo chamado `draw`. Assim vamos poder definir um vector que terá uma trait object. Uma trait object aponta para ambos instancia do tipo que implementa a trait e uma tabela usada para procurar o metodos da trait desse tipo em runtime. Criamos a trait object espficando um ponteiro `&` ou `Box<T>` smart pointer, e então `dyn` seguido da trait.


```rust
pub trait Draw {
    fn draw(&self);
}
```

Sendo assim, podemos definir uma struct `Screen` que é dona de um vector chamado componentes. Esse vector é do tipo `Box<dyn Draw>` que é uma trait object. Será qualquer tipo dentro de uma box que implemente a trait `Draw`.



This syntax should look familiar from our discussions on how to define traits in Chapter 10. Next comes some new syntax: Listing 17-4 defines a struct named Screen that holds a vector named components. This vector is of type Box<dyn Draw>, which is a trait object; it’s a stand-in for any type inside a Box that implements the Draw trait.

```rust
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}
```

Ainda na struct `Screen` podemos implementar um metodo que percorre o vector e chama o draw de cada component. 

```rust
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

Se fosse por exemplo uma struct com tipo generico com trait bounds, nao daria certo. Pois o tipo generico so pode ser substituido por apenas um tipo concreto por vez.

# Implementando a trait

Alguns exemplos de implementacoes da trait:

## Button

```rust
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // code to actually draw a button
    }
}
```

## SelectBox

```rust
use gui::Draw;

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // code to actually draw a select box
    }
}

```


## Criando screen

```rust
use gui::{Button, Screen};

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No"),
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

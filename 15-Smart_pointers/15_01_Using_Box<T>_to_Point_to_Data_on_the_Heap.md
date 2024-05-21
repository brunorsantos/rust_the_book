## Situacoes uteis para usar Box<T>

- Quando se tem um tipo que nao se sabe o tamanho em tempo de compilacao e vc está usando em um contexto que exige isso.(Usando box, usa o heap em que nao é necessario saber o tamanho)
- Quando sem tem uma quantidade grande de dados em que é necessario transferir ownership tendo certeza que nao será feito copia deles
- Quando vc quer ser dono de um valor e vc nao quer saber o tipo, apenas se esse valor implementa um trait.

A unica funcionalidade do Box na pratica é salvar o dado na heap e fazer "indirecao". Que significa ao inves de salvar o valor em si, salvar o pointer de memoria para o valor 

## Habilitando tipos recursivos com Box<T>

Esse codigo nao é possivel:

```rust
enum List {
    Cons(i32, List),
    Nil,
}
```
Pois o rust nao consegue determinar o espaco necessario para alocar o dados, ele vai verificar todos os elementos. `Cons` precisa de um `i32` + `List`, que precisaria do espaço de `Cons` que seria `i32` + `List`... Isso faria uma verificacao infinita.

Ao compilar o rust daria o erro

```
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +

```

Que sugere a "indireção". Que seria ao inves de salvar o valor diretamente, salvar o ponteiro que aponta para o valor. 

O tamanho de um ponteiro nunca muda, sendo assim podemos usar o Box<T> para essa indireção

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```


## Em funcoes

Consirando o exemplo:

```rust
fn largest_i32(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> &char {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
}
```

Podemos unificar em uma funcao `fn largest<T>(list: &[T]) -> &T {` em que lemos: A funcao largest é generica de T. Tendo um parametro que é um slice do tipo T e retorna uma referencia para o tipo T

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

Ao complilar será informado:

```
error[E0369]: binary operation `>` cannot be applied to type `&T`
...
help: consider restricting type parameter `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
  |             ++++++++++++++++++++++
```


Ao seguir o exemplo vamos restringir que T seja um tipo que implemente `PartialOrd` o codigo funciona, ja que que a Sdt library implementa `PartialOrd` para `i32` e `char`

## Em Struct definitions

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

Point é um struct generico de T e os campos x e y sao mesmo tipo.

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

Caso se deseje que cada valor do struct possa ter um tipo

## Em definiçoes de Enum

Temos os exemplos de Option e Result

```rust
enum Option<T> {
    Some(T),
    None,
}
```

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

## Em definicoes de metodos

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

Criando um metodo com nome x em Point<T> que retorna uma referencia para o dado no campo x.

Temos que declarar o T depois do impl, para o Rust identificar que o tipo é generico e nao concreto. Assim metodos escritos com um impl que declaram o tipo generico, vão ser definidos em qualquer instancia do tipo, nao importa qual tipo tipo concreto vai substituir o tipo generico

Tambem é possivel especificar limitacoes em tipo genericos. Definindo metodos sobre apenas um tipo espeficico. Podemos por exemplo implementar metodos apenas para Point<f32>, ao inves para qualquer Point<T>. Nesse caso nao colocar tipos depois de impl

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```


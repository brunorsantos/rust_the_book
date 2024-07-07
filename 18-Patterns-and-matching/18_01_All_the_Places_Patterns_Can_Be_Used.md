Patterns aparecem em vários lugares que as vezes nao sabemos que estamos usando

## match Arms

```
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

Exemplo

```rust
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Precisam ser exaustivos.

Se usar `_` que faz match com tudo e nao faz bind de variavel

## Conditional if let Expressions

Faz match de apenas e caso e pode opcionalmente usar else. O compilar nao checa exaustao
Exemplo

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {color}, as the background");
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

`if let` tambem uitliza como match o conceito `shadowed variables`. Ex `if let Ok(age) = age` cria uma shadowed variavel `age` que contem o valor detro de Ok, que só é valido dentro do escopo das chaves

## while let Conditional Loops


Similar ao if let, porem matem o loop enquanto o pattern contiuar a dar match

Exemplo

```rust
    let mut stack = Vec::new();

    stack.push(1);
    stack.push(2);
    stack.push(3);

    while let Some(top) = stack.pop() {
        println!("{top}");
    }

```

## for Loops

No `for` tudo que vem imediatamente depois já um pattern, por exemplo, em `for x in y` x é o pattern.

Ex:

```rust
    let v = vec!['a', 'b', 'c'];

    for (index, value) in v.iter().enumerate() {
        println!("{value} is at index {index}");
    }
```


## let Statements

Toda vez que se usa um let, se usa pattern

```
let PATTERN = EXPRESSION;
```

In declaracoes como `let x = 5`, o nome da variavel é simplesmente o forma simples de um pattern. Rust compara a expressao a um pattern e faz o assign se encontra. 

Para ver o pattern de forma mais clara, podemos considerar:

```rust
    let (x, y, z) = (1, 2, 3);
```

Rust compara the value (1, 2, 3) com o pattern (x, y, z) e ve se o valor bate com o pattern. Se o numero de valores não batesse daria erro de compilacao por exemplo.

## Function Parameters

TAmbem pode ser patterns

```rust
fn foo(x: i32) {
    // code goes here
}
```

Nesse caso acima `x` tambem é um pattern. Sendo assim podemos tambem usar uma tupla, como fizemos com match:

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({x}, {y})");
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```


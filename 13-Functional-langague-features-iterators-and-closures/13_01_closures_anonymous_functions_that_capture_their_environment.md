Rust’s closures are anonymous functions you can save in a variable or pass as arguments to other functions. 

Unlike functions, closures can capture values from the scope in which they’re defined

```rust
struct Inventory {
    shirts: Vec<ShirtColor>,
}

impl Inventory {
    fn giveaway(&self, user_preference: Option<ShirtColor>) -> ShirtColor {
        user_preference.unwrap_or_else(|| self.most_stocked())
    }
```

Estamos chamando o metodo `unwrap_or_else` do `Option<T>`  definido na std library, que espera uma closure sem argumentos e retorna um T (Mesmo tipo da option).

Interesante reaparar que a closure chama `self.most_stocked()` da instancia atual de Inventory. A std nao sabe nada sobre Inventory. A closure captura uma referencia imutavel da instancia de Inventory e passa ela para o `unwrap_or_else`.

## Inferencia de tipo e anotacao

Outra diferenca entre funcoes e closures, é que closures geralamente nao exigem que se anote o tipo na definicao.

Nao como funcoes que definem um contrato com usuarios (Tanto que nem nome tem). Sa geralmente curtas e relavantes apenas um contexto limitado. O compilador pode inferir os tipos e tipo do retorno.

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

As linhas 3 e 4, precisariam da closure ser avaliadas para poderem complilar, similar ao que ocorre com `let v = Vec::new();`

```rust
    let example_closure = |x| x;

    let s = example_closure(String::from("hello"));
    let n = example_closure(5);
```

Nesse caso, rust vai inferir os tipos do parametros e retorno como String, apos a primeira execucao, depois vai dar erro na segunda pq estamos pasando um valor incopativel (inteiro)

## Capturando referencias ou movendo ownership

3 Formas de capturar o ambiente:

- borrowing immutably
- borrowing mutably 
- taking ownership

Closure decide qual forma baseado no corpo da funcao


Nesse exmplo a closure captura como borowing immutable pois so é necessario para fazer um print:

Como é um referencia immutavel pode ser utilizar a mesma referencia antes de executar a funcao
```rust
fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    let only_borrows = || println!("From closure: {:?}", list);

    println!("Before calling closure: {:?}", list);
    only_borrows();
    println!("After calling closure: {:?}", list);
}
```

Nesse exemplo, como precisamos inserir valores na lista, a closure captura como referencia mutable.

A variavel list, nao pode ser utilizada entre esse borrow e a execucao do closure. Pois nao é possivel alterar a varivel com uma referencia mutavel ativa.
```rust
fn main() {
    let mut list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    let mut borrows_mutably = || list.push(7);

    borrows_mutably();
    println!("After calling closure: {:?}", list);
}
```



Para forçar a transferencia de ownership para closure, usamos a keyword `move`.
```rust
use std::thread;

fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    thread::spawn(move || println!("From thread: {:?}", list))
        .join()
        .unwrap();
}
```

## Movendo valores capturados para fora de closure e Fn traits

Depois da referencia ser capturada ou o ownership do valor do ambiente ser capturado pela closuse. O corpo do closuse define o que acontece com o valor ou referencias nas execucoes futuras. O corpo define o que acontece quando a closure for executada depois.

O corpo pode: Mover o valor capturado para fora da closure, mutar o valor capturado, nem mover nem capturar ou ate nem capturar nada.

Dependendo de como a closure de comporta ela pode implentar as traits:

- `FnOnce` - Closure so pode ser chamada uma vez. Todas elas implementam ao menos essa. Quando uma closuse move o valor capturado para fora do body, ela so implementa ela, e ela pode ser chamada apenas uma vez.

- `FnMut` - Se aplica a closures que nao movem os valores para fora do seu body, mas podem mutar o valor capturado. Podem sem chamadas mais de uma vez

- `Fn` - Nao movem valor para fora e nao mutam os valores. Ou entao nem capturam nada do ambiente em que elas foram definidas. Podem ser chamadas mais de uma vez e nem mutam o ambiente, o que é importante pois pode ser chamados multiplas vezes e concorrentemente.



Usando a implementacao do `unwrap_or_else` da Std:
```rust
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Ela define um trait bound, dizendo que o tipo generico F é  `FnOnce() -> T`. Dizendo que o F deve ser capaz de ser chamado uma vez e retorna T.

Usando 'FnOnce' estamos dizendo que unwrap_or_else vai chamar f no maximo uma vez. Olhando o codigo isso é verdade (Pode ser chamado uma ou nenhuma vez).

Como todas as closures implementam `FnOnce`. Estamos deixando unwrap_or_else o mais flexivel possivel.



Usando como exemplo o metodo `sort_by_key` de Slice. Ele usa `FnMut` e nao `FnOnce` como trait bound. Significando que a closure passada deve ser capaz de ser chamada mais de uma vez.


```rust
impl<T> [T] {
    pub fn sort_by_key<K, F>(&mut self, mut f: F)
    where
        F: FnMut(&T) -> K,
        K: Ord,
    {
        stable_sort(self, |a, b| f(a).lt(&f(b)));
    }
}
```

A closure recebe um argumento, que é uma referencia ao tipo generico do slice atual e retora um K ordenavel.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    list.sort_by_key(|r| r.width);
    println!("{:#?}", list);
}
```

`sort_by_key` precisa receber um `FnMut`, pois obviamente, `sort_by_key` vai precisar de chamar f mais de uma vez para ordenar o slice.

Como a closure `|r| r.width` nem captura nada do ambiente, ela será `Fn` (Que tambem é `FnMut`). Cumprindo o requisito da trait bound.




O proximo exemplo mostra uma closure que implementa apenas `FnOnce` tentando ser passada como parametro para `sort_by_key`.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut sort_operations = vec![];
    let value = String::from("by key called");

    list.sort_by_key(|r| {
        sort_operations.push(value);
        r.width
    });
    println!("{:#?}", list);
}
```

A corpo da closure passada para o `sort_by_key`, move um valor capturado (`value`) que é uma String para fora da closure (fazendo push para o vector `sort_operations`). Assim, essa closure poderá ser chamada apenas uma vez. (Não poderia ser chamada mais de uma vez, pois `value` nao existe mais no environment para ser utilizada uma segunda vez).

Sendo assim esse codigo daria erro.



```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut num_sort_operations = 0;
    list.sort_by_key(|r| {
        num_sort_operations += 1;
        r.width
    });
    println!("{:#?}, sorted in {num_sort_operations} operations", list);
}
```

O exemplo acima funciona, pois a closure esta apenas capturando uma referencia mutavel do ambiente. Sendo assim será `FnMut`

Sendo assim, ao definir um trait bound para closures o ideal é tentar usar o mais flexivel: 

- `FnOnce`, falando que sua implementacao vai chamar apenas uma vez a funcao, sendo assim qualquer closure pode ser utilizada

- `FnMut`, Falando que sua implementacao mesmo precisando chamar mais de uma vez a funcao, lida bem caso a closure precise fazer alguma mutacao.

- `Fn`, vamos estar falando que nossa implementacao alem de precisar de chamar mais de uma vez a funcao, nao permite que haja mutacao do ambiente em que closure foi definida
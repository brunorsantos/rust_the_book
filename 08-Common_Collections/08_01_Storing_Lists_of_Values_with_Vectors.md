Vetores sao salvos na heap, podem ter tamanho alterado durante a execucao.
Podemos ler valores de vetores tanto usando indice quanto utilizando `get()`

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("The third element is {third}");

let third: Option<&i32> = v.get(2);
match third {
    Some(third) => println!("The third element is {third}"),
    None => println!("There is no third element."),
}
```


Esse codigo nao funciona, pois um referencia imutavel está sendo mantida enquanto é tentado fazer uma alteracao usando referencia mutavel com o `push()`

```rust
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];

    v.push(6);

    println!("The first element is: {first}");
```

Como qualquer outra estrutura, quando dropamos um vetor, os elementos desse vetor tambem sao dropados.


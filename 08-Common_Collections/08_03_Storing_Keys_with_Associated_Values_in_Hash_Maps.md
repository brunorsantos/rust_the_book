HashMap<K,V> nao estão no prelude, entao é preciso importar elas da Std no modulo collections


```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name).copied().unwrap_or(0);
```


o metodo `.get()` do Hasmap retorna um `Option<&i32>` (nesse caso). Em que se pode usar `.copied()` no option para transformar em `Option<i32>`

## Ownership

Para tipos que implementam copy, o valor ser copiado, senao é movido ao inseir no hashMap

```rust
    use std::collections::HashMap;

    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name and field_value are invalid at this point, try using them and
    // see what compiler error you get!
```

Tambem é possivel inserir referencias

## Sobrescrever

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);

    println!("{:?}", scores);
```

`.insert()` sempre sobrescreve o valor

Para verificar se ja existe podemos usar o metodo `.entry()`.

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);

    println!("{:?}", scores);
```

O metodo retorna um enum `Entry` que possui um metodo `or_insert` que retorna uma referencia mutavel do valor se a key existe. Senao insere o parametro e ainda assim retorna a referencia mutavel para este valor.

O caso abaixo é um bom exemplo desse comportamento, em que `or_insert` retorna uma referencia mutavel e depois essa referencia mutavel é deferenciada e alterada (Somando +1).

```rust
    use std::collections::HashMap;

    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{:?}", map);
```


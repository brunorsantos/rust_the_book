Iterator é uma trait que tem 2 coisas principais tem um tipo associado `Item` e um metodo `next`.

O tipo associado é o que vai ser retornado, e o next vai retornar um Some ate o iterator ainda ter algo.


Quando fazemos um for como:

```rust
    for x in vec!["a", "b", "c"] {}
```

É como se estivemos escrevendo isso:

```rust
    let mut iter = vec!["a", "b", "c"].into_iter();
    while let Some(x) = iter.next() {}
```

## Associated types observation

Na pratica associated types funciona similar a generic type em trait.

Ex:
```rust
trait Iterator {
    type Item
    fn next(&mut self) -> Option<Self::Item>;
}
```

Da pra ser criado como:

```rust
trait Iterator<Item>
fn next(&mut self) -> Option<Item>;
```

Porem ao implementar o tipo como `SomethingIterable`, eu nao vou querer ter implementacoes diferentes para cada possivel `Item`, sendo assim, melhora na hora de escrever por nao ter que ficar repetindo o tipo generico.


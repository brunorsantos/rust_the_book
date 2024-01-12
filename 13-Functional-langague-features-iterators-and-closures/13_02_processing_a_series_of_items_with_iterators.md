O padrao do iterator permite performar uma tarefa em uma sequencia.

O iterator que é reposavel pela logica de iterar e determinar quando a sequencia acaba.

Iterator sao lazy. Isto é, nao tem efeito ate vc chamar metodos que consome o iterator para utilizarem ele.

Esse codigo nao faz nada util
```rust
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

```

## The Iterator Trait and the next Method

Todos iterators implementam uma trait chamada `Iterator` definida na std

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

A trait acima usa um associated type. Sendo assim, quem implementa a trait `Iterator` deve definir um tipo `Item` e esse tipo será retornado no metodo next. Em outra palavras, `Item` é o que será retornado no iterator.

A trait `Iterator` obriga apenas o preenchimento do method `next`. Que retorna um Option, seja com Some ou None caso chegou no fim da iteracao.

```rust
    #[test]
    fn iterator_demonstration() {
        let v1 = vec![1, 2, 3];

        let mut v1_iter = v1.iter();

        assert_eq!(v1_iter.next(), Some(&1));
        assert_eq!(v1_iter.next(), Some(&2));
        assert_eq!(v1_iter.next(), Some(&3));
        assert_eq!(v1_iter.next(), None);
    }
```
Pontos a observar

- O `next` muda o estado interno, entao tivemos que usar a variavel com mut
- Cada `next` come um item do iterator
- os valores que temos sao referencias imutavies dos valores do vector.
- Para tomar ownership do v1 deve se usar `into_iter`
- Se quizer uma referencia mutavel, deve se usar `iter_mut`

## Metodos que consumem o iterator

A trait `Iterator` ja tem diversos metodos com default implementations na std. Geralmente eles chamam o metodo `next`.

Esses metodos sao chamados de `consuming adaptors`, pois eles usam o iterator. Ex: o metodo `sum` que obtem o ownership do iterator (iterando sobre ele repetidamente chamand o `next`, pois isso consume o iterator) e retorna a quantidade total de itens.

```rust
    #[test]
    fn iterator_sum() {
        let v1 = vec![1, 2, 3];

        let v1_iter = v1.iter();

        let total: i32 = v1_iter.sum();

        assert_eq!(total, 6);
    }
```

`v1_iter` não está mais disponivel

## Metodos que produzem outro iterator


# Seguindo o point para um valor

Uma referencia regular é um tipo de ponteiro. (Uma seta para um valor salvo em outro lugar)

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

Como Y é na verdade `&i32`
Quando usamos `*` estamos fazendo `deref` em y, entao vamos vamos acesso ao inteiro que ele esta apontando.

# Usando `Box<T>` como uma referencia

Podemos reescrever a mesma coisa porem usando `Box<T>` ao inves de referencia. O operador pode derreference pode ser usando em `Box<T>`.

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

A diferença é que `Box<T>` está apontando para um copia do valor de x ao inves de apontar para um referencia de x.


# Definindo o proprio Smart Pointer e implementando a Deref Trait

Considerando o `MyBox<T>`

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

Ao usar o `*` precisamos que o tipo implemente a trait deref, que exige que implementamos um metodo chamado `deref`. Que pega emprestado o self e retorna uma referencia do inner.

Exmplo de implementacao em cima do `MyBox` personalizado:

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}
```

Com essa implementacao podemos usar dessa forma:

```rust
fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

Sem a Deref trait o rust so pode fazer derreference &.

Quando usamos `*y` rust faz:

```
*(y.deref())
```

# Implicit Deref Coercions com funcoes em metodos

Deref Coercions, converte uma referencia para um tipo implmenta the Deref trait para uma referencia de outro tipo. (Em parametro de funcoes e metodos)

Ex: &String para @str. Pq `String` implementa `Deref` retornando `&str`

```rust
impl ops::Deref for String {
    type Target = str;

    #[inline]
    fn deref(&self) -> &str {
        unsafe { str::from_utf8_unchecked(&self.vec) }
    }
}
```

Sendo assim podemos fazer multiplas Deref Coercions de uma vez

```rust
fn hello(name: &str) {
    println!("Hello, {name}!");
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

Funciona pois MyBox vai ter deref para o &T (&String no caso) e &String tem deref para &str

# Deref Coercion com Mutabilidade

- From &T to &U when T: Deref<Target=U>
- From &mut T to &mut U when T: DerefMut<Target=U>
- From &mut T to &U when T: Deref<Target=U>
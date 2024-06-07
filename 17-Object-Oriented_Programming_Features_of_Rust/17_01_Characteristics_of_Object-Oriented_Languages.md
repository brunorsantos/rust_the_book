# Objetos contem dado e comportamento

Segundo o livro do gang of four

> Object-oriented programs are made up of objects. An object packages both data and the procedures that operate on that data. The procedures are typically called methods or operations.

Segundo essa definicao Rust é orientado a objetos: strucs e enunm tem os dados e `impl` block dao o metodos de structs e enums. Mesmo que structs e enum com metodos nao são chamados de objetos no Rust.


# Encapsulamento

Outro aspecto é esse que significa que detalhes da implementacao nao estão disponiveis para o codigo usando o objeto. Sendo a API publica a unica forma de interagir.

```
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}
```

A struct está marcada como `pub`, entao outro codigo pode utilzar.

```rust
impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            }
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```

Apenas o metodos publicos `add`, `remove` and `avarage` estão disponiveis.

Rust coumpre os requisitos para utilizar encapsulamento.

# Heranca como type system e como compartilhamento de codigo

Podemos escolher herança por 2 motivos: 

## Reuso de codigo

Voce pode implementar um comportamento em um tipo e a heranca te habilita de reusar essa implementaao para um tipo diferente. Podemos fazer isso em rust usando default method implementation de traits. Sendo que podemos tambem sobrescrever essa implementacao default quando implementamos a trait.

## Type system

Para habilitar um tipo filho de ser usado em mesmos lugares que um tipo pai. (Polimorfismo), assim vc pode substituir objetos um por outro em tempo de execucao se eles compartilharem caracteristicas.

Rust usa uma abordagem diferente ao usar trait objects ao inves de heranca.
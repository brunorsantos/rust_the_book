# Unit-like Structs

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

Structs sem campo, são uteis quando se precisa implementar uma trait em que nao se tem nenhum dado para salvar no tipo em questao.


# Ownership of Struct Data

É possivel que struct grave referencias (que vao ser owned por por outra coisa), mas para isso deve se usar lifetimes. Que vao garantir que a dado referenciado pela struct viva enquanto a struct.
Existem dois profiles `dev` e `release`. 
Em que o `dev` é o padrao ao executar build e possiu valores adequados para desenvolvimento..

É possivel sobrescrever valores para cada profile no Cargo.toml

```
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` por exemplo, controla o numero de otimizacoes que o Rust vai aplicar no codigo entre 0 e 3. Quando mais aplica, mais a compilacao vai demorar.

[Lista de configuracoes](https://doc.rust-lang.org/cargo/reference/profiles.html), nas doc do cargo
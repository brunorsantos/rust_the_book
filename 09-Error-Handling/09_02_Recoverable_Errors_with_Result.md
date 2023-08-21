Como Option o Result está carregado no prelude, entao nao precisa ser importado.

`unwrap_or_else` é uma boa alteranativa passando closure para match chegando Result<T, E>

## Operador ?

Valores de erro que tem o ? chamados por eles, sao chamado pela funcao `from` definidas na trait `From` na std library, que é usada para converter tipos de um para outro. 
Quando o operador ? chama a funcao from e tipo do erro é convertido para o erro definido na funcao atual. 
Isto é util quando uma funcao retorna um tipo tipo de erro que representa todos os caminhos que uma funcao deve falhar, mesmo que por razoes diferentes.

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

Por exemplo, poderiamos mudar `read_username_from_file` para retorna uma tipo customizado `OurError`, porem teriamos que definir  `impl From<io::Error> for OurError` para construir uma instancia de `OurError` a partir de um `io::Error` 

## Anyhow


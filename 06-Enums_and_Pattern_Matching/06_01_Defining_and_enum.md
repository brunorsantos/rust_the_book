Uma vantagem dos enums é alem de poder tem um valor associado a cada valor do enum, é que cada valor pode ter um tipo diferente associado.



```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```

É possivel ter valores nomeado em enums tambem:

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

Sendo que enums tambem aceitam metodos como struct usando `impl`
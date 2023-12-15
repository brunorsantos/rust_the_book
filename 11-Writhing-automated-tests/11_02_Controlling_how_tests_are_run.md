
O comportamenento padrao do `cargo test` é rodar os testes em paralo e caputar o output gerado durante a execucao.

Algumas opcoes sao para o cargo test e outras para o binario gerado

o Separador -- no meio indicao se o arg é para o comando ou para o binario.

Ex:
```
cargo test -- --help
```

Para nao executar os testes em paralelo, podemos limitar o numero de threads para um

```
$ cargo test -- --test-threads=1
```

# Showing function output

Por padrao se um teste passa, rust captura qualquer coisa no standard output. Sendo assim quando um teste passa a gente nao ve nenhum resultado de um println

Para vizualizar devemos usar o argumento `--show-output`

```
$ cargo test -- --show-output
```

# Running Single Tests

Podemos passar no test o nome ou parte do nome (podendo encontrar mais de um teste) 

```
cargo test add
```
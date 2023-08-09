Quando copiamos uma variavel `s1` do tipo `String` para outra `s2` isso ocorre:

![Alt text](image.png)

Em que s1 nao sera mais possivel de ser utilizada (foi moved)

Tipos primitivos que estao na stack implementam `Copy`, significando que nao sao movidos quando passados de variavel para outra ou em um funcão.

É possivel implementar copy em structs e enums que em que todos os tipo dentro tenham tamanho determinado (sao possiveis de ser salvos na stack). É bem util em enums simples por exemplo.

# Permitindo transferencia de ownership entre Threads com `Send`

A trait `Send` de `std:marker` indica que o ownership do valores do tipo podem ser transferido implementando `Send` podem ser transferidos entre threads.

Quase todo tipo implementa `Send`, com algumas excessões como `RC<T>`. Ela é implementada para uso em single-thread, para evitar pagar pelo performance de lidar com thread-safe

Tipos compostos totalmente de `Send`, vão automaticamente ser marcados como `Send` tambem.
Quase todos tipos primitivo são `Send`, exceto raw pointes

# Permitindo acessos por multiplas threads com `Sync`

A trait `Sync` de `std:marker` indica que é seguro para o tipo implementando `Sync` ser referenciado entre multiplas thread.(Sync permite que referências a um valor sejam compartilhadas entre threads.)

Se um tipo T é `Sync`, então uma referência imutável a esse tipo (&T) pode ser enviada para outra thread sem problemas.
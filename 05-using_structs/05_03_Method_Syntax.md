# Defining Methods

`&self` é atalho para `self: &Self`.

Usar `self` fazendo move em metodos é raro. É usado geralmente quando quer transformar a instancia e outra estrutura, previnindo o chamador de utilizar a instancia depois disso.

## Associated Functions

São os construtores chamados usando `::`
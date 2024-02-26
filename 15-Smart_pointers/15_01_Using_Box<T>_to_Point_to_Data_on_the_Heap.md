Situacoes uteis para usar Box<T>

- Quando se tem um tipo que nao se sabe o tamanho em tempo de compilacao e vc está usando em um contexto que exige isso.(Usando box, usa o heap em que nao é necessario saber o tamanho)
- Quando sem tem uma quantidade grande de dados em que é necessario transferir ownership tendo certeza que nao será feito copia deles
- Quando vc quer ser dono de um valor e vc nao quer saber o tipo, apenas se esse valor implementa um trait.
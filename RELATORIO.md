# Relatório do Laboratório 2 - Chamadas de Sistema

---

## Exercício 1a - Observação printf() vs 1b - write()

### Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 9 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre printf() e write()?**

```
O printf() é um comando da linguagem, em que esse aciona o write() para poder imprimir.
Já o Write() é um comando do sistema operacional linux, em que esse se comunica com o kernel, para este executar a ação.
```

**3. Qual implementação você acha que é mais eficiente? Por quê?**

```
O printf(), por ter um buffer para agrupar diversas chamadas de write(), é mais eficiente.
Assim, ele chama o syscall quando está cheio ou forçado.
```

---

## Exercício 2 - Leitura de Arquivo

### Resultados da execução:
- File descriptor: _____
- Bytes lidos: _____

### Comando strace:
```bash
strace -e open,read,close ./ex2_leitura
```

### Análise

**1. Por que o file descriptor não foi 0, 1 ou 2?**

```
Os registradores 0, 1 e 2 são reservados para entradas e saídas padrão (0 = stdin, 1 = stdout e 2 = stderr). O fd retornado foi o 3, o primeiro livre para o processo utilizar.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
A variável bytes_lidos nunca armazenou > 0 (números negativos), logo, nenhum erro no processo ocorreu, entendendo que ele foi executado em sua completude e sem erros.
```

---

## Exercício 3 - Contador com Loop

### Resultados (BUFFER_SIZE = 64):
- Linhas: 24 (esperado: 25)
- Caracteres: 1299
- Chamadas read(): 21
- Tempo: 0.000591 segundos

### Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |        82       | 0.000249  |
| 64          |        24       | 0.000591  |
| 256         |        6        | 0.000067  |
| 1024        |        2        | 0.000075  |

### Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Com o buffer maior, o printf consegue armazenar mais chamadas para serem feitas ao write, acionando-o menos vezes.
```

**2. Como você detecta o fim do arquivo?**

```
Quando a viariável bytes_lidos armazena o valor 0, retornado do comando read, significa que acabou o conteúdo.
```

---

## Exercício 4 - Cópia de Arquivo

### Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000193 segundos
- Throughput: 6901.72 KB/s

### Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [X] Idênticos [ ] Diferentes

### Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Para ter certeza que o arquivo foi copiado corretamente e em sua totalidade.
```

**2. Que flags são essenciais no open() do destino?**

```
O_CREAT: Para criar o arquivo caso ele não exista.
O_WRONLY: Para indicar que a abertura é apenas para escrita.
O_TRUNC: Para limpar o arquivo caso ele já exista e possua conteúdo.
```

---

## Análise Geral

### Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
Sua chamada já é a demonstração da transição, pois a syscall funciona como ponte entre o pedido do usuário ao kernell, onde este irá executar o que o modo usuário precisa.
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Indicar ao sistema qual arquivo deve ser selecionado, atribuindo um número (fd) ao arquivo.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Quanto maior o buffer, mais os comandos da linguagem conseguem acumular chamadas ao sistema, precisando mudar de modo (usuário/kernell) menos vezes e perdendo menos tempo entre transições. No entanto, quando maior o buffer mais memória deve ser alocada para este.
```

### Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** cp do sistema

**Por que você acha que foi mais rápido?**

```
Eu diria que o o do sistema foi mais rápido pois ele tem 0m0.003s de sys e 0m0.000s de user e o outro tem 0m0.002s em cada.
Imagino que o sistema tenha sido mais rápido por não precisar ficar alterando de modo (até porque o user dele estpa zerado).
```

---

## Entrega

Certifique-se de ter:
- [X] Todos os códigos com TODOs completados
- [X] Traces salvos em `traces/`
- [X] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Exercícios do Laboratório 2 - Instruções Passo-a-Passo

Este arquivo contém instruções detalhadas para cada exercício. Siga os passos na ordem para obter o máximo aprendizado.

## Preparação Inicial

### 1. Verificar Ferramentas

```bash
# Verificar se tudo está instalado
gcc --version
strace --version
make --version
```

### 2. Compilar Todos os Exercícios

```bash
# Opção 1: Usando Makefile (mais fácil)
make all

# Opção 2: Usando gcc diretamente
gcc src/ex1a_printf.c -o ex1a_printf
gcc src/ex1b_write.c -o ex1b_write
gcc src/ex2_leitura.c -o ex2_leitura
gcc src/ex3_contador.c -o ex3_contador
gcc src/ex4_copia.c -o ex4_copia
```

### 3. Criar Diretório para Traces

```bash
mkdir -p traces
```

---

## Exercício 1a - Observando printf()

**Objetivo:** Entender como printf() se comporta com strace.

### Passo 1: Executar Normalmente

```bash
./ex1a_printf
```

### Passo 2: Executar com strace

```bash
strace -e write ./ex1a_printf
```

**O que observar:**
- Quantas chamadas `write()` aparecem?
- printf() pode agrupar dados

### Passo 3: Salvar Trace

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
```

---

## Exercício 1b - Observando write()

**Objetivo:** Entender como write() se comporta com strace.

### Passo 1: Executar Normalmente

```bash
./ex1b_write
```

### Passo 2: Executar com strace

```bash
strace -e write ./ex1b_write
```

**O que observar:**
- Cada write() gera exatamente uma syscall
- File descriptor 1 = stdout

### Passo 3: Salvar Trace

```bash
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
```

### Comparação 1a vs 1b:

1. **Quantas syscalls cada método gerou?**
2. **Por que há diferença?**
3. **Qual é mais eficiente?**

---

## Exercício 2 - Leitura Básica de Arquivo

**Objetivo:** Implementar leitura usando syscalls básicas.

### Passo 1: Analisar o Código

1. Abra `src/ex2_leitura.c`
2. Leia os comentários e identifique os TODOs
3. Entenda o que cada TODO deve fazer

### Passo 2: Completar os TODOs

**TODO 1:** Abrir arquivo
```c
fd = open("dados/teste1.txt", O_RDONLY);
```

**TODO 2:** Verificar erro de abertura
```c
if (fd < 0) {
```

**TODO 3:** Ler dados
```c
bytes_lidos = read(fd, buffer, BUFFER_SIZE - 1);
```

**TODO 4:** Verificar erro de leitura
```c
if (bytes_lidos < 0) {
```

**TODO 5:** Adicionar terminador
```c
buffer[bytes_lidos] = '\0';
```

**TODO 6:** Fechar arquivo
```c
if (close(fd) < 0) {
```

### Passo 3: Compilar e Testar

```bash
gcc -Wall -g src/ex2_leitura.c -o ex2_leitura
./ex2_leitura
```

**Deve mostrar:**
- File descriptor usado
- Número de bytes lidos
- Conteúdo do arquivo

### Passo 4: Observar com strace

```bash
# Ver syscalls de arquivo
strace -e open,read,close ./ex2_leitura

# Salvar trace completo
strace -o traces/ex2_trace.txt ./ex2_leitura
```

### Passo 5: Experimentos
strace
1. **Mudar tamanho do buffer:**
   - Mude `BUFFER_SIZE` para 10
   - Recompile e execute
   - O que acontece com arquivos maiores que o buffer?

2. **Arquivo inexistente:**
   - Mude o nome do arquivo para algo que não existe
   - Observe as mensagens de erro

### Perguntas para Reflexão:

1. **Qual file descriptor foi usado? Por que não 0, 1 ou 2?**
2. **O arquivo foi lido completamente? Como você sabe?**
3. **O que acontece se esquecer de fechar o arquivo?**
4. **Por que verificar retorno de cada syscall?**

---

## Exercício 3 - Contador de Linhas com Loop

**Objetivo:** Implementar loop de leitura e analisar múltiplas syscalls.

### Passo 1: Analisar a Estrutura

1. Abra `src/ex3_contador.c`
2. Note que o `BUFFER_SIZE` é pequeno (64 bytes)
3. Isto força múltiplas chamadas `read()`

### Passo 2: Completar os TODOs

**TODO 1:** Condição do loop
```c
while ((bytes_lidos = read(fd, buffer, BUFFER_SIZE)) > 0) {
```

**TODO 2:** Contar quebras de linha
```c
if (buffer[i] == '\n') {
    total_linhas++;
}
```

**TODO 3:** Somar caracteres
```c
total_caracteres += bytes_lidos;
```

**TODO 4:** Verificar erro
```c
if (bytes_lidos < 0) {
```

### Passo 3: Compilar e Testar

```bash
gcc -Wall -g src/ex3_contador.c -o ex3_contador
./ex3_contador
```

**Deve mostrar:**
- Número de linhas (deve ser 25)
- Total de caracteres
- Número de chamadas read()
- Tempo de execução

### Passo 4: Análise com strace

```bash
# Contar syscalls
strace -c ./ex3_contador

# Ver detalhes dos reads
strace -e read ./ex3_contador

# Salvar estatísticas
strace -c -o traces/ex3_stats.txt ./ex3_contador
```

### Passo 5: Experimentos com Buffer

1. **Buffer muito pequeno (16 bytes):**
```c
#define BUFFER_SIZE 16
```

2. **Buffer médio (256 bytes):**
```c
#define BUFFER_SIZE 256
```

3. **Buffer grande (1024 bytes):**
```c
#define BUFFER_SIZE 1024
```

Para cada tamanho:
- Recompile e execute
- Use `strace -c` para contar syscalls
- Anote os resultados

### Tabela de Análise:

| Buffer Size | Syscalls read() | Tempo (s) | Chars/seg |
|-------------|-----------------|-----------|-----------|
| 16          | ?               | ?         | ?         |
| 64          | ?               | ?         | ?         |
| 256         | ?               | ?         | ?         |
| 1024        | ?               | ?         | ?         |

### Perguntas para Análise:

1. **Como o tamanho do buffer afeta o número de syscalls?**
2. **Qual é a relação entre syscalls e performance?**
3. **Todas as chamadas read() retornaram BUFFER_SIZE bytes?**
4. **Como você sabe que chegou ao fim do arquivo?**

---

## Exercício 4 - Cópia de Arquivo (Mais Complexo)

**Objetivo:** Implementar programa completo de cópia de arquivo.

### Passo 1: Entender a Complexidade

Este exercício envolve:
- Abrir arquivo para leitura E escrita
- Loop read/write coordenado
- Tratamento de erros em múltiplas etapas
- Verificação de integridade

### Passo 2: Completar os TODOs (Principais)

**TODO 1:** Abrir origem
```c
fd_origem = open("dados/origem.txt", O_RDONLY);
```

**TODO 2:** Criar destino
```c
fd_destino = open("dados/destino.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
```

**TODO 3:** Loop de cópia
```c
while ((bytes_lidos = read(fd_origem, buffer, BUFFER_SIZE)) > 0) {
```

**TODO 4:** Escrever dados
```c
bytes_escritos = write(fd_destino, buffer, bytes_lidos);
```

**TODO 5:** Verificar escrita
```c
if (bytes_escritos != bytes_lidos) {
```

**TODO 6:** Atualizar contador
```c
total_bytes += bytes_escritos;
```

**TODO 7:** Verificar erro de leitura
```c
if (bytes_lidos < 0) {
```

**TODO 8:** Fechar arquivos
```c
if (close(fd_origem) < 0) { ... }
if (close(fd_destino) < 0) { ... }
```

### Passo 3: Compilar e Testar

```bash
gcc -Wall -g src/ex4_copia.c -o ex4_copia
./ex4_copia
```

### Passo 4: Verificar Integridade

```bash
# Verificar se a cópia foi perfeita
diff dados/origem.txt dados/destino.txt

# Se não houver saída, a cópia foi bem-sucedida!

# Comparar tamanhos
ls -la dados/origem.txt dados/destino.txt
```

### Passo 5: Análise Detalhada com strace

```bash
# Ver padrão read/write
strace -e open,read,write,close ./ex4_copia

# Com timestamps para ver performance
strace -T -e read,write ./ex4_copia

# Salvar trace completo
strace -o traces/ex4_trace.txt ./ex4_copia
```

### Passo 6: Experimentos de Performance

Teste diferentes tamanhos de buffer editando `BUFFER_SIZE`:

```bash
# Buffer pequeno
sed -i 's/#define BUFFER_SIZE 256/#define BUFFER_SIZE 64/' src/ex4_copia.c
gcc -Wall -g src/ex4_copia.c -o ex4_copia
time ./ex4_copia

# Buffer grande  
sed -i 's/#define BUFFER_SIZE 64/#define BUFFER_SIZE 1024/' src/ex4_copia.c
gcc -Wall -g src/ex4_copia.c -o ex4_copia
time ./ex4_copia
```

### Análise do Trace

Examine `traces/ex4_trace.txt` e procure:

1. **Sequência de abertura:**
```
open("dados/origem.txt", O_RDONLY) = 3
open("dados/destino.txt", O_WRONLY|O_CREAT|O_TRUNC, 0644) = 4
```

2. **Padrão read/write:**
```
read(3, "conteudo...", 256) = 256
write(4, "conteudo...", 256) = 256
read(3, "mais conteudo...", 256) = 180
write(4, "mais conteudo...", 180) = 180
read(3, "", 256) = 0
```

3. **Fechamento:**
```
close(3) = 0
close(4) = 0
```

### Perguntas para Análise:

1. **O número de reads e writes é igual? Por quê?**
2. **Todos os writes escreveram exatamente o que foi lido?**
3. **Como você saberia se o disco ficou cheio?**
4. **Por que as flags O_CREAT e O_TRUNC são importantes?**
5. **O que acontece se esquecer de fechar os arquivos?**

---

## Análise Final e Relatório

### Compilar Todos os Traces

Certifique-se de ter salvado traces de todos os exercícios:

```bash
ls traces/
# Deve mostrar:
# ex1a_trace.txt
# ex1b_trace.txt
# ex2_trace.txt  
# ex3_stats.txt
# ex4_trace.txt
```

### Comparar Performance

Use os dados coletados para comparar:

1. **Número de syscalls vs tamanho do buffer**
2. **Tempo de execução vs eficiência**
3. **Diferenças entre métodos de I/O**

### Conectar com a Teoria

Relacione suas observações com os conceitos da aula:

- **Modo Usuário vs Kernel:** Cada linha do strace é uma transição
- **File Descriptors:** Como o kernel identifica arquivos
- **Bufferização:** Por que bibliotecas usam buffers internos
- **Performance:** Overhead de syscalls e otimizações

### Preparar Entrega

1. **Código completo** (todos os TODOs preenchidos)
2. **Traces salvos** na pasta `traces/`
3. **Relatório** preenchido com análises
4. **Commit no Git** com todas as modificações

```bash
git add .
git commit -m "Completar todos os exercícios do laboratório"
git push
```

## Problemas Comuns e Soluções

### Erro de Compilação

```bash
# Se der erro de biblioteca não encontrada
sudo apt install libc6-dev

# Se der erro de permissão
chmod +x ex2_leitura
```

### Erro de Arquivo Não Encontrado

```bash
# Verificar se arquivos de teste existem
ls dados/
# Se não existirem:
make dados
```

### strace Não Funciona

```bash
# Instalar strace
sudo apt install strace

# Se ainda não funcionar, pode ser restrição de segurança
sudo strace ./programa
```

### Programa Não Executa

```bash
# Verificar permissões
ls -la ex2_leitura
# Se necessário:
chmod +x ex2_leitura

# Verificar dependências
ldd ex2_leitura
```

**Lembre-se:** Se tiver dúvidas, consulte a documentação em `docs/` e use `make help` para ver comandos disponíveis!
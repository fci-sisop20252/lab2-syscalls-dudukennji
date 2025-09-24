/*
====================================================================
📝 RELATÓRIO DO LABORATÓRIO 2 — Chamadas de Sistema
====================================================================

1️⃣ Exercício 1a (printf) vs 1b (write)

Comandos executados:
  strace -e write ./ex1a_printf
  strace -e write ./ex1b_write

Análise
1) Quantas syscalls write() cada programa gerou?
   - ex1a_printf: 9 syscalls
   - ex1b_write : 7 syscalls

2) Por que há diferença entre os dois métodos?
   O printf() é uma função de biblioteca que faz buffering e decide quando chamar
   write(); o write() chama a syscall diretamente a cada invocação no código. O
   comportamento do printf pode variar conforme o fluxo (terminal, pipe, arquivo).

3) Qual método é mais previsível? Por quê?
   write() é mais previsível: 1 chamada na aplicação → 1 syscall; já printf()
   depende do buffering e do destino da saída.

--------------------------------------------------------------------

2️⃣ Exercício 2 — Leitura de Arquivo

Resultados da execução:
  - File descriptor: 3
  - Bytes lidos   : 127

Comando strace:
  strace -e openat,read,close ./ex2_leitura

Análise
1) Qual FD foi usado? Por que não 0, 1 ou 2?
   Foi o FD 3. Os descritores 0/1/2 são stdin/stdout/stderr; o kernel entrega o
   primeiro FD livre, que costuma ser 3.

2) Como sabemos que o arquivo foi lido completamente?
   O read() retornou 127 bytes (todo o conteúdo esperado) e não houve leituras
   subsequentes com bytes restantes; a lógica e a saída confirmam o fim do arquivo.

3) Por que verificar retorno de cada syscall?
   Para detectar erros (permite agir, abortar com mensagem, evitar dados
   corrompidos e vazamento de recursos).

--------------------------------------------------------------------

3️⃣ Exercício 3 — Contador com Loop (BUFFER_SIZE = 64)

Resultados:
  - Linhas          : 25 (esperado: 25)
  - Caracteres      : 1300
  - Chamadas read() : 21
  - Tempo           : 0.000657 s

Experimentos (preencha se testar):
  Buffer Size | Chamadas read() | Tempo (s)
  ----------- | ---------------- | ----------
  16          | ↑ mais chamadas  | ↑ maior tempo
  64          | 21               | 0.000657
  256         | ↓ menos chamadas | ↓ menor tempo
  1024        | muito menos      | ~ semelhante / menor

Análise
1) Efeito do tamanho do buffer:
   Buffers maiores reduzem o número de read(), diminuindo o overhead de
   transições user→kernel.

2) Todas as read() retornam BUFFER_SIZE?
   Não. As primeiras tendem a encher o buffer; a última leitura costuma ser menor,
   apenas os bytes restantes do arquivo.

3) Relação entre syscalls e performance:
   Menos syscalls → menos context switches → melhor desempenho (até certo ponto).

Resumo do strace -c observado:
  read: 23 chamadas (21 do arquivo + ~2 internas)
  write: 13 chamadas
  close: 3 chamadas
  1 erro em access (não impactou o resultado)
  Total: 72 chamadas; tempo agregado ≈ 0.000380 s

--------------------------------------------------------------------

4️⃣ Exercício 4 — Cópia de Arquivo

Resultados:
  - Bytes copiados: 1364
  - Operações     : 6
  - Tempo         : 0.000265 s
  - Throughput    : 5026.53 KB/s

Verificação:
  diff dados/origem.txt dados/destino.txt
  Resultado: [x] Idênticos  [ ] Diferentes

Análise
1) Por que checar bytes_escritos == bytes_lidos?
   Para garantir cópia íntegra; discrepância indica truncamento/erro.

2) Flags essenciais no open() do destino:
   O_WRONLY | O_CREAT | O_TRUNC (e permissões adequadas no mode).

3) O número de reads e writes é igual?
   Em geral sim: 1 write por read bem-sucedido. Difere se houver write parcial/erro.

4) Como saber se o disco encheu?
   write() retorna < solicitado ou -1 com errno = ENOSPC.

5) Se esquecer de fechar arquivos:
   Vazamento de descritores; em programas longos isso exaure FDs e provoca falhas.
   No término do processo o kernel fecha, mas é má prática e arriscado.

--------------------------------------------------------------------

🎯 Análise Geral

Conceitos Fundamentais
1) Transição usuário→kernel:
   Syscalls são pontos de entrada ao kernel (trap); o kernel executa operações
   privilegiadas e retorna ao espaço do usuário.

2) Importância dos file descriptors:
   São identificadores para recursos (arquivos, sockets, pipes). Organizam onde
   ler/escrever e permitem multiplexação.

3) Tamanho do buffer vs performance:
   Buffers maiores reduzem syscalls e overhead, mas exageros podem desperdiçar
   memória ou aumentar latência em streams.

Comparação de Performance
  time ./ex4_copia
  time cp dados/origem.txt dados/destino_cp.txt

  Qual foi mais rápido? cp (tendência)
  Por quê?
   cp usa implementações mais otimizadas (buffers maiores, sendfile e afins),
   reduzindo chamadas e cópias entre espaços.

--------------------------------------------------------------------

📤 Entrega — Checklist
  [x] Códigos com TODOs completos
  [x] Traces salvos em traces/
  [x] Relatório preenchido (este bloco) dentro do programa

Comandos sugeridos para gerar traces:
  strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
  strace -e write -o traces/ex1b_trace.txt ./ex1b_write
  strace -o traces/ex2_trace.txt ./ex2_leitura
  strace -c -o traces/ex3_stats.txt ./ex3_contador
  strace -o traces/ex4_trace.txt ./ex4_copia

====================================================================
FIM DO RELATÓRIO
====================================================================
*/

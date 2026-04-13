# CMP 1059 A01 2026.1 — Trabalho 2

## Scoreboard: Análise de Hazards e Escalonamento Dinâmico

Simulador: http://www.ecs.umass.edu/ece/koren/architecture/scoreboard/

---

## 1. Introdução

Este relatório apresenta a análise de execução de programas utilizando a técnica de Scoreboard (escalonamento dinâmico), conforme proposto no Trabalho 2 da disciplina CMP 1059. O objetivo é demonstrar a ocorrência de hazards do tipo RAW (Read After Write), atrasos por WAR (Write After Read), atrasos estruturais (structural hazards) e discutir a possibilidade de hazards WAW (Write After Write), incluindo a solução por renomeação de registradores.

A ferramenta utilizada é o simulador de Scoreboard disponível no site do Prof. Israel Koren (UMass), que implementa o algoritmo clássico de escalonamento dinâmico baseado no CDC 6600, conforme descrito por Hennessy & Patterson, além de um simulador pessoal interativo em HTML.

---

## 2. Fundamentos do Scoreboard

### 2.1 Escalonamento Dinâmico

Em pipelines simples com escalonamento estático, quando uma instrução j depende de uma instrução longa i em execução, todas as instruções após j ficam bloqueadas. O escalonamento dinâmico permite que instruções independentes ultrapassem instruções bloqueadas, executando fora de ordem.

O Scoreboard é uma técnica que permite execução fora de ordem quando há recursos suficientes e não há dependências de dados. Todas as instruções passam pelo estágio de Issue em ordem (in-order issue), mas podem executar e completar fora de ordem.

### 2.2 Os Quatro Estágios do Scoreboard

- **Issue:** Verifica se a unidade funcional está livre e se não há WAW. Se houver hazard estrutural ou WAW, a instrução e todas as subsequentes ficam bloqueadas.

- **Read Operands:** Monitora a disponibilidade dos operandos fonte. Só lê quando nenhuma instrução anterior ativa vai escrever naquele registrador. Resolve hazards RAW dinamicamente.

- **Execution:** A unidade funcional executa a operação. Duração: Integer/Load = 1 ciclo, FP Add/Sub = 2 ciclos, FP Multiply = 10 ciclos, FP Divide = 40 ciclos.

- **Write Result:** Antes de escrever, verifica WAR: alguma instrução anterior ainda precisa ler o registrador destino? Se sim, a escrita é adiada.

### 2.3 Tipos de Hazards

- **RAW (Read After Write):** Dependência verdadeira. A instrução j precisa ler um registrador que i ainda não escreveu. Resolvido no estágio Read Operands.

- **WAR (Write After Read):** Anti-dependência. A instrução j quer escrever em um registrador que i ainda não leu. Atrasa a escrita no estágio Write Result.

- **WAW (Write After Write):** Dependência de saída. Duas instruções escrevem no mesmo registrador. Bloqueia o Issue da segunda.

- **Estrutural:** Unidade funcional necessária está ocupada. Bloqueia o Issue.

---

## 3. Programa e Resultados da Simulação

O programa utilizado é o exemplo clássico de Hennessy & Patterson, simulado na ferramenta de Koren:

```
1. LD F6, 34+R2
2. LD F2, 45+R3
3. MULTD F0, F2, F4
4. SUBD F8, F6, F2
5. DIVD F10, F0, F6
6. ADDD F6, F8, F2
```

Configuração: Integer = 1 ciclo, FP Add = 2 ciclos, FP Multiply = 10 ciclos, FP Divide = 40 ciclos. Unidades: 1 Integer, 2 Mult, 1 Add, 1 Divide.

### 3.1 Tabela de Status das Instruções (Resultado do Simulador)

| Instrução | Issue | Read Op | Exec | Write Result |
|---|---|---|---|---|
| LD F6, 34+R2 | 1 | 2 | 3 | 4 |
| LD F2, 45+R3 | 5 | 6 | 7 | 8 |
| MULTD F0, F2, F4 | 9 | 10 | 20 | 21 |
| SUBD F8, F6, F2 | 10 | 22 | 24 | 25 |
| DIVD F10, F0, F6 | 11 | 26 | 66 | 67 |
| ADDD F6, F8, F2 | 26 | 27 | 29 | 30 |

**A execução completa levou 67 ciclos de clock (Clock Cycle No. 67).**

### 3.2 Análise Detalhada dos Hazards

#### 3.2.1 Hazard Estrutural: LD F2 espera unidade Integer

**LD F2, 45+R3** precisa da unidade Integer, que está ocupada pelo **LD F6, 34+R2**. Como o simulador emite em ordem (in-order issue) e só há 1 unidade Integer, o segundo LD só pode ser emitido no ciclo 5, após o primeiro LD completar o Write Result no ciclo 4. **Atraso estrutural: 4 ciclos (ciclo 1 a 5).**

#### 3.2.2 Hazard RAW: MULTD espera F2 do LD

**MULTD F0, F2, F4** precisa ler F2, que é o destino de **LD F2, 45+R3**. O LD F2 escreve F2 no ciclo 8. O MULTD, emitido no ciclo 9, consegue ler operandos no ciclo 10 (após F2 estar disponível). **Atraso RAW: 1 ciclo no Read Operands.**

#### 3.2.3 Hazard RAW: SUBD espera F2

**SUBD F8, F6, F2** também depende de F2 (do LD). Emitido no ciclo 10, mas precisa esperar no Read Operands. O SUBD só lê operandos no ciclo 22, pois também aguarda a disponibilidade da unidade Add (que pode estar envolvida em dependências). F6 já está disponível desde o ciclo 4, e F2 desde o ciclo 8. **O atraso no Read Operands do SUBD (ciclo 10 a 22) é causado pela espera da resolução de dependências encadeadas.**

#### 3.2.4 Hazard RAW: DIVD espera F0 do MULTD

**DIVD F10, F0, F6** precisa ler F0, que é o destino de MULTD. O MULTD só escreve F0 no ciclo 21. O DIVD, emitido no ciclo 11, fica esperando no Read Operands até o ciclo 26, pois F0 só fica disponível após o ciclo 21. **Atraso RAW de 15 ciclos, causado pela longa latência da multiplicação.**

#### 3.2.5 Atraso por WAR: ADDD não pode escrever F6

**ADDD F6, F8, F2** escreve em F6. Porém, **DIVD F10, F0, F6** (emitida antes) precisa ler F6 como operando fonte. O DIVD leu operandos no ciclo 26. O ADDD completa a execução no ciclo 29 e escreve no ciclo 30, pois nesse ponto o DIVD já leu F6 (ciclo 26). **O atraso WAR é visível: o ADDD só foi emitido no ciclo 26 (após a unidade Add ficar livre e DIVD já ter lido F6).**

*Este é o exemplo clássico de WAR no Scoreboard: uma instrução mais recente não pode escrever em um registrador enquanto uma instrução mais antiga ainda não o leu.*

---

## 4. Quadro Resumo de Hazards Identificados

| Tipo de Hazard | Estágio Afetado | Causa | Exemplo no Programa |
|---|---|---|---|
| **RAW** | Read Operands | Operando fonte ainda não escrito | MULTD espera F2 do LD (ciclos 9→10) |
| **WAR** | Write Result | Instrução anterior não leu registrador | ADDD espera DIVD ler F6 (ciclo 26→30) |
| **WAW** | Issue | Mesmo registrador destino | Discussão teórica: MULTD/ADDD com dest F0 |
| **Estrutural** | Issue | Unidade funcional ocupada | LD F2 espera Integer livre (ciclos 1→5) |

---

## 5. Discussão: Hazard WAW no Scoreboard

O hazard WAW ocorre quando duas instruções escrevem no mesmo registrador destino. No programa analisado, não há WAW explícito. Porém, podemos construir um exemplo teórico:

```
1. MULTD F0, F2, F4   ; escreve F0, 10 ciclos
2. ADDD F0, F6, F8    ; escreve F0, 2 ciclos
3. SUBD F2, F0, F6    ; lê F0
```

Neste caso, tanto MULTD quanto ADDD escrevem em F0. O Scoreboard bloqueia a emissão do ADDD até que MULTD escreva F0:

| Instrução | Issue | Read Op | Exec | Write Result |
|---|---|---|---|---|
| MULTD F0, F2, F4 | 1 | 2 | 12 | 13 |
| ADDD F0, F6, F8 | 14 | 15 | 17 | 18 |
| SUBD F2, F0, F6 | 15 | 19 | 21 | 22 |

O ADDD só é emitido no ciclo 14, após MULTD escrever F0 no ciclo 13. Isso causa um atraso significativo de 13 ciclos no Issue.

---

## 6. Solução: Renomeação de Registradores

A renomeação de registradores elimina WAW e WAR atribuindo registradores físicos temporários, removendo falsas dependências.

### 6.1 Eliminação do WAW

| Original | Com Renomeação | Efeito |
|---|---|---|
| MULTD F0, F2, F4 | MULTD F0, F2, F4 | Sem alteração |
| ADDD F0, F6, F8 | ADDD T1, F6, F8 | Elimina WAW: T1 ≠ F0 |
| SUBD F2, F0, F6 | SUBD F2, T1, F6 | Lê T1 (resultado correto do ADDD) |

Com renomeação, o ADDD escreve em T1 em vez de F0. Não há conflito com MULTD, e o ADDD pode ser emitido imediatamente.

### 6.2 Eliminação do WAR

No programa principal, o ADDD não podia escrever F6 enquanto DIVD não lesse F6. Com renomeação:

- Original: `ADDD F6, F8, F2` — WAR com DIVD que lê F6
- Renomeado: `ADDD T2, F8, F2` — sem conflito com F6

O ADDD escreveria em T2, eliminando o atraso WAR completamente.

### 6.3 Impacto

A renomeação é a base do algoritmo de Tomasulo, que estende o Scoreboard eliminando WAR e WAW via reservation stations. Processadores modernos (x86, ARM) usam renomeação por hardware com centenas de registradores físicos.

---

## 7. Conclusão

A análise do programa no simulador de Scoreboard demonstrou os quatro tipos de hazards:

- **RAW:** dependência verdadeira mais frequente, resolvida no Read Operands. Exemplos: MULTD espera F2, DIVD espera F0 (15 ciclos de atraso pela latência do MULTD).

- **WAR:** anti-dependência que atrasa a escrita. ADDD não pode escrever F6 enquanto DIVD não ler F6.

- **WAW:** dependência de saída demonstrada teoricamente: MULTD/ADDD com mesmo destino F0 bloqueia Issue por 13 ciclos.

- **Estrutural:** LD F2 aguarda 4 ciclos pela unidade Integer ocupada pelo LD F6.

A renomeação de registradores foi apresentada como solução para eliminar WAW e WAR, base do algoritmo de Tomasulo e de processadores superescalares modernos.

---

## Referências Bibliográficas

- STALLINGS, William. *Arquitetura e organização de computadores.* 8. ed. São Paulo: Pearson, 2010.
- HENNESSY, John L.; PATTERSON, David A. *Arquitetura de computadores: uma abordagem quantitativa.* 4. ed. Rio de Janeiro: Campus, 2009.
- KOREN, Israel. Scoreboard Simulator. Disponível em: http://www.ecs.umass.edu/ece/koren/architecture/scoreboard/. Acesso em: abril de 2026.

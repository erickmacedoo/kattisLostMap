# Marco 1 - Formulação (H - Kattis Lost Map)

**Problema:** Kattis - Lost Map (H) |
**Classe:** árvore geradora mínima | 
**Status:** concluído

## 1. Problema em linguagem própria

Existem `n` vilarejos ligados por estradas de mão dupla. Só foram construídas as estradas necessárias para todo vilarejo alcançar todos os outros, sem nenhuma sobrando. Existe uma tabela com a distância entre cada par de vilarejos, mas o mapa das estradas foi perdido. A tarefa é **reconstruir o mapa só a partir da tabela**, dizendo quais estradas existem.

## 2. Entrada, saída, objetivo e restrições

- **Entrada:** `n` (2 ≤ n ≤ 2.500) e uma matriz `n × n`, em que a posição `(i, j)` é a distância entre os vilarejos `i` e `j`. Um único caso por entrada.
- **Saída:** `n - 1` linhas, cada uma com um par `u v` (vilarejos de 1 a n) indicando uma estrada.
- **Objetivo:** recuperar exatamente o conjunto original de estradas.
- **Restrições:**
  - diagonal igual a 0; demais distâncias maiores que 0 e menores que 10⁷;
  - matriz simétrica;
  - rede original com `n - 1` estradas (conexa e sem ciclos);
  - a matriz chega a 6,25 milhões de números, então a leitura precisa ser rápida.

## 3. Vértices, arestas, direção e pesos

Sobre a tabela, montamos um **grafo completo**:
 
- **Vértices:** os `n` vilarejos;
- **Arestas:** todos os pares `(i, j)`, como candidatas a estrada (`n(n-1)/2` no total);
- **Direção:** não direcionado (estrada de mão dupla);
- **Peso:** a distância `d[i][j]` da tabela;
- **Capacidades:** não se aplicam (não há fluxo).
**Por que isso é uma árvore geradora mínima:** a rede original é uma árvore. Para dois vilarejos que **não** são ligados por estrada, o caminho entre eles na árvore passa por 2 ou mais estradas, todas com comprimento positivo. Logo, a distância `d(u, v)` é a soma desses comprimentos, e fica **estritamente maior** que cada estrada do caminho. Ou seja, a aresta `(u, v)` é a mais pesada do ciclo que ela formaria com a árvore, e nunca entra na árvore mínima. Já as estradas reais têm `d(u, v)` igual ao próprio comprimento. Conclusão: a árvore geradora mínima do grafo completo é **única** e é exatamente a rede original, então nenhum desempate muda o resultado.

## 4. Instância pequena

5 vilarejos. Estradas reais (comprimento): 1-2 (3), 1-3 (2), 3-4 (4), 3-5 (1).

**Como a tabela nasce da árvore** (soma dos comprimentos no caminho):

| Par | Caminho | Distância |
|---|---|---|
| 1–2 | 1-2 | 3 |
| 1–3 | 1-3 | 2 |
| 1–4 | 1-3-4 | 2 + 4 = 6 |
| 1–5 | 1-3-5 | 2 + 1 = 3 |
| 2–3 | 2-1-3 | 3 + 2 = 5 |
| 2–4 | 2-1-3-4 | 3 + 2 + 4 = 9 |
| 2–5 | 2-1-3-5 | 3 + 2 + 1 = 6 |
| 3–4 | 3-4 | 4 |
| 3–5 | 3-5 | 1 |
| 4–5 | 4-3-5 | 4 + 1 = 5 |
 
**Entrada:**
```
5
0 3 2 6 3
3 0 5 9 6
2 5 0 4 1
6 9 4 0 5
3 6 1 5 0
```
 
**Saída esperada**:
```
1 2
1 3
3 4
3 5
```

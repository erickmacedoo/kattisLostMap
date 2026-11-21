# Marco 3 - Rastreamento Manual

**Problema:** Kattis - Lost Map (H) |
**Algoritmo:** Prim (`PrimMST` do algs4, com a adaptação para matriz) |
**Instância:** a pequena, de 5 vilarejos |
**Início:** vilarejo 1 (índice 0 no código)

## 1. Estruturas de dados e estados iniciais (algs4)

| Estrutura | Papel | Estado inicial (n = 5) |
|---|---|---|
| `marked[]` (`boolean`) | `true` quando o vértice já está na árvore | todos `false` |
| `distTo[]` (`double`) | menor peso de uma aresta que liga o vértice à árvore | todos `+∞` |
| `edgeTo[]` (`Edge`) | aresta que liga o vértice à árvore (na adaptação, guarda só o pai) | todos vazios |
| `pq` (`IndexMinPQ<Double>`) | vértices fora da árvore com ligação conhecida, com chave = `distTo` | vazia |
| `graph.adj(v)` (`EdgeWeightedGraph`) | arestas de cada vértice | na nossa adaptação, vira a matriz `m[v][w]` |
 
Ao entrar em `prim(graph, s)`, com `s` = vilarejo 1: `distTo[1] = 0` e `pq.insert(1, 0)`. Estado: `distTo = [0, ∞, ∞, ∞, ∞]`, `pq = {1:0}`.
 
*Na versão adaptada, `pq` vira "os vértices não marcados com `distTo` finito", e `delMin` vira "escolher o não marcado de menor `distTo`". As decisões são as mesmas, então este rastreamento vale para as duas versões.*

## 2. Rastreamento passo a passo

Notação: `pq` mostra `vértice:chave`; `edgeTo` mostra o vértice da árvore na outra ponta da aresta (o pai). Vilarejos de 1 a 5 (no código, índice = vilarejo - 1).

| Passo | Operação | `marked` | `distTo` [1..5] | `edgeTo` [1..5] | `pq` |
|---|---|---|---|---|---|
| 0 | início, `prim(1)`: `distTo[1]=0`, `insert(1,0)` | nenhum | [0, ∞, ∞, ∞, ∞] | [-, -, -, -, -] | {1:0} |
| 1 | `delMin` → **1**; `scan(1)`: 1–2 (3), 1–3 (2), 1–4 (6), 1–5 (3), todas melhoram | {1} | [0, 3, 2, 6, 3] | [–, 1, 1, 1, 1] | {3:2, 2:3, 5:3, 4:6} |
| 2 | `delMin` → **3**; `scan(3)`: 3–1 obsoleta; 3–2 (5 não < 3); 3–4 (4 < 6, melhora); 3–5 (1 < 3, melhora) | {1, 3} | [0, 3, 2, 4, 1] | [–, 1, 1, 3, 3] | {5:1, 2:3, 4:4} |
| 3 | `delMin` → **5**; `scan(5)`: 5–1 e 5–3 obsoletas; 5–2 (6 não < 3); 5–4 (5 não < 4) | {1, 3, 5} | [0, 3, 2, 4, 1] | [–, 1, 1, 3, 3] | {2:3, 4:4} |
| 4 | `delMin` → **2**; `scan(2)`: 2–1, 2–3 e 2–5 obsoletas; 2–4 (9 não < 4) | {1, 2, 3, 5} | [0, 3, 2, 4, 1] | [–, 1, 1, 3, 3] | {4:4} |
| 5 | `delMin` → **4**; `scan(4)`: todos já marcados | {1, 2, 3, 4, 5} | [0, 3, 2, 4, 1] | [–, 1, 1, 3, 3] | {} |
 
`pq` vazia: o `while` termina.
 
**Resultado:** `edgeTo` = 2←1, 3←1, 4←3, 5←3. Estradas: **1–2, 1–3, 3–4, 3–5**, peso total 3 + 2 + 4 + 1 = **10**. É a saída esperada, na mesma ordem em que `edges()` as devolve (por índice do vértice).

## 3. Justificativa das decisões e do resultado

**Por que cada `delMin` escolhe o vértice certo:** a cada passo existe um corte entre quem está na árvore e quem está fora. A aresta mais leve que cruza esse corte sempre pertence à árvore mínima, e o `distTo` mantém exatamente essa aresta para cada vértice de fora.

| Passo | Árvore | Fora | Arestas que cruzam o corte | Mais leve | Entra |
|---|---|---|---|---|---|
| 1 | {1} | {2, 3, 4, 5} | 1–2 (3), 1–3 (2), 1–4 (6), 1–5 (3) | **1–3 (2)** | 3 |
| 2 | {1, 3} | {2, 4, 5} | 1–2 (3), 1–4 (6), 1–5 (3), 3–2 (5), 3–4 (4), 3–5 (1) | **3–5 (1)** | 5 |
| 3 | {1, 3, 5} | {2, 4} | 1–2 (3), 1–4 (6), 3–2 (5), 3–4 (4), 5–2 (6), 5–4 (5) | **1–2 (3)** | 2 |
| 4 | {1, 2, 3, 5} | {4} | 1–4 (6), 2–4 (9), 3–4 (4), 5–4 (5) | **3–4 (4)** | 4 |

**Por que a aresta 1–5 (peso 3) não entrou:** no passo 1, o vilarejo 5 era alcançado por 1–5 (3). No passo 2, o `scan(3)` achou 3–5 (1), que é mais leve, então `distTo[5]` caiu de 3 para 1 e `edgeTo[5]` passou de 1 para 3. Isso é o `decreaseKey`.
 
**O que é aresta obsoleta:** quando `marked[w]` já é `true`, a aresta liga dois vértices que já estão na árvore. Aceitá-la fecharia um ciclo, então o `scan` a ignora (`continue`).
 
**Por que o resultado está certo:** são `n − 1 = 4` arestas, formam uma árvore, e o peso 10 bate com a rede original do Marco 1 (3 + 2 + 4 + 1).

## 4. Etapa explicada individualmente (código algs4)

Trechos do `PrimMST.java` do repositório algs4.java, uma etapa por integrante.

### Etapa A — Inicialização (construtor)

```java
for (int v = 0; v < graph.V(); v++)
    distTo[v] = Double.POSITIVE_INFINITY;
 
for (int v = 0; v < graph.V(); v++)
    if (!marked[v]) prim(graph, v);
```

- Toda distância começa em infinito: "ainda não sei como chegar nesse vértice".
- O segundo laço roda o Prim a partir de cada vértice ainda não marcado. Como o nosso grafo é conexo, ele roda uma vez só, a partir do vértice 0 (vilarejo 1).
- **Na nossa adaptação:** `distTo`, `edgeTo` e `marked` continuam iguais; só não há `pq` para criar.

### Etapa B — Laço principal (`prim`)
 
```java
private void prim(EdgeWeightedGraph graph, int s) {
    distTo[s] = 0.0;
    pq.insert(s, distTo[s]);
    while (!pq.isEmpty()) {
        int v = pq.delMin();
        scan(graph, v);
    }
}
```
 
- O vértice inicial recebe distância 0 e entra na fila.
- A cada volta, `delMin()` devolve o vértice de menor chave, que é o próximo a entrar na árvore, e `scan` o processa.
- O laço termina quando a fila esvazia, ou seja, quando não há mais vértices alcançáveis.
- **Na nossa adaptação:** o `delMin()` vira uma varredura linear em `distTo[]`, escolhendo o não marcado de menor valor. É isso que faz o custo ser `O(V²)`.

### Etapa C — Processar um vértice (`scan`)
 
```java
private void scan(EdgeWeightedGraph graph, int v) {
    marked[v] = true;
    for (Edge e : graph.adj(v)) {
        int w = e.other(v);
        if (marked[w]) continue;
        if (e.weight() < distTo[w]) {
            distTo[w] = e.weight();
            edgeTo[w] = e;
            if (pq.contains(w)) pq.decreaseKey(w, distTo[w]);
            else                pq.insert(w, distTo[w]);
        }
    }
}
```
 
- Marca `v` como parte da árvore.
- Para cada vizinho `w` ainda fora da árvore, vê se a ligação `v–w` é mais barata do que a melhor conhecida até agora. Se for, atualiza `distTo[w]` e `edgeTo[w]`.
- `decreaseKey` ou `insert` apenas mantêm a fila em dia com o novo valor.
- **Na nossa adaptação:** `graph.adj(v)` vira um laço `w` de 0 a `n − 1` lendo `m[v][w]`. Como `v` já está marcado, o próprio `v` é pulado pelo `if (marked[w])`.

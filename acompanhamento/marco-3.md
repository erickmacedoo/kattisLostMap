# Marco 3 - Rastreamento Manual

**Problema:** Kattis - Lost Map (H) |
**Algoritmo:** Prim (`PrimMST` do algs4, sem alterações) |
**Instância:** a pequena, de 5 vilarejos |
**Início:** vilarejo 1 (índice 0 no código)

## 1. Estruturas de dados e estados iniciais (algs4)

| Estrutura | Papel | Estado inicial (n = 5) |
|---|---|---|
| `EdgeWeightedGraph` (`graph.adj(v)`) | listas de adjacência, com um `Edge` por aresta | `V = 5`, `E = 10`, listas montadas pelo `Main` (abaixo) |
| `marked[]` (`boolean`) | `true` quando o vértice já está na árvore | todos `false` |
| `distTo[]` (`double`) | menor peso de uma aresta que liga o vértice à árvore | todos `+∞` |
| `edgeTo[]` (`Edge`) | essa aresta, ou seja, com quem o vértice se liga | todos vazios |
| `pq` (`IndexMinPQ<Double>`) | vértices fora da árvore com ligação conhecida, chave = `distTo` | vazia |
 
Listas de adjacência de partida (formato `vizinho(peso)`, mais recente primeiro, que é a ordem do `Bag`):
 
```
adj[1] = 5(3) 4(6) 3(2) 2(3)
adj[2] = 5(6) 4(9) 3(5) 1(3)
adj[3] = 5(1) 4(4) 2(5) 1(2)
adj[4] = 5(5) 3(4) 2(9) 1(6)
adj[5] = 4(5) 3(1) 2(6) 1(3)
```

Ao entrar em `prim(graph, s)`, com `s` = vilarejo 1: `distTo[1] = 0` e `pq.insert(1, 0)`. Estado: `distTo = [0, ∞, ∞, ∞, ∞]`, `pq = {1:0}`.

## 2. Rastreamento passo a passo

Notação: `pq` mostra `vértice:chave`; `edgeTo` mostra o vértice da árvore na outra ponta da aresta (o pai). Vilarejos de 1 a 5 (no código, índice = vilarejo − 1). Dentro de cada `scan`, as arestas são examinadas na ordem da lista de adjacência.
 
| Passo | Operação | `marked` | `distTo` [1..5] | `edgeTo` [1..5] | `pq` |
|---|---|---|---|---|---|
| 0 | início, `prim(1)`: `distTo[1]=0`, `insert(1,0)` | nenhum | [0, ∞, ∞, ∞, ∞] | [–, –, –, –, –] | {1:0} |
| 1 | `delMin` → **1**; `scan(1)`: 5(3), 4(6), 3(2), 2(3), todas menores que ∞ e fora da fila: 4 × `insert` | {1} | [0, 3, 2, 6, 3] | [–, 1, 1, 1, 1] | {3:2, 2:3, 5:3, 4:6} |
| 2 | `delMin` → **3**; `scan(3)`: 5(1) < 3, `decreaseKey`; 4(4) < 6, `decreaseKey`; 2(5) não < 3; 1 já marcado | {1, 3} | [0, 3, 2, 4, 1] | [–, 1, 1, 3, 3] | {5:1, 2:3, 4:4} |
| 3 | `delMin` → **5**; `scan(5)`: 4(5) não < 4; 3 marcado; 2(6) não < 3; 1 marcado | {1, 3, 5} | [0, 3, 2, 4, 1] | [–, 1, 1, 3, 3] | {2:3, 4:4} |
| 4 | `delMin` → **2**; `scan(2)`: 5 marcado; 4(9) não < 4; 3 e 1 marcados | {1, 2, 3, 5} | [0, 3, 2, 4, 1] | [–, 1, 1, 3, 3] | {4:4} |
| 5 | `delMin` → **4**; `scan(4)`: 5, 3, 2 e 1 já marcados | {1, 2, 3, 4, 5} | [0, 3, 2, 4, 1] | [–, 1, 1, 3, 3] | {} |
 
`pq` vazia: o `while` de `prim` termina. Como o grafo é conexo, o laço do construtor não inicia outra busca.
 
**Resultado:** `edgeTo` = 2←1, 3←1, 4←3, 5←3. Estradas: **1–2, 1–3, 3–4, 3–5**, peso total 3 + 2 + 4 + 1 = **10**. É a saída esperada.

## 3. Justificativa das decisões e do resultado

**Por que cada `delMin` escolhe o vértice certo:** a cada passo existe um corte entre quem está na árvore e quem está fora. A aresta mais leve que cruza esse corte sempre pertence à árvore mínima, e o `distTo` mantém exatamente essa aresta para cada vértice de fora.
 
| Passo | Árvore | Fora | Arestas que cruzam o corte | Mais leve | Entra |
|---|---|---|---|---|---|
| 1 | {1} | {2, 3, 4, 5} | 1–2 (3), 1–3 (2), 1–4 (6), 1–5 (3) | **1–3 (2)** | 3 |
| 2 | {1, 3} | {2, 4, 5} | 1–2 (3), 1–4 (6), 1–5 (3), 3–2 (5), 3–4 (4), 3–5 (1) | **3–5 (1)** | 5 |
| 3 | {1, 3, 5} | {2, 4} | 1–2 (3), 1–4 (6), 3–2 (5), 3–4 (4), 5–2 (6), 5–4 (5) | **1–2 (3)** | 2 |
| 4 | {1, 2, 3, 5} | {4} | 1–4 (6), 2–4 (9), 3–4 (4), 5–4 (5) | **3–4 (4)** | 4 |
 
**Por que a aresta 1–5 (peso 3) não entrou:** no passo 1, o vilarejo 5 era alcançado por 1–5 (3). No passo 2, o `scan(3)` achou 3–5 (1), que é mais leve, então `distTo[5]` caiu de 3 para 1 e `edgeTo[5]` passou de 1 para 3. Isso é o `decreaseKey`.
 
**`insert` ou `decreaseKey`:** se o vértice ainda não está na fila (`pq.contains(w)` é falso), ele é inserido; se já está e a nova ligação é melhor, a chave dele é diminuída.
 
**O que é aresta obsoleta:** quando `marked[w]` já é `true`, a aresta liga dois vértices que já estão na árvore. Aceitá-la fecharia um ciclo, então o `scan` a ignora (`continue`).
 
**Por que o resultado está certo:** são `n − 1 = 4` arestas, formam uma árvore, e o peso 10 bate com a rede original do Marco 1 (3 + 2 + 4 + 1).

## 4. Etapa explicada individualmente (código algs4)

Trechos do `PrimMST.java` do repositório algs4.

### Etapa A — Inicialização (construtor)
 
```java
for (int v = 0; v < graph.V(); v++)
    distTo[v] = Double.POSITIVE_INFINITY;
 
for (int v = 0; v < graph.V(); v++)
    if (!marked[v]) prim(graph, v);
```
 
- Toda distância começa em infinito: "ainda não sei como chegar nesse vértice".
- O segundo laço roda o Prim a partir de cada vértice ainda não marcado. Como o nosso grafo é conexo, ele roda uma vez só, a partir do vértice 0 (vilarejo 1).

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
- `decreaseKey` ou `insert` mantêm a fila em dia com o novo valor.

## 5. Onde entra a nossa entrada
 
O único código nosso é o `Main`, que prepara o grafo antes de chamar o `PrimMST`:
 
```java
EdgeWeightedGraph graph = new EdgeWeightedGraph(n);
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++) {
        in.nextToken();
        if (i < j) graph.addEdge(new Edge(i, j, in.nval));
    }
PrimMST mst = new PrimMST(graph);
```
 
Cada número da matriz é lido; os pares com `i < j` viram uma aresta, e o `PrimMST` roda exatamente como no algs4.


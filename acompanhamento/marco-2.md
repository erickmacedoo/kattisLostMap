# Marco 2 - Escolha do algoritmo

**Problema:** Kattis - Lost Map (H) | 
**Classe:** árvore geradora mínima |
**Algoritmo escolhido:** Prim, com a classe `PrimMST` do algs4 **sem alterações** (com adaptações na forma da entrada)

## 1. Prim ou Kruskal?

Dados do problema: `n ≤ 2.500` e o grafo é **completo**, ou seja, `E = n(n−1)/2 = 3.123.750` arestas. A entrada vem como matriz, e o `Main` a transforma no grafo que as classes do algs4 esperam.

| | Prim (`PrimMST`) | Kruskal (`KruskalMST`) |
|---|---|---|
| Ideia | Cresce a árvore a partir de um vértice, sempre pegando a ligação mais barata entre quem está dentro e quem está fora | Ordena todas as arestas e aceita da mais leve para a mais pesada, se não fechar ciclo |
| Estruturas | `marked[]`, `distTo[]`, `edgeTo[]` e fila `IndexMinPQ` | vetor com todas as arestas ordenado e `UF` (union-find) |
| Tempo (doc. do algs4) | `Θ(E log V)` | `Θ(E log E)` |
| Espaço extra (doc. do algs4) | `Θ(V)` | `Θ(E)` |
| Tempo medido, `n = 2.500` (aleatória / caminho / estrela) | 1,8–3,0 s / 2,8–5,1 s / 1,0–1,1 s | 5,2–5,9 s / 3,8–4,2 s / 1,6–1,7 s |
| Menor heap que funcionou (rede aleatória) | 256 MB | 384 MB (falhou com 320 MB) |


1. **Na teoria, os dois ficam na mesma ordem de tempo.** Com `E ≈ V²/2`, temos `log E ≈ 2 log V`, então ambos são `O(V² log V)`. A diferença está nos detalhes.
2. **O Prim usa bem menos memória extra.** A fila dele tem no máximo `V` itens; o Kruskal precisa de um vetor com as 3,1 milhões de arestas para ordenar. Nos testes, o Prim funcionou com 256 MB e o Kruskal falhou com 320 MB.
3. **Na prática, o Prim foi mais rápido ou igual** nos três formatos de rede testados.
4. **O rastreamento manual é direto:** a cada passo, basta olhar o corte entre a árvore e o resto e pegar a aresta mais leve.
5. **A saída é a mesma nos dois:** a árvore mínima é única (Marco 1), então a escolha é só de eficiência e simplicidade.

**Decisão:** usar o `PrimMST` do algs4 **sem nenhuma alteração**. A única diferença é a entrada: o `Main` lê a matriz, cria um `Edge(i, j, distância)` para cada par `i < j` e entrega o `EdgeWeightedGraph` ao `PrimMST`. O `i < j` evita duplicar as arestas, pois a matriz é simétrica.

## 2. Instância pequena representada

Matriz lida da entrada (no código, índice = vilarejo − 1):
 
```
        1  2  3  4  5
   1    0  3  2  6  3
   2    3  0  5  9  6
   3    2  5  0  4  1
   4    6  9  4  0  5
   5    3  6  1  5  0
```
 
Grafo completo (10 arestas candidatas):
 
| Aresta | Peso | Estrada original? |
|---|---|---|
| 1–2 | 3 | sim |
| 1–3 | 2 | sim |
| 1–4 | 6 | não |
| 1–5 | 3 | não |
| 2–3 | 5 | não |
| 2–4 | 9 | não |
| 2–5 | 6 | não |
| 3–4 | 4 | sim |
| 3–5 | 1 | sim |
| 4–5 | 5 | não |

Como o grafo fica no `EdgeWeightedGraph` (`V = 5`, `E = 10`). Cada vizinho aparece como `vizinho(peso)`, e a lista mostra o mais recente primeiro, que é a ordem do `Bag`:
 
```
adj[1] = 5(3) 4(6) 3(2) 2(3)
adj[2] = 5(6) 4(9) 3(5) 1(3)
adj[3] = 5(1) 4(4) 2(5) 1(2)
adj[4] = 5(5) 3(4) 2(9) 1(6)
adj[5] = 4(5) 3(1) 2(6) 1(3)
```

## 3. Validação de entrada e saída esperada

**Entrada válida quando:**
- `2 ≤ n ≤ 2.500`;
- diagonal igual a 0;
- fora da diagonal, `0 < d < 10⁷`;
- `d[i][j] = d[j][i]`.

**Saída válida quando:**
- tem `n − 1` linhas `u v`, com vilarejos de 1 a n;
- as arestas formam uma árvore (conexa e sem ciclos);
- **teste de consistência:** recalculando as distâncias entre todos os pares dentro dessa árvore, obtém-se de volta a matriz de entrada.

**Formato adotado:** `u < v`, em ordem crescente, como nos exemplos do enunciado.

**Cuidado prático:** a leitura de até 6,25 milhões de números deve usar entrada rápida (`BufferedReader`/`StreamTokenizer`), pois `Scanner` tende a ser lento demais.

**Conferência** (feita com script auxiliar, que não faz parte da solução):

| Caso | n | Entrada válida | Distâncias da saída reproduzem a matriz | `PrimMST` devolve a saída esperada | Peso total |
|---|---|---|---|---|---|
| Instância pequena | 5 | sim | sim | sim | 10 |
| Exemplo 1 do enunciado | 4 | sim | sim | sim | 4 |
| Exemplo 2 do enunciado | 7 | sim | sim | sim | 15 |
 
Além disso, em 300 redes aleatórias (`n` de 2 a 40; formas aleatória, em caminho e em estrela), o programa devolveu sempre a rede original, sem nenhuma divergência.

## 4. Complexidade de pior caso

Seja `V = n` e `E = V(V−1)/2` (grafo completo, então o número de arestas é sempre o mesmo).

| | **`PrimMST` (escolhido)** | `KruskalMST` |
|---|---|---|
| Tempo | **`Θ(E log V) = O(V² log V)`** | `Θ(E log E) = O(V² log V)` |
| Espaço do grafo | `O(V + E) = O(V²)` | `O(V + E) = O(V²)` |
| Espaço extra do algoritmo | **`Θ(V)`** | `Θ(E) = Θ(V²)` |
| Para V = 2.500 | ~35 milhões de passos (`E · log V`) | ~67 milhões (`E · log E`) |

*(Os números da última linha são ordens de grandeza, não medições.)*

**Por que o Prim é `Θ(E log V)`:**
- cada vértice sai da fila uma única vez com `delMin`, que custa `log V`: no total, `V log V`;
- cada aresta é examinada ao processar suas duas pontas, e cada exame pode gerar um `insert` ou `decreaseKey`, que também custam `log V`: no total, no máximo `E log V`;
- a segunda parcela domina, então o total é `E log V`.

**Espaço:** o grafo guarda um objeto `Edge` por aresta e duas posições de lista (`Bag`) por aresta, ou seja, `O(V²)`. Já o que o `PrimMST` acrescenta são os vetores `marked[]`, `distTo[]`, `edgeTo[]` e as estruturas da fila de prioridade, todos de tamanho `V`: **`Θ(V)`**.
 
**Demais custos:** a leitura e a montagem do grafo custam `O(V²)`, e a saída (`n − 1` pares, ordenados) custa `O(V log V)`, ambas menores que o custo do algoritmo.

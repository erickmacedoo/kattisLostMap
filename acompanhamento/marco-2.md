# Marco 2 - Escolha do algoritmo

**Problema:** Kattis - Lost Map (H) | 
**Classe:** árvore geradora mínima |
**Algoritmo escolhido:** Prim (versão com matriz, adaptado do `PrimMST` do algs4) |

## 1. Prim ou Kruskal?

Dados do problema: `n ≤ 2.500` e o grafo é **completo**, ou seja, `E = n(n−1)/2 = 3.123.750` arestas. A entrada já é uma matriz com `n² = 6,25 milhões` de números.

| | Prim (`PrimMST`) | Kruskal (`KruskalMST`) |
|---|---|---|
| Ideia | Cresce a árvore a partir de um vértice, sempre pegando a ligação mais barata entre quem está dentro e quem está fora | Ordena todas as arestas e aceita da mais leve para a mais pesada, se não fechar ciclo |
| Estruturas | `marked[]`, `distTo[]`, `edgeTo[]` e fila de prioridade | lista de arestas ordenada, `UF` (union-find) |
| Tempo | `O(E log V)`; com matriz e varredura linear, `O(V²)` | `O(E log E)` |
| Combina com grafo | denso | esparso |

**Por que Prim:**

1. **O grafo é denso.** Com `E ≈ V²/2`, o Prim com matriz custa `O(V²)`, enquanto o Kruskal custa `O(V² log V)`.
2. **O Prim denso chega ao limite possível.** Só ler a entrada já custa `Ω(V²)`, então nenhum algoritmo é mais rápido que isso. O Prim com matriz é `O(V²)`; o Kruskal acrescenta o fator `log`.
3. **O Kruskal precisa criar e ordenar 3,1 milhões de arestas**, além do `UF`. O Prim usa a matriz que já foi lida e só 3 vetores de tamanho `n`.
4. **O resultado é o mesmo.** A árvore mínima é única, então a escolha é só de eficiência e simplicidade.

**Adaptação prevista do `PrimMST`:** manter `marked[]`, `distTo[]`, `edgeTo[]` e a lógica de `scan(v)`; trocar o `EdgeWeightedGraph` pela matriz `int[][]`; trocar o `IndexMinPQ` por uma varredura linear em `distTo[]` (escolher o vértice não marcado de menor distância); `edgeTo[]` passa a guardar o vértice pai, em vez de um objeto `Edge`.
 
**Plano B:** usar o `PrimMST` do algs4 sem alterações (lista + `IndexMinPQ`). Funciona, mas cria milhões de objetos e é mais pesado.

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

| Caso | n | Entrada válida | Distâncias da saída reproduzem a matriz | Prim com matriz devolve a saída esperada | Peso total |
|---|---|---|---|---|---|
| Instância pequena | 5 | sim | sim | sim | 10 |
| Exemplo 1 do enunciado | 4 | sim | sim | sim | 4 |
| Exemplo 2 do enunciado | 7 | sim | sim | sim | 15 |
 
Em 500 redes aleatórias (n de 2 a 12), o Prim com matriz devolveu sempre a rede original, sem nenhuma divergência.

## 4. Complexidade de pior caso

Seja `V = n` e `E = V(V−1)/2` (grafo completo, então o número de arestas é sempre o mesmo).

| | **Prim com matriz (escolhido)** | `PrimMST` do algs4 (lista + `IndexMinPQ`) | Kruskal |
|---|---|---|---|
| Tempo | **`O(V²)`** | `O(E log V) = O(V² log V)` | `O(E log E) = O(V² log V)` |
| Espaço total | **`O(V²)`** (matriz) | `O(V + E) = O(V²)` (objetos) | `O(E) = O(V²)` |
| Espaço auxiliar | **`O(V)`** | `O(V)` | `O(V)` (só o `UF`, sem contar a lista de arestas) |
| Para V = 2.500 | ~6,25 milhões de operações | ~35 milhões (`E · log V`) | ~67 milhões (`E · log E`) |

*(Os números da última linha são ordens de grandeza, não medições.)*

**Por que o Prim com matriz é `O(V²)`:**
- o laço principal roda `V` vezes (uma por vértice que entra na árvore);
- em cada volta, uma varredura `O(V)` escolhe o vértice não marcado de menor `distTo[]`, e outra varredura `O(V)` atualiza `distTo[]` e `edgeTo[]` com as ligações desse vértice;
- total: `V × 2V = O(V²)`, sem depender dos valores dos pesos, então o pior caso é igual ao caso comum.

**Espaço:** a matriz ocupa `V²` inteiros (~25 MB para V = 2.500), e já seria necessária só para guardar a entrada. O algoritmo acrescenta apenas `marked[]`, `distTo[]` e `edgeTo[]`, três vetores de tamanho `V`. Cabe com folga nos 1024 MB do problema.

**Demais custos:** a leitura é `O(V²)`, e a saída (`n - 1` pares, ordenados) é `O(V log V)`, ambas menores ou iguais ao custo do algoritmo.

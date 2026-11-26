# Problema H - Lost Map

Grupo H - Resolução de Problemas com Grafos (UNIFOR), Prof. Ricardo Carubbi.

## 1. Problema

O problema [**Lost Map** (Kattis)](https://open.kattis.com/problems/lostmap) descreve `n` vilarejos ligados por estradas de mão dupla. Só foram construídas as estradas necessárias para que todo vilarejo alcance todos os outros, ou seja, a rede é uma árvore.
O mapa das estradas foi perdido, mas existe uma tabela com a distância entre cada par de vilarejos (soma dos comprimentos das estradas do menor caminho).

O objetivo é reconstruir o mapa a partir da tabela.

- **Entrada:** `n` (2 ≤ n ≤ 2.500) e uma matriz `n × n` de distâncias. A diagonal é 0, as demais distâncias estão entre 1 e 10⁷ - 1, e a matriz é simétrica.
- **Saída:** `n - 1` linhas `u v`, cada uma indicando uma estrada entre os vilarejos `u` e `v` (numerados de 1 a n).

## 2. Integrantes

Erick Araújo Macedo - *2518801*

Gabriel Cajado Cavalcante - *2420382*

Henrique Varela Barbosa Mouta - *2420507*

## 3. Linguagem

Java.

## 4. Execução

É necessário ter o **JDK** instalado. A solução está em `src/Main.java`.

### Compilação
```bash
javac -sourcepath src -d out src/Main.java
```

### Execução
A entrada é lida pela entrada padrão e a saída vai para a saída padrão.

```bash
java -cp src Main < entrada.txt
```

No PowerShell (Windows):
```powershell
Get-Content entrada.txt | java -cp src Main
```

### Exemplo
```bash
printf '5\n0 3 2 6 3\n3 0 5 9 6\n2 5 0 4 1\n6 9 4 0 5\n3 6 1 5 0\n' | java -cp out Main
```

Saída:
```
1 2
1 3
3 4
3 5
```

## 5. Modelagem

A tabela é modelada como um **grafo completo, não direcionado e ponderado**:

- **Vértices:** os `n` vilarejos;
- **Arestas:** todos os pares `(i, j)`, como estradas candidatas (`n(n−1)/2` no total);
- **Peso:** a distância `d[i][j]` da tabela.

As estradas reais são exatamente as arestas da **árvore geradora mínima** desse grafo. Para dois vilarejos que não são ligados por uma estrada, o caminho entre eles passa por 2 ou mais estradas de comprimento positivo, então a distância é **estritamente maior** que cada estrada do caminho. Assim, essa aresta é sempre a mais pesada do ciclo que formaria com a árvore, e nunca entra. Já as estradas reais têm distância igual ao próprio comprimento. Por isso a árvore geradora mínima é **única** e é a rede original.

## 6. Algoritmo

Foi utilizado o algoritmo de **Prim**. Ele cresce a árvore a partir de um vértice, sempre pegando a ligação mais barata entre quem já está na árvore e quem ainda está fora. O estado é mantido em quatro estruturas:
 
- `marked[v]`: `true` quando `v` já está na árvore;
- `distTo[v]`: menor peso conhecido de uma aresta que liga `v` à árvore (começa em infinito);
- `edgeTo[v]`: essa aresta, ou seja, com quem `v` se liga;
- `pq`: fila de prioridade com os vértices de fora que já têm ligação conhecida, ordenados por `distTo`.

Passo a passo:
 
1. o vértice inicial recebe `distTo = 0` e entra na fila;
2. `delMin` tira da fila o vértice de menor `distTo`;
3. `scan` marca esse vértice e, para cada vizinho `w` ainda fora, se a aresta é menor que `distTo[w]`, atualiza `distTo[w]` e `edgeTo[w]` (e a fila);
4. repete até a fila esvaziar. Os `edgeTo` formam as `n − 1` estradas.

A escolha do menor `distTo` é correta porque a aresta mais leve que cruza o corte entre a árvore e o resto sempre pertence à árvore geradora mínima.
 
O rastreamento passo a passo em uma instância pequena está em [`acompanhamento/marco-3.md`](./acompanhamento/marco-3.md), e a escolha entre Prim e Kruskal em [`acompanhamento/marco-2.md`](./acompanhamento/marco-2.md).

## 7. Implementação de referência

Foi usada a classe **`PrimMST`** da biblioteca `algs4`, disponibilizada pelo professor, **diretamente, sem alterações**:

- [PrimMST.java - disponibilizado pelo repositório do professor](https://github.com/carubbi/RPG/blob/main/algs4-java/algs4/PrimMST.java)

Ela usa `EdgeWeightedGraph` (listas de adjacência de objetos `Edge`, em `Bag`) e `IndexMinPQ` (fila de prioridade indexada). As demais classes de `src/algs4/` (`In`, `StdIn`, `StdOut`, `StdRandom`, `Queue`, `Stack`, `UF`) são dependências que o código do algs4 referencia.

## 8. Alterações

**Nas classes do algs4: nenhuma.** Os 12 arquivos de `src/algs4/` são idênticos aos do repositório do algs4.
 
A única diferença é a **forma da entrada**. O `PrimMST` recebe um `EdgeWeightedGraph`, e o problema entrega uma matriz. Por isso o `Main.java` faz:
 
| Etapa | O que o `Main` faz |
|---|---|
| Leitura | Lê `n` e a matriz com `StreamTokenizer` (leitura rápida) |
| Montagem do grafo | Para cada par `i < j`, cria `new Edge(i, j, distância)` e chama `graph.addEdge(...)` |
| Algoritmo | `new PrimMST(graph)` |
| Saída | Para cada aresta de `mst.edges()`, imprime `menor+1  maior+1`, em ordem crescente |
 
O `i < j` evita duplicar: a matriz é simétrica, então só a metade de cima vira aresta. Os vilarejos de 1 a n ficam nos índices de 0 a n−1, e a saída soma 1.

## 9. Justificativas

- **Prim e não Kruskal:** no nosso grafo completo, os dois ficam na mesma ordem de tempo (`O(V² log V)`), mas o Prim guarda só `V` itens na fila, enquanto o Kruskal guarda e ordena as 3,1 milhões de arestas. Nos testes, o Prim funcionou com 256 MB de heap e o Kruskal falhou com 320 MB. O Prim também foi mais rápido ou igual nos três formatos de rede testados.
- **Não alterar o algs4:** a classe já é a implementação conhecida e testada do algoritmo. Mudar só a entrada reduz o risco de erro e deixa a explicação simples.
- **Entrada rápida:** a matriz chega a 6,25 milhões de números, então não se usa `Scanner`.
- **Saída com `u < v`, em ordem crescente:** mesmo formato dos exemplos do enunciado.
- **Empates não importam:** como a árvore geradora mínima é única (seção 5), qualquer desempate devolve a mesma saída.

## 10. Correção esperada

A saída está correta quando as `n - 1` linhas formam exatamente a rede original. Isso pode ser conferido de duas formas:

- **Teste de consistência:** recalculando as distâncias entre todos os pares dentro da árvore devolvida, obtém-se de volta a matriz de entrada;
- **Comparação direta:** em redes geradas pelo verificador, a saída é igual ao conjunto de estradas usado para gerar a matriz.

### Exemplos do enunciado e instância pequena

| Caso | Entrada | Saída esperada | Resultado |
|---|---|---|---|
| Exemplo 1 | `n = 4` | `1 2`, `1 3`, `1 4` | confere |
| Exemplo 2 | `n = 7` | `1 4`, `1 5`, `2 4`, `3 5`, `4 6`, `4 7` | confere |
| Instância pequena | `n = 5` (seção 4) | `1 2`, `1 3`, `3 4`, `3 5` | confere |

## 11. Complexidade

Seja `V = n` e `E = V(V−1)/2`.
 
- **Tempo:** `Θ(E log V)`, conforme a documentação do `PrimMST`. Cada vértice sai da fila uma vez (`V` operações de custo `log V`), e cada aresta é examinada, podendo causar uma atualização da fila (também `log V`). No nosso grafo completo, isso dá `O(V² log V)`. A leitura e a montagem do grafo custam `O(V²)`, menos que isso.
- **Espaço do grafo:** `O(V + E) = O(V²)`, pois o `EdgeWeightedGraph` guarda um objeto `Edge` por aresta e duas posições de lista por aresta.
- **Espaço auxiliar do algoritmo:** `Θ(V)`, para `marked[]`, `distTo[]`, `edgeTo[]` e a fila (que tem no máximo `V` itens).

Para `V = 2.500`, `E = 3.123.750` e `E · log₂ V` é da ordem de 35 milhões de passos.

## 12. Casos especiais

| Caso | Entrada | Saída | Observação |
|---|---|---|---|
| Menor rede (`n = 2`) | `2` / `0 5` / `5 0` | `1 2` | Uma única estrada |
| Rede em caminho | `4` / `0 1 3 6` / `1 0 2 5` / `3 2 0 3` / `6 5 3 0` | `1 2`, `2 3`, `3 4` | Maior profundidade da árvore |
| Rede em estrela | `4` / `0 1 1 1` / `1 0 2 2` / `1 2 0 2` / `1 2 2 0` | `1 2`, `1 3`, `1 4` | Maior grau; vários valores repetidos na matriz |
| Distâncias repetidas | exemplo anterior, e estrela com `n = 2.500` | saída correta | A árvore é única, então empates não mudam o resultado |
| Tamanho máximo (`n = 2.500`) | cerca de 6,25 milhões de números | `2.499` linhas | Ver tempos na seção 11 |
| Formato da entrada | espaços extras, linhas em branco, fim de linha do Windows | mesma saída | O leitor separa os números e ignora o espaçamento |
 
Distâncias de até 10⁷ − 1 são exatas em `double`, que é o tipo do peso no `Edge`. Entradas fora do enunciado (por exemplo, matriz que não vem de uma árvore) não são tratadas, pois o problema garante que não ocorrem.

## 13. Evidência do Accepted

> Inserir aqui o print da página de submissão do Kattis.

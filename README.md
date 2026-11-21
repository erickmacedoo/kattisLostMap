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

É necessário ter o **JDK** instalado. A solução está em `src/Main.java`, sem pacote e sem dependências externas.

### Compilação
```bash
javac src/Main.java
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
printf '5\n0 3 2 6 3\n3 0 5 9 6\n2 5 0 4 1\n6 9 4 0 5\n3 6 1 5 0\n' | java -cp src Main
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

Foi utilizado o algoritmo de **Prim**, em sua versão para grafos densos, com a matriz de distâncias.

O Prim cresce a árvore a partir de um vértice. Durante a execução:

1. `distTo[v]` guarda o menor peso conhecido de uma aresta que liga `v` à árvore (começa em infinito) e `edgeTo[v]` guarda o vértice da árvore na outra ponta;
2. `distTo[0] = 0`: o vilarejo 1 é o ponto de partida;
3. a cada volta, escolhe-se o vértice **fora da árvore** com menor `distTo` e ele entra na árvore (`marked[v] = true`);
4. para cada vértice `w` ainda fora, se `d[v][w] < distTo[w]`, atualiza-se `distTo[w]` e `edgeTo[w] = v`;
5. após `n` voltas, `edgeTo` contém as `n − 1` estradas.

A escolha do vértice de menor `distTo` é correta porque a aresta mais leve que cruza o corte entre a árvore e o resto sempre pertence à árvore geradora mínima.

O rastreamento passo a passo em uma instância pequena está em [`acompanhamento/marco-3.md`](./acompanhamento/marco-3.md).

## 7. Implementação de referência

Foi usada como referência a classe **`PrimMST`** da biblioteca `algs4.java`, disponibilizada pelo professor:

- [PrimMST.java - disponibilizado pelo repositório do professor](https://github.com/carubbi/RPG/blob/main/algs4-java/algs4/PrimMST.java)

Na referência, o grafo é um `EdgeWeightedGraph` (listas de adjacência de objetos `Edge`) e os vértices candidatos ficam em uma fila de prioridade `IndexMinPQ`. Os conceitos aproveitados foram:

- `marked[]`, `distTo[]` e `edgeTo[]`;
- o método `prim(s)`, que parte de um vértice inicial;
- o método `scan(v)`, que marca `v` e atualiza `distTo`/`edgeTo` dos vizinhos ainda fora da árvore, ignorando arestas obsoletas (ligam dois vértices já na árvore).

## 8. Alterações

| Referência (`PrimMST`) | Nossa adaptação |
|---|---|
| Entrada: `EdgeWeightedGraph` | Entrada: matriz `int[][] dist`, lida diretamente da entrada padrão |
| `IndexMinPQ` para escolher o próximo vértice | Método `closest()`: varredura linear em `distTo[]`, escolhendo o não marcado de menor valor |
| `edgeTo` é `Edge[]` | `edgeTo` é `int[]`, guarda o vértice pai |
| `distTo` é `double[]` | `distTo` é `int[]`, com `Integer.MAX_VALUE` como infinito |
| `prim` roda a partir de cada vértice não marcado (floresta) | Roda uma única vez a partir do vértice 0, pois a entrada garante rede conexa |
| `edges()` devolve uma `Queue<Edge>` | `edges()` devolve os pares `u v` (com `u < v`) já ordenados |
| Possui `weight()`, `check()` e tolerância de ponto flutuante | Removidos: não são necessários para a saída pedida |
| Classe em `package algs4`, com dependências | Classe copiada para `src/Main.java`, sem pacote e sem dependências |

Também foram escritos a **leitura rápida** (`StreamTokenizer` com `BufferedReader`) e a **conversão para a saída** (vértice = índice + 1).

A lógica central (`prim` e `scan`) segue a da referência: mesmos vetores, mesma regra de atualização `<`.

## 9. Justificativas

- **Prim e não Kruskal:** o grafo é completo (até 3.123.750 arestas) e a entrada já é uma matriz. O Prim com matriz custa `O(V²)`, enquanto o Kruskal precisaria criar e ordenar todas as arestas (`O(E log E)`).
- **Matriz e não lista de adjacência:** em um grafo completo, a lista não traz ganho, pois todo vértice é vizinho de todos, e criaria milhões de objetos. A matriz ocupa cerca de 25 MB.
- **Varredura linear no lugar da fila de prioridade:** em grafo denso, a fila faria até `E` atualizações de custo `log V`. A varredura custa `O(V)` por vértice, `O(V²)` no total.
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

Seja `V = n` e `E = V(V-1)/2`.

- **Tempo:** `O(V²)`. O laço principal roda `V` vezes e, em cada volta, duas varreduras de `O(V)` (escolher o vértice e atualizar os vizinhos). Não depende dos valores dos pesos, então o pior caso é igual ao caso comum. A leitura (`O(V²)`) e a ordenação da saída (`O(V log V)`) não alteram essa ordem.
- **Espaço total:** `O(V²)`, ocupado pela matriz, que já seria necessária para guardar a entrada.
- **Espaço auxiliar do algoritmo:** `O(V)`, para `marked[]`, `distTo[]` e `edgeTo[]`.

Medição com `n = 2.500` (entrada de cerca de 40 MB), em máquina de teste com OpenJDK 21: cerca de 1,1 s para rede aleatória e 1,3 s para rede em caminho, incluindo a partida da JVM. A execução também funcionou com limite de memória de 128 MB para o heap (`-Xmx128m`). Esses valores variam conforme a máquina e não substituem o resultado do juiz.

## 12. Casos especiais

| Caso | Entrada | Saída | Observação |
|---|---|---|---|
| Menor rede (`n = 2`) | `2` / `0 5` / `5 0` | `1 2` | Uma única estrada |
| Rede em caminho | `4` / `0 1 3 6` / `1 0 2 5` / `3 2 0 3` / `6 5 3 0` | `1 2`, `2 3`, `3 4` | Maior profundidade da árvore |
| Rede em estrela | `4` / `0 1 1 1` / `1 0 2 2` / `1 2 0 2` / `1 2 2 0` | `1 2`, `1 3`, `1 4` | Maior grau; vários valores repetidos na matriz |
| Distâncias repetidas | exemplo anterior | mesma saída | A árvore é única, então empates não mudam o resultado |
| Tamanho máximo (`n = 2.500`) | cerca de 6,25 milhões de números | `2.499` linhas | Leitura rápida e matriz de cerca de 25 MB |
| Formato da entrada | espaços extras, linhas em branco, fim de linha do Windows | mesma saída | O leitor separa os números e ignora o espaçamento |

Distâncias de até 10⁷ - 1 cabem em `int`, e `Integer.MAX_VALUE` nunca é igual a uma distância real, então serve como infinito. Entradas fora do enunciado (por exemplo, matriz que não vem de uma árvore) não são tratadas, pois o problema garante que não ocorrem.

## 13. Evidência do Accepted

> Inserir aqui o print da página de submissão do Kattis (status **Accepted**).

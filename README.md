# Função `buildStateGraphAnimated()`

## Objetivo da função
Essa função constrói um **grafo de estados possíveis** de um quebra-cabeça (tipo Klotski ou Sliding Blocks), onde:

- Cada **nó** representa uma configuração válida do tabuleiro (`state`).  
- Cada **aresta (link)** conecta dois estados que diferem por um único movimento de bloco.  
- O grafo é usado para **visualizar os estados e a solução em 3D**.

---

## Passo a passo

### Inicialização
```javascript
const initialState = getCurrentPuzzleState();
const visited = new Set();
const queue = [{ state: initialState, parent: null }];
const stateToNodeId = new Map();
let graphData = { nodes: [], links: [] };
```

- `initialState` → estado atual do puzzle.  
- `visited` → guarda os estados já visitados (para evitar ciclos).  
- `queue` → fila de estados a serem explorados; cada item guarda o **estado atual** e seu **pai** (para conectar o grafo).  
- `stateToNodeId` → mapeia a string de um estado para o `id` do nó no grafo.  
- `graphData` → estrutura que vai armazenar **nós (nodes)** e **arestas (links)**.  

---

### Loop principal
```javascript
while (queue.length > 0 && graphData.nodes.length < 1000) {
    const { state, parent } = queue.shift();
    const stateKey = stateToString(state);

    if (visited.has(stateKey)) continue;
    visited.add(stateKey);
}
```

- Remove o primeiro item da fila (`queue.shift()`).  
- Converte o estado para uma string única (`stateToString`) para verificar duplicatas.  
- Se já foi visitado, ignora.  
- Marca como visitado.  

✅ **Fila + verificação de visitado → caracteriza BFS (Breadth-First Search / Busca em Largura)**.

---

### Criação do nó no grafo
```javascript
const nodeId = graphData.nodes.length;
stateToNodeId.set(stateKey, nodeId);
const isSolution = isWinState(state);

const newNode = {
    id: nodeId,
    isSolution: isSolution,
    label: `Estado ${nodeId}`
};
graphData.nodes.push(newNode);
nodeStates.set(nodeId, state);
```

- Cada estado vira um **nó** com:  
  - `id` → número sequencial  
  - `isSolution` → flag se é estado final  
  - `label` → apenas para exibição  

---

### Conexão com o nó pai
```javascript
if (parent !== null) {
    graphData.links.push({ source: parent, target: nodeId });
}
```

- Se o estado atual veio de outro estado (`parent`), cria-se uma **aresta** ligando os nós.

---

### Atualização de UI e animação
```javascript
window.updateGraphData({ ...graphData });
document.getElementById('stateCount').textContent = graphData.nodes.length;
document.getElementById('edgeCount').textContent = graphData.links.length;
```

- Atualiza visualmente a contagem de **nós** e **arestas**.  
- Permite ver a geração do grafo **em tempo real** (animação).

---

### Gerar estados seguintes
```javascript
const nextStates = generateNextStates(state);
for (const next of nextStates) {
    let nextStateKey = stateToString(next);
    if (!visited.has(nextStateKey)) {
        queue.push({ state: next, parent: nodeId });
    } else {
        let nextNodeId = stateToNodeId.get(nextStateKey);
        graphData.links.push({ source: nodeId, target: nextNodeId });
    }
}
```

- `generateNextStates(state)` → retorna todos os estados possíveis a partir do atual.  
- Para cada próximo estado:  
  - Se **não visitado**, adiciona na fila (`queue`) para explorar depois.  
  - Se já visitado, apenas conecta a **aresta** no grafo (evita duplicação de nó).  

---

### Finalização
```javascript
finishGeneration(graphData);
```

- Depois que a fila fica vazia (ou atingiu limite de nós), chama a função para **finalizar o grafo**.

---

## Algoritmo utilizado

A função utiliza **BFS (Breadth-First Search / Busca em Largura)**:

- Porque usa uma **fila (`queue`)**.  
- Explora primeiro todos os estados possíveis a uma distância do inicial antes de ir mais fundo.  

Se fosse **DFS (Depth-First Search)**, ela usaria uma **pilha (stack)** ou **recursão**, explorando cada caminho até o fim antes de voltar.

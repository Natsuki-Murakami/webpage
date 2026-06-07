<script setup>
import { computed, ref } from 'vue'

const nodeIds = ['A', 'B', 'C', 'D', 'E', 'F', 'G']

const edgeKey = (u, v) => [u, v].sort().join('-')
const stepItems = [
  { id: 0, text: 'Step 0(初期化)' },
  { id: 1, text: 'Step 1(初期点設定)' },
  { id: 2, text: 'Step 2(探索開始)' },
  { id: 3, text: 'Step 3(終了判定)' },
  { id: 4, text: 'Step 4(後退)' },
  { id: 5, text: 'Step 5(未探索辺の選択)' },
  { id: 6, text: 'Step 6(探索済み判定)' },
  { id: 7, text: 'Step 7(未探索点発見)' },
  { id: 8, text: 'Step 8(前進)' },
]

const shuffle = (values) => {
  const array = [...values]
  for (let i = array.length - 1; i > 0; i -= 1) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
  return array
}

const compareNodeId = (left, right) => left.localeCompare(right)

const distance = (a, b) => Math.hypot(a.x - b.x, a.y - b.y)
const pointSegmentDistance = (point, a, b) => {
  const dx = b.x - a.x
  const dy = b.y - a.y
  const lengthSquared = dx * dx + dy * dy
  if (lengthSquared === 0) return distance(point, a)

  const t = Math.max(0, Math.min(1, ((point.x - a.x) * dx + (point.y - a.y) * dy) / lengthSquared))
  const projection = {
    x: a.x + t * dx,
    y: a.y + t * dy,
  }
  return distance(point, projection)
}

const ccw = (a, b, c) => (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x)

const segmentsIntersect = (a, b, c, d) => {
  const abC = ccw(a, b, c)
  const abD = ccw(a, b, d)
  const cdA = ccw(c, d, a)
  const cdB = ccw(c, d, b)
  return abC * abD < 0 && cdA * cdB < 0
}

const generateNodes = () => {
  for (let attempt = 0; attempt < 300; attempt += 1) {
    const points = []

    while (points.length < nodeIds.length) {
      const candidate = {
        x: 48 + Math.random() * 344,
        y: 40 + Math.random() * 170,
      }

      if (points.every((point) => distance(point, candidate) >= 72)) {
        points.push(candidate)
      }
    }

    const sorted = [...points].sort((left, right) => {
      const leftScore = left.x + left.y
      const rightScore = right.x + right.y
      if (leftScore !== rightScore) return leftScore - rightScore
      return left.x - right.x
    })

    const xs = sorted.map((point) => point.x)
    const ys = sorted.map((point) => point.y)
    const spreadOk = Math.max(...xs) - Math.min(...xs) >= 210 && Math.max(...ys) - Math.min(...ys) >= 90
    if (!spreadOk) continue

    return sorted.map((point, index) => ({
      id: nodeIds[index],
      x: point.x,
      y: point.y,
    }))
  }

  return [
    { id: 'A', x: 60, y: 55 },
    { id: 'B', x: 155, y: 95 },
    { id: 'C', x: 120, y: 185 },
    { id: 'D', x: 245, y: 80 },
    { id: 'E', x: 225, y: 190 },
    { id: 'F', x: 340, y: 115 },
    { id: 'G', x: 380, y: 195 },
  ]
}

const makeAdjacency = (edges) => {
  const adjacency = Object.fromEntries(nodeIds.map((id) => [id, []]))
  for (const [u, v] of edges) {
    adjacency[u].push(v)
    adjacency[v].push(u)
  }
  for (const id of nodeIds) adjacency[id] = adjacency[id].sort(compareNodeId)
  return adjacency
}

const buildFrames = (edges, adjacency) => {
  const visited = new Set(['A'])
  const parents = { A: 'A' }
  const tree = new Set()
  const scanned = new Set()
  const remaining = new Set(edges.map(([u, v]) => edgeKey(u, v)))
  const frames = [
    {
      label: '開始',
      step: 'Step 2',
      activeSteps: [0, 1, 2],
      current: 'A',
      visited: ['A'],
      tree: [],
      scanned: [],
      parents: { A: 'A' },
    },
  ]

  let current = 'A'

  while (true) {
    const nextNode = adjacency[current].find((neighbor) => remaining.has(edgeKey(current, neighbor)))

    if (!nextNode) {
      if (parents[current] === current) {
        frames.push({
          label: '終了',
          step: 'Step 3',
          activeSteps: [3],
          current,
          visited: [...visited].sort(),
          tree: [...tree],
          scanned: [...scanned],
          parents: { ...parents },
        })
        break
      }

      const previous = parents[current]
      frames.push({
        label: '後退',
        step: 'Step 4',
        activeSteps: [4],
        current: previous,
        visited: [...visited].sort(),
        tree: [...tree],
        scanned: [...scanned],
        parents: { ...parents },
      })
      current = previous
      continue
    }

    const key = edgeKey(current, nextNode)
    remaining.delete(key)

    if (visited.has(nextNode)) {
      scanned.add(key)
      frames.push({
        label: '既探索辺',
        step: 'Step 5, 6',
        activeSteps: [5, 6],
        current,
        visited: [...visited].sort(),
        tree: [...tree],
        scanned: [...scanned],
        parents: { ...parents },
      })
      continue
    }

    visited.add(nextNode)
    parents[nextNode] = current
    tree.add(key)
    frames.push({
      label: '前進',
      step: 'Step 7, 8',
      activeSteps: [5, 7, 8],
      current: nextNode,
      visited: [...visited].sort(),
      tree: [...tree],
      scanned: [...scanned],
      parents: { ...parents },
    })
    current = nextNode
  }

  return frames
}

const canAddEdge = (nodes, selectedEdges, candidate) => {
  const [u, v] = candidate
  const a = nodes.find((node) => node.id === u)
  const b = nodes.find((node) => node.id === v)

  for (const node of nodes) {
    if (node.id === u || node.id === v) continue
    if (pointSegmentDistance(node, a, b) < 28) return false
  }

  for (const [x, y] of selectedEdges) {
    if (u === x || u === y || v === x || v === y) continue
    const c = nodes.find((node) => node.id === x)
    const d = nodes.find((node) => node.id === y)
    if (segmentsIntersect(a, b, c, d)) return false
  }

  return true
}

const makeCandidatePairs = (ids, nodes) => {
  const pairs = []
  for (let i = 0; i < ids.length; i += 1) {
    for (let j = i + 1; j < ids.length; j += 1) {
      if (j - i > 3) continue
      const u = ids[i]
      const v = ids[j]
      const a = nodes.find((node) => node.id === u)
      const b = nodes.find((node) => node.id === v)
      pairs.push([u, v, distance(a, b)])
    }
  }
  return pairs.sort((left, right) => left[2] - right[2])
}

const connectGroup = (ids, nodes, selectedEdges) => {
  const pairs = makeCandidatePairs(ids, nodes)

  for (let i = 1; i < ids.length; i += 1) {
    const current = ids[i]
    const options = shuffle(
      pairs
        .filter(([u, v]) => (u === current && ids.indexOf(v) < i) || (v === current && ids.indexOf(u) < i))
        .map(([u, v]) => [u, v])
    )

    const chosen = options.find((edge) => canAddEdge(nodes, selectedEdges, edge))
    if (chosen) {
      selectedEdges.push(chosen)
      continue
    }

    selectedEdges.push([ids[i - 1], current])
  }

  const extraPairs = shuffle(pairs.map(([u, v]) => [u, v]))
  const maxExtra = Math.max(1, Math.floor(ids.length / 2))
  const extraTarget = Math.floor(Math.random() * (maxExtra + 1))
  let extraCount = 0

  for (const edge of extraPairs) {
    if (extraCount >= extraTarget) break
    if (selectedEdges.some(([u, v]) => edgeKey(u, v) === edgeKey(edge[0], edge[1]))) continue
    if (!canAddEdge(nodes, selectedEdges, edge)) continue
    selectedEdges.push(edge)
    extraCount += 1
  }
}

const mode = ref('connected')

const generateEdges = (nodes) => {
  const selectedEdges = []

  if (mode.value === 'connected') {
    connectGroup(nodeIds, nodes, selectedEdges)
  } else {
    const split = 3 + Math.floor(Math.random() * 2)
    connectGroup(nodeIds.slice(0, split), nodes, selectedEdges)
    connectGroup(nodeIds.slice(split), nodes, selectedEdges)
  }

  return selectedEdges
}

const generateGraph = () => {
  for (let attempt = 0; attempt < 200; attempt += 1) {
    const nodes = generateNodes()
    const edges = generateEdges(nodes)
    const adjacency = makeAdjacency(edges)
    const frames = buildFrames(edges, adjacency)
    if (edges.length > 0) return { nodes, edges, frames }
  }

  const nodes = [
    { id: 'A', x: 52, y: 55 },
    { id: 'B', x: 108, y: 85 },
    { id: 'C', x: 170, y: 92 },
    { id: 'D', x: 232, y: 120 },
    { id: 'E', x: 294, y: 132 },
    { id: 'F', x: 344, y: 164 },
    { id: 'G', x: 392, y: 190 },
  ]
  const edges =
    mode.value === 'connected'
      ? [['A', 'B'], ['B', 'C'], ['C', 'D'], ['D', 'E'], ['E', 'F'], ['F', 'G'], ['B', 'D'], ['D', 'F']]
      : [['A', 'B'], ['B', 'C'], ['D', 'E'], ['E', 'F'], ['F', 'G']]
  return { nodes, edges, frames: buildFrames(edges, makeAdjacency(edges)) }
}

const graph = ref(generateGraph())
const index = ref(0)

const frame = computed(() => graph.value.frames[index.value])
const nodeMap = computed(() => new Map(graph.value.nodes.map((node) => [node.id, node])))

const prev = () => {
  if (index.value > 0) index.value -= 1
}

const next = () => {
  if (index.value < graph.value.frames.length - 1) index.value += 1
}

const regenerate = () => {
  graph.value = generateGraph()
  index.value = 0
}

const setMode = (nextMode) => {
  if (mode.value === nextMode) return
  mode.value = nextMode
  regenerate()
}

const nodeState = (id) => {
  if (frame.value.current === id) return 'current'
  if (frame.value.visited.includes(id)) return 'visited'
  return 'unvisited'
}

const parentOf = (id) => frame.value.parents[id] ?? id

const edgeState = (u, v) => {
  const key = edgeKey(u, v)
  if (frame.value.tree.includes(key)) return 'tree'
  if (frame.value.scanned.includes(key)) return 'scanned'
  return 'idle'
}

const nodeById = (id) => nodeMap.value.get(id)
</script>

<template>
  <div class="dfs-demo">
    <div class="dfs-toolbar">
      <div class="dfs-status">
        <span>{{ index + 1 }} / {{ graph.frames.length }}</span>
      </div>
      <div class="dfs-buttons">
        <button class="dfs-button dfs-arrow-button" @click="prev" :disabled="index === 0" aria-label="前へ">◀</button>
        <button class="dfs-button dfs-arrow-button" @click="next" :disabled="index === graph.frames.length - 1" aria-label="次へ">▶</button>
        <button class="dfs-button" :class="{ 'dfs-button-active': mode === 'connected' }" @click="setMode('connected')">連結</button>
        <button class="dfs-button" :class="{ 'dfs-button-active': mode === 'disconnected' }" @click="setMode('disconnected')">非連結</button>
        <button class="dfs-button dfs-icon-button" @click="regenerate" aria-label="再生成">↻</button>
      </div>
    </div>

    <div class="dfs-stage">
      <svg viewBox="0 0 440 255" class="dfs-graph" role="img" aria-label="深さ優先探索の実行例">
        <line
          v-for="edge in graph.edges"
          :key="`edge-${edge[0]}-${edge[1]}`"
          :x1="nodeById(edge[0]).x"
          :y1="nodeById(edge[0]).y"
          :x2="nodeById(edge[1]).x"
          :y2="nodeById(edge[1]).y"
          class="dfs-edge"
          :class="`dfs-edge-${edgeState(edge[0], edge[1])}`"
        />

        <g v-for="node in graph.nodes" :key="node.id" :transform="`translate(${node.x}, ${node.y})`">
          <circle class="dfs-node" :class="`dfs-node-${nodeState(node.id)}`" r="20" />
          <text class="dfs-node-name" text-anchor="middle" dy="-5">{{ node.id }}</text>
          <text class="dfs-node-parent" text-anchor="middle" dy="12">p={{ parentOf(node.id) }}</text>
        </g>
      </svg>

      <div class="dfs-side">
        <div class="dfs-legend">
          <div><span class="dfs-swatch dfs-swatch-current"></span>現在地 <i>s</i></div>
          <div><span class="dfs-swatch dfs-swatch-visited"></span>探索済み頂点</div>
          <div><span class="dfs-swatch dfs-swatch-unvisited"></span>未探索頂点</div>
          <div><span class="dfs-line"></span>未探索辺</div>
          <div><span class="dfs-line dfs-line-tree"></span>X に加わった辺</div>
          <div><span class="dfs-line dfs-line-scanned"></span>X 以外の参照済みの辺</div>
        </div>
        <div class="dfs-steps">
          <div
            v-for="item in stepItems"
            :key="item.id"
            class="dfs-step-item"
            :class="{ 'dfs-step-item-active': frame.activeSteps.includes(item.id) }"
          >
            <span>{{ item.text }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dfs-demo {
  margin: 0.15rem auto 0.35rem auto;
  max-width: 96%;
}

.dfs-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.55rem;
  margin-bottom: 0.35rem;
}

.dfs-buttons {
  display: flex;
  gap: 0.35rem;
}

.dfs-status {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.4rem;
  color: #003366;
  font-size: 0.92rem;
}

.dfs-button {
  border: 1px solid #003366;
  border-radius: 6px;
  background: white;
  color: #003366;
  padding: 0.16rem 0.6rem;
  font-size: 0.78rem;
  font-weight: 600;
}

.dfs-arrow-button {
  min-width: 2.2rem;
  padding: 0.16rem 0.4rem;
  font-size: 0.95rem;
  line-height: 1;
}

.dfs-icon-button {
  min-width: 2.2rem;
  padding: 0.16rem 0.4rem;
  font-size: 0.92rem;
  line-height: 1;
}

.dfs-button-active {
  background: #003366;
  color: white;
}

.dfs-button:disabled {
  opacity: 0.4;
}

.dfs-stage {
  display: grid;
  grid-template-columns: minmax(0, 24.5rem) 14.0rem;
  gap: 0.65rem;
  align-items: start;
  justify-content: space-between;
}

.dfs-graph {
  width: 42rem;
  height: 28rem;
  border: 1px solid #d6e0ea;
  border-radius: 10px;
  background: #fcfdff;
}

.dfs-edge {
  stroke-linecap: round;
}

.dfs-edge-idle {
  stroke: #c4cfda;
  stroke-width: 2.5;
}

.dfs-edge-tree {
  stroke: #003366;
  stroke-width: 5;
}

.dfs-edge-scanned {
  stroke: #6c87a0;
  stroke-width: 2.5;
  stroke-dasharray: 6 6;
}

.dfs-node {
  stroke-width: 2.2;
}

.dfs-node-current {
  fill: #ffde59;
  stroke: #d17a00;
}

.dfs-node-visited {
  fill: #d9ecff;
  stroke: #003366;
}

.dfs-node-unvisited {
  fill: white;
  stroke: #7f8fa0;
}

.dfs-node-name {
  font-size: 13px;
  font-weight: 700;
  fill: #1f2f3f;
}

.dfs-node-parent {
  font-size: 10.5px;
  fill: #1f2f3f;
}

.dfs-side {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.32rem;
}

.dfs-legend,
.dfs-steps {
  border: 1px solid #d6e0ea;
  border-radius: 10px;
  background: #f8fbff;
  padding: 0.55rem 0.7rem;
  font-size: 1rem;
  line-height: 1.4;
  width: 100%;
}

.dfs-legend > div {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.dfs-step-item {
  display: block;
  color: #4f6478;
  text-align: left;
}

.dfs-step-item + .dfs-step-item {
  margin-top: 0.18rem;
}

.dfs-step-item-active {
  color: #cc0000;
  font-weight: 700;
}

.dfs-legend > div + div {
  margin-top: 0.2rem;
}

.dfs-swatch {
  display: inline-block;
  width: 0.78rem;
  height: 0.78rem;
  border-radius: 999px;
  border: 2px solid transparent;
  flex: 0 0 auto;
}

.dfs-swatch-current {
  background: #ffde59;
  border-color: #d17a00;
}

.dfs-swatch-visited {
  background: #d9ecff;
  border-color: #003366;
}

.dfs-swatch-unvisited {
  background: white;
  border-color: #7f8fa0;
}

.dfs-line {
  display: inline-block;
  width: 1.15rem;
  height: 0;
  border-top: 2.5px solid #c4cfda;
  flex: 0 0 auto;
}

.dfs-line-tree {
  border-top-width: 5px;
  border-top-color: #003366;
}

.dfs-line-scanned {
  border-top-color: #6c87a0;
  border-top-style: dashed;
}
</style>

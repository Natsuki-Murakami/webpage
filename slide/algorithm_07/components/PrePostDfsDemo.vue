<script setup>
import { computed, ref } from 'vue'

const nodeIds = ['A', 'B', 'C', 'D', 'E', 'F', 'G']

const edgeKey = (u, v) => [u, v].sort().join('-')
const stepItems = [
  { id: 0, text: 'Step 0(初期化)' },
  { id: 1, text: 'Step 1(初期点設定)' },
  { id: 2, text: 'Step 2(前順序付与)' },
  { id: 3, text: 'Step 3(終了判定)' },
  { id: 4, text: 'Step 4(後退と後順序)' },
  { id: 5, text: 'Step 5(辺の選択)' },
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

const cloneState = (current, visited, tree, scanned, parents, preorder, postorder) => ({
  current,
  visited: [...visited].sort(compareNodeId),
  tree: [...tree],
  scanned: [...scanned],
  parents: { ...parents },
  preorder: { ...preorder },
  postorder: { ...postorder },
})

const buildFrames = (edges, adjacency) => {
  const visited = new Set(['A'])
  const parents = { A: 'A' }
  const preorder = { A: 1 }
  const postorder = {}
  const tree = new Set()
  const scanned = new Set()
  const remaining = new Set(edges.map(([u, v]) => edgeKey(u, v)))
  const frames = [
    {
      label: '開始',
      activeSteps: [0, 1, 2],
      ...cloneState('A', visited, tree, scanned, parents, preorder, postorder),
    },
  ]

  let preIndex = 2
  let postIndex = 1
  let current = 'A'

  while (true) {
    const nextNode = adjacency[current].find((neighbor) => remaining.has(edgeKey(current, neighbor)))

    if (!nextNode) {
      if (parents[current] === current) {
        postorder[current] = postIndex
        frames.push({
          label: '終了',
          activeSteps: [3],
          ...cloneState(current, visited, tree, scanned, parents, preorder, postorder),
        })
        break
      }

      const previous = parents[current]
      postorder[current] = postIndex
      postIndex += 1
      frames.push({
        label: '後退',
        activeSteps: [4],
        ...cloneState(previous, visited, tree, scanned, parents, preorder, postorder),
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
        activeSteps: [5, 6],
        ...cloneState(current, visited, tree, scanned, parents, preorder, postorder),
      })
      continue
    }

    visited.add(nextNode)
    parents[nextNode] = current
    preorder[nextNode] = preIndex
    preIndex += 1
    tree.add(key)
    frames.push({
      label: '発見',
      activeSteps: [5, 7, 8, 2],
      ...cloneState(nextNode, visited, tree, scanned, parents, preorder, postorder),
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

const edgeState = (u, v) => {
  const key = edgeKey(u, v)
  if (frame.value.tree.includes(key)) return 'tree'
  if (frame.value.scanned.includes(key)) return 'scanned'
  return 'idle'
}

const nodeById = (id) => nodeMap.value.get(id)
const preorderOf = (id) => frame.value.preorder[id] ?? 0
const postorderOf = (id) => frame.value.postorder[id] ?? 0
</script>

<template>
  <div class="prepost-demo">
    <div class="prepost-toolbar">
      <div class="prepost-status">
        <span>{{ index + 1 }} / {{ graph.frames.length }}</span>
      </div>
      <div class="prepost-buttons">
        <button class="prepost-button prepost-arrow-button" @click="prev" :disabled="index === 0" aria-label="前へ">◀</button>
        <button class="prepost-button prepost-arrow-button" @click="next" :disabled="index === graph.frames.length - 1" aria-label="次へ">▶</button>
        <button class="prepost-button" :class="{ 'prepost-button-active': mode === 'connected' }" @click="setMode('connected')">連結</button>
        <button class="prepost-button" :class="{ 'prepost-button-active': mode === 'disconnected' }" @click="setMode('disconnected')">非連結</button>
        <button class="prepost-button prepost-icon-button" @click="regenerate" aria-label="再生成">↻</button>
      </div>
    </div>

    <div class="prepost-stage">
      <svg viewBox="0 0 440 255" class="prepost-graph" role="img" aria-label="前順序番号と後順序番号の実行例">
        <line
          v-for="edge in graph.edges"
          :key="`edge-${edge[0]}-${edge[1]}`"
          :x1="nodeById(edge[0]).x"
          :y1="nodeById(edge[0]).y"
          :x2="nodeById(edge[1]).x"
          :y2="nodeById(edge[1]).y"
          class="prepost-edge"
          :class="`prepost-edge-${edgeState(edge[0], edge[1])}`"
        />

        <g v-for="node in graph.nodes" :key="node.id" :transform="`translate(${node.x}, ${node.y})`">
          <circle class="prepost-node" :class="`prepost-node-${nodeState(node.id)}`" r="22" />
          <text class="prepost-node-name" text-anchor="middle" x="-18" y="-17">{{ node.id }}</text>
          <text class="prepost-node-pre" text-anchor="middle" x="-10" y="-2">{{ preorderOf(node.id) }}</text>
          <text class="prepost-node-post" text-anchor="middle" x="10" y="15">{{ postorderOf(node.id) }}</text>
        </g>
      </svg>

      <div class="prepost-side">
        <div class="prepost-legend">
          <div><span class="prepost-swatch prepost-swatch-current"></span>現在地 <i>s</i></div>
          <div><span class="prepost-swatch prepost-swatch-visited"></span>探索済み頂点</div>
          <div><span class="prepost-swatch prepost-swatch-unvisited"></span>未探索頂点</div>
          <div><span class="prepost-line"></span>未探索辺</div>
          <div><span class="prepost-line prepost-line-tree"></span>X に加わった辺</div>
          <div><span class="prepost-line prepost-line-scanned"></span>X 以外の参照済みの辺</div>
          <div><span class="prepost-number prepost-number-pre">1</span>前順序番号</div>
          <div><span class="prepost-number prepost-number-post">1</span>後順序番号</div>
        </div>
        <div class="prepost-steps">
          <div
            v-for="item in stepItems"
            :key="item.id"
            class="prepost-step-item"
            :class="{ 'prepost-step-item-active': frame.activeSteps.includes(item.id) }"
          >
            <span>{{ item.text }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.prepost-demo {
  margin: 0.15rem auto 0.35rem auto;
  max-width: 96%;
}

.prepost-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.55rem;
  margin-bottom: 0.35rem;
}

.prepost-buttons {
  display: flex;
  gap: 0.35rem;
}

.prepost-status {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.4rem;
  color: #003366;
  font-size: 0.92rem;
}

.prepost-button {
  border: 1px solid #003366;
  border-radius: 6px;
  background: white;
  color: #003366;
  padding: 0.16rem 0.6rem;
  font-size: 0.78rem;
  font-weight: 600;
}

.prepost-arrow-button {
  min-width: 2.2rem;
  padding: 0.16rem 0.4rem;
  font-size: 0.95rem;
  line-height: 1;
}

.prepost-icon-button {
  min-width: 2.2rem;
  padding: 0.16rem 0.4rem;
  font-size: 0.92rem;
  line-height: 1;
}

.prepost-button-active {
  background: #003366;
  color: white;
}

.prepost-button:disabled {
  opacity: 0.4;
}

.prepost-stage {
  display: grid;
  grid-template-columns: minmax(0, 23.8rem) 14.8rem;
  gap: 0.65rem;
  align-items: start;
  justify-content: space-between;
}

.prepost-graph {
  width: 41.1rem;
  height: 28rem;
  border: 1px solid #d6e0ea;
  border-radius: 10px;
  background: #fcfdff;
}

.prepost-edge {
  stroke-linecap: round;
}

.prepost-edge-idle {
  stroke: #c4cfda;
  stroke-width: 2.5;
}

.prepost-edge-tree {
  stroke: #003366;
  stroke-width: 5;
}

.prepost-edge-scanned {
  stroke: #6c87a0;
  stroke-width: 2.5;
  stroke-dasharray: 6 6;
}

.prepost-node {
  stroke-width: 2.2;
}

.prepost-node-current {
  fill: #ffde59;
  stroke: #d17a00;
}

.prepost-node-visited {
  fill: #d9ecff;
  stroke: #003366;
}

.prepost-node-unvisited {
  fill: white;
  stroke: #7f8fa0;
}

.prepost-node-name {
  font-size: 12.5px;
  font-weight: 700;
  fill: #1f2f3f;
}

.prepost-node-pre,
.prepost-node-post {
  font-size: 17px;
  font-weight: 800;
}

.prepost-node-pre {
  fill: #0b63b6;
}

.prepost-node-post {
  fill: #c53131;
}

.prepost-side {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.26rem;
}

.prepost-legend,
.prepost-steps {
  border: 1px solid #d6e0ea;
  border-radius: 10px;
  background: #f8fbff;
  padding: 0.48rem 0.64rem;
  font-size: 1rem;
  line-height: 1.4;
  width: 100%;
}

.prepost-legend > div {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.prepost-step-item {
  display: block;
  color: #4f6478;
  text-align: left;
}

.prepost-step-item + .prepost-step-item {
  margin-top: 0.14rem;
}

.prepost-step-item-active {
  color: #cc0000;
  font-weight: 700;
}

.prepost-legend > div + div {
  margin-top: 0.16rem;
}

.prepost-swatch {
  display: inline-block;
  width: 0.78rem;
  height: 0.78rem;
  border-radius: 999px;
  border: 2px solid transparent;
  flex: 0 0 auto;
}

.prepost-swatch-current {
  background: #ffde59;
  border-color: #d17a00;
}

.prepost-swatch-visited {
  background: #d9ecff;
  border-color: #003366;
}

.prepost-swatch-unvisited {
  background: white;
  border-color: #7f8fa0;
}

.prepost-line {
  display: inline-block;
  width: 1.15rem;
  height: 0;
  border-top: 2.5px solid #c4cfda;
  flex: 0 0 auto;
}

.prepost-line-tree {
  border-top-width: 5px;
  border-top-color: #003366;
}

.prepost-line-scanned {
  border-top-color: #6c87a0;
  border-top-style: dashed;
}

.prepost-number {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 1rem;
  font-size: 1rem;
  font-weight: 800;
  flex: 0 0 auto;
}

.prepost-number-pre {
  color: #0b63b6;
}

.prepost-number-post {
  color: #c53131;
}
</style>

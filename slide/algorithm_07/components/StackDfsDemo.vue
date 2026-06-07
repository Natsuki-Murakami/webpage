<script setup>
import { computed, ref } from 'vue'

const nodeIds = ['A', 'B', 'C', 'D', 'E', 'F', 'G']

const edgeKey = (u, v) => [u, v].sort().join('-')
const stepItems = [
  { id: 0, text: 'Step 0(初期化)' },
  { id: 1, text: 'Step 1(初期点設定)' },
  { id: 2, text: 'Step 2(探索開始)' },
  { id: 3, text: 'Step 3(辺の追加)' },
  { id: 4, text: 'Step 4(終了判定)' },
  { id: 5, text: 'Step 5(辺の取り出し)' },
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

const buildFrames = (edges, adjacency) => {
  const visited = new Set(['A'])
  const tree = new Set()
  const scanned = new Set()
  const remaining = new Set(edges.map(([u, v]) => edgeKey(u, v)))
  const stack = []
  const stacked = new Set()
  const frames = []
  let current = 'A'

  const snapshotStack = () =>
    [...stack].reverse().map((edge, index) => ({
      ...edge,
      isTop: index === 0,
      label: `(${edge.u}, ${edge.v})`,
    }))

  const collectIncidentEdges = (source) => {
    const edgesToPush = []
    for (const neighbor of adjacency[source]) {
      const key = edgeKey(source, neighbor)
      if (!remaining.has(key)) continue
      remaining.delete(key)
      edgesToPush.push({ u: source, v: neighbor, key })
    }
    return edgesToPush
  }

  const makeFrame = (payload) => ({
    current,
    highlightCurrent: false,
    visited: [...visited].sort(),
    tree: [...tree],
    scanned: [...scanned],
    stacked: [...stacked],
    stack: snapshotStack(),
    ...payload,
  })

  frames.push(
    makeFrame({
      label: '開始',
      step: 'Step 0, 1, 2',
      activeSteps: [0, 1, 2],
      highlightCurrent: true,
      message: '始点 A を選び, A を探索済みにする.',
    })
  )

  for (const edge of collectIncidentEdges(current)) {
    stack.push(edge)
    stacked.add(edge.key)
    frames.push(
      makeFrame({
        label: '辺を追加',
        step: 'Step 3',
        activeSteps: [3],
        pushedEdges: [edge.key],
        message: `${current} に接続する辺 ${edge.u}, ${edge.v} を stack に追加する.`,
      })
    )
  }

  while (true) {
    if (stack.length === 0) {
      frames.push(
        makeFrame({
          label: '終了',
          step: 'Step 4',
          activeSteps: [4],
          message: 'stack が空になったので探索を終了する.',
        })
      )
      break
    }

    const edge = stack.pop()
    frames.push(
      makeFrame({
        label: '取り出し',
        step: 'Step 5',
        activeSteps: [5],
        focusEdge: edge.key,
        message: `${edge.u} から ${edge.v} への辺を stack の先頭から取り出す.`,
      })
    )

    if (visited.has(edge.v)) {
      scanned.add(edge.key)
      frames.push(
        makeFrame({
          label: '既探索',
          step: 'Step 6',
          activeSteps: [6],
          message: `${edge.v} は探索済みなので, この辺は DFS 木に加えない.`,
        })
      )
      continue
    }

    tree.add(edge.key)
    frames.push(
      makeFrame({
        current: edge.u,
        label: '未探索点発見',
        step: 'Step 7',
        activeSteps: [7],
        message: `${edge.v} という未探索点を発見し, 辺を X に加える.`,
      })
    )

    current = edge.v
    frames.push(
      makeFrame({
        label: '前進',
        step: 'Step 8, 2',
        activeSteps: [8, 2],
        highlightCurrent: true,
        message: `${edge.v} に移動し, この点を現在地として探索を始める.`,
      })
    )

    visited.add(edge.v)

    for (const nextEdge of collectIncidentEdges(current)) {
      stack.push(nextEdge)
      stacked.add(nextEdge.key)
      frames.push(
        makeFrame({
          label: '辺を追加',
          step: 'Step 3',
          activeSteps: [3],
          pushedEdges: [nextEdge.key],
          message: `${current} に接続する辺 ${nextEdge.u}, ${nextEdge.v} を stack に追加する.`,
        })
      )
    }
  }

  return frames
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
  if (frame.value.highlightCurrent && frame.value.current === id) return 'current'
  if (frame.value.visited.includes(id)) return 'visited'
  return 'unvisited'
}

const edgeState = (u, v) => {
  const key = edgeKey(u, v)
  if (frame.value.focusEdge === key) return 'focus'
  if (frame.value.tree.includes(key)) return 'tree'
  if (frame.value.scanned.includes(key)) return 'scanned'
  if (frame.value.stacked?.includes(key)) return 'pushed'
  if (frame.value.pushedEdges?.includes(key)) return 'pushed'
  return 'idle'
}

const nodeById = (id) => nodeMap.value.get(id)
</script>

<template>
  <div class="stack-dfs-demo">
    <div class="stack-dfs-toolbar">
      <div class="stack-dfs-status">
        <span>{{ index + 1 }} / {{ graph.frames.length }}</span>
      </div>
      <div class="stack-dfs-buttons">
        <button class="stack-dfs-button stack-dfs-arrow-button" @click="prev" :disabled="index === 0" aria-label="前へ">◀</button>
        <button class="stack-dfs-button stack-dfs-arrow-button" @click="next" :disabled="index === graph.frames.length - 1" aria-label="次へ">▶</button>
        <button class="stack-dfs-button" :class="{ 'stack-dfs-button-active': mode === 'connected' }" @click="setMode('connected')">連結</button>
        <button class="stack-dfs-button" :class="{ 'stack-dfs-button-active': mode === 'disconnected' }" @click="setMode('disconnected')">非連結</button>
        <button class="stack-dfs-button stack-dfs-icon-button" @click="regenerate" aria-label="再生成">↻</button>
      </div>
    </div>

    <div class="stack-dfs-stage">
      <div class="stack-dfs-graph-box">
        <svg viewBox="0 0 440 255" class="stack-dfs-graph" role="img" aria-label="stack利用深さ優先探索の実行例">
          <line
            v-for="edge in graph.edges"
            :key="`edge-${edge[0]}-${edge[1]}`"
            :x1="nodeById(edge[0]).x"
            :y1="nodeById(edge[0]).y"
            :x2="nodeById(edge[1]).x"
            :y2="nodeById(edge[1]).y"
            class="stack-dfs-edge"
            :class="`stack-dfs-edge-${edgeState(edge[0], edge[1])}`"
          />

          <g v-for="node in graph.nodes" :key="node.id" :transform="`translate(${node.x}, ${node.y})`">
            <circle class="stack-dfs-node" :class="`stack-dfs-node-${nodeState(node.id)}`" r="20" />
            <text class="stack-dfs-node-name" text-anchor="middle" dy="4">{{ node.id }}</text>
          </g>
        </svg>
      </div>

      <div class="stack-dfs-stack-column">
        <div class="stack-dfs-stack-box">
          <div class="stack-dfs-box-title"><i>stack</i> の状態</div>
          <div class="stack-dfs-stack-items">
            <div v-if="frame.stack.length === 0" class="stack-dfs-stack-empty">empty</div>
            <div
              v-for="item in frame.stack"
              :key="`${item.key}-${item.label}`"
              class="stack-dfs-stack-item"
              :class="{
                'stack-dfs-stack-item-top': item.isTop,
                'stack-dfs-stack-item-pushed': frame.pushedEdges?.includes(item.key),
              }"
            >
              <span class="stack-dfs-stack-rank">{{ item.isTop ? 'top' : '' }}</span>
              <span>{{ item.label }}</span>
            </div>
          </div>
        </div>
      </div>

      <div class="stack-dfs-side">
        <div class="stack-dfs-legend">
          <div><span class="stack-dfs-swatch stack-dfs-swatch-current"></span>現在地 <i>s</i></div>
          <div><span class="stack-dfs-swatch stack-dfs-swatch-visited"></span>探索済み頂点</div>
          <div><span class="stack-dfs-swatch stack-dfs-swatch-unvisited"></span>未探索頂点</div>
          <div><span class="stack-dfs-line"></span>未探索辺</div>
          <div><span class="stack-dfs-line stack-dfs-line-pushed"></span>stack に積んだ辺</div>
          <div><span class="stack-dfs-line stack-dfs-line-tree"></span>X に加わった辺</div>
          <div><span class="stack-dfs-line stack-dfs-line-scanned"></span>X 以外の参照済みの辺</div>
          <div><span class="stack-dfs-line stack-dfs-line-focus"></span>注目中の辺</div>
        </div>

        <div class="stack-dfs-steps">
          <div
            v-for="item in stepItems"
            :key="item.id"
            class="stack-dfs-step-item"
            :class="{ 'stack-dfs-step-item-active': frame.activeSteps.includes(item.id) }"
          >
            <span>{{ item.text }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.stack-dfs-demo {
  margin: 0.15rem auto 0.35rem auto;
  max-width: 96%;
}

.stack-dfs-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.55rem;
  margin-bottom: 0.35rem;
}

.stack-dfs-buttons {
  display: flex;
  gap: 0.35rem;
}

.stack-dfs-status {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #003366;
  font-size: 0.9rem;
  font-weight: 700;
}

.stack-dfs-button {
  border: 1px solid #003366;
  border-radius: 6px;
  background: white;
  color: #003366;
  padding: 0.16rem 0.6rem;
  font-size: 0.78rem;
  font-weight: 600;
}

.stack-dfs-arrow-button {
  min-width: 2.2rem;
  padding: 0.16rem 0.4rem;
  font-size: 0.95rem;
  line-height: 1;
}

.stack-dfs-icon-button {
  min-width: 2.2rem;
  padding: 0.16rem 0.4rem;
  font-size: 0.92rem;
  line-height: 1;
}

.stack-dfs-button-active {
  background: #003366;
  color: white;
}

.stack-dfs-button:disabled {
  opacity: 0.4;
}

.stack-dfs-stage {
  display: grid;
  grid-template-columns: 32.4rem 9.8rem 13.2rem;
  gap: 0.65rem;
  align-items: stretch;
  justify-content: start;
}

.stack-dfs-graph-box,
.stack-dfs-stack-box,
.stack-dfs-legend,
.stack-dfs-steps {
  border: 1px solid #d6e0ea;
  border-radius: 10px;
  background: #f8fbff;
}

.stack-dfs-graph-box {
  padding: 0.3rem;
}

.stack-dfs-stack-column {
  display: flex;
  flex-direction: column;
  width: 100%;
}

.stack-dfs-graph {
  width: 100%;
  height: 25.2rem;
  background: #fcfdff;
}

.stack-dfs-edge {
  stroke-linecap: round;
}

.stack-dfs-edge-idle {
  stroke: #c4cfda;
  stroke-width: 2.5;
}

.stack-dfs-edge-tree {
  stroke: #003366;
  stroke-width: 5;
}

.stack-dfs-edge-pushed {
  stroke: #f08c00;
  stroke-width: 4.5;
}

.stack-dfs-edge-scanned {
  stroke: #6c87a0;
  stroke-width: 2.5;
  stroke-dasharray: 6 6;
}

.stack-dfs-edge-focus {
  stroke: #cc0000;
  stroke-width: 5.5;
}

.stack-dfs-node {
  stroke-width: 2.2;
}

.stack-dfs-node-current {
  fill: #ffde59;
  stroke: #d17a00;
}

.stack-dfs-node-visited {
  fill: #d9ecff;
  stroke: #003366;
}

.stack-dfs-node-unvisited {
  fill: white;
  stroke: #7f8fa0;
}

.stack-dfs-node-name {
  font-size: 13px;
  font-weight: 700;
  fill: #1f2f3f;
}

.stack-dfs-side {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  gap: 0.32rem;
  height: 100%;
}

.stack-dfs-legend,
.stack-dfs-stack-box,
.stack-dfs-steps {
  padding: 0.48rem 0.65rem;
  font-size: 0.93rem;
  line-height: 1.35;
  width: 100%;
}

.stack-dfs-box-title {
  color: #003366;
  font-weight: 700;
  margin-bottom: 0.24rem;
}

.stack-dfs-legend > div {
  display: flex;
  align-items: center;
  gap: 0.45rem;
}

.stack-dfs-legend > div + div {
  margin-top: 0.16rem;
}

.stack-dfs-stack-items {
  display: flex;
  flex-direction: column;
  gap: 0.18rem;
  min-height: 24.1rem;
  justify-content: flex-start;
}

.stack-dfs-stack-empty {
  color: #6b7c8e;
  font-style: italic;
}

.stack-dfs-stack-item {
  display: grid;
  grid-template-columns: 1.45rem 1fr;
  gap: 0.14rem;
  align-items: center;
  padding: 0.16rem 0.26rem;
  border-radius: 6px;
  background: white;
  border: 1px solid #d7e1eb;
  font-size: 1.12rem;
  font-weight: 600;
  line-height: 1.2;
}

.stack-dfs-stack-item-top {
  border-color: #cc0000;
  background: #fff3f3;
}

.stack-dfs-stack-item-pushed {
  border-color: #f08c00;
  background: #fff6e8;
}

.stack-dfs-stack-item-top.stack-dfs-stack-item-pushed {
  border-color: #cc0000;
  background: linear-gradient(90deg, #fff3f3 0%, #fff6e8 100%);
}

.stack-dfs-stack-rank {
  color: #cc0000;
  font-size: 0.62rem;
  font-weight: 700;
  text-transform: uppercase;
}

.stack-dfs-step-item {
  display: block;
  color: #4f6478;
  text-align: left;
}

.stack-dfs-step-item + .stack-dfs-step-item {
  margin-top: 0.14rem;
}

.stack-dfs-step-item-active {
  color: #cc0000;
  font-weight: 700;
}

.stack-dfs-swatch {
  display: inline-block;
  width: 0.78rem;
  height: 0.78rem;
  border-radius: 999px;
  border: 2px solid transparent;
  flex: 0 0 auto;
}

.stack-dfs-swatch-current {
  background: #ffde59;
  border-color: #d17a00;
}

.stack-dfs-swatch-visited {
  background: #d9ecff;
  border-color: #003366;
}

.stack-dfs-swatch-unvisited {
  background: white;
  border-color: #7f8fa0;
}

.stack-dfs-line {
  display: inline-block;
  width: 1.15rem;
  height: 0;
  border-top: 2.5px solid #c4cfda;
  flex: 0 0 auto;
}

.stack-dfs-line-tree {
  border-top-width: 5px;
  border-top-color: #003366;
}

.stack-dfs-line-pushed {
  border-top-width: 4.5px;
  border-top-color: #f08c00;
}

.stack-dfs-line-scanned {
  border-top-color: #6c87a0;
  border-top-style: dashed;
}

.stack-dfs-line-focus {
  border-top-width: 5px;
  border-top-color: #cc0000;
}
</style>

<script setup>
import { computed, onBeforeUnmount, ref } from 'vue'

const nodeRadius = 7
const shuffle = (values) => {
  const array = [...values]
  for (let i = array.length - 1; i > 0; i -= 1) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
  return array
}

const makeNodes = (nodeCount) => {
  const distance = (a, b) => Math.hypot(a.x - b.x, a.y - b.y)
  const points = []

  while (points.length < nodeCount) {
    const candidate = {
      x: 18 + Math.random() * 324,
      y: 22 + Math.random() * 171,
    }

    if (points.every((point) => distance(point, candidate) >= 22)) {
      points.push(candidate)
    }
  }

  const ordered = [...points].sort((left, right) => {
    const leftScore = left.x + left.y
    const rightScore = right.x + right.y
    if (leftScore !== rightScore) return leftScore - rightScore
    return left.x - right.x
  })

  return ordered.map((point, id) => ({
    id,
    x: point.x,
    y: point.y,
  }))
}

const pointSegmentDistance = (point, a, b) => {
  const dx = b.x - a.x
  const dy = b.y - a.y
  const lengthSquared = dx * dx + dy * dy
  if (lengthSquared === 0) return Math.hypot(point.x - a.x, point.y - a.y)

  const t = Math.max(0, Math.min(1, ((point.x - a.x) * dx + (point.y - a.y) * dy) / lengthSquared))
  const projection = {
    x: a.x + t * dx,
    y: a.y + t * dy,
  }
  return Math.hypot(point.x - projection.x, point.y - projection.y)
}

const ccw = (a, b, c) => (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x)

const segmentsIntersect = (a, b, c, d) => {
  const abC = ccw(a, b, c)
  const abD = ccw(a, b, d)
  const cdA = ccw(c, d, a)
  const cdB = ccw(c, d, b)
  return abC * abD < 0 && cdA * cdB < 0
}

const edgeIntersectsOtherEdge = (nodes, edges, from, to) => {
  const a = nodes[from]
  const b = nodes[to]
  return edges.some(([u, v]) => {
    if (u === from || u === to || v === from || v === to) return false
    return segmentsIntersect(a, b, nodes[u], nodes[v])
  })
}

const edgeOverlapsOtherNode = (nodes, from, to) => {
  const a = nodes[from]
  const b = nodes[to]
  return nodes.some((node, index) => {
    if (index === from || index === to) return false
    return pointSegmentDistance(node, a, b) < nodeRadius + 6
  })
}

const makeEdges = (nodes) => {
  const distances = []
  for (let i = 0; i < nodes.length; i += 1) {
    for (let j = i + 1; j < nodes.length; j += 1) {
      distances.push({
        from: i,
        to: j,
        dist: Math.hypot(nodes[i].x - nodes[j].x, nodes[i].y - nodes[j].y),
      })
    }
  }

  distances.sort((left, right) => left.dist - right.dist)

  const edges = []
  const reached = new Set([0])

  while (reached.size < nodes.length) {
    const candidate = distances.find(
      ({ from, to }) =>
        reached.has(from) !== reached.has(to) &&
        !edgeOverlapsOtherNode(nodes, from, to) &&
        !edgeIntersectsOtherEdge(nodes, edges, from, to)
    )
    if (!candidate) return null
    edges.push([candidate.from, candidate.to])
    reached.add(candidate.from)
    reached.add(candidate.to)
  }

  const extraCandidates = shuffle(
    distances.filter(
      ({ from, to }) => !edges.some((edge) => (edge[0] === from && edge[1] === to) || (edge[0] === to && edge[1] === from))
    )
  )
  const extraMin = Math.max(1, Math.floor(nodes.length * 0.2))
  const extraMax = Math.max(extraMin, Math.floor(nodes.length * 0.8))
  const extraCount = extraMin + Math.floor(Math.random() * (extraMax - extraMin + 1))
  for (const candidate of extraCandidates.slice(0, extraCount)) {
    if (edgeOverlapsOtherNode(nodes, candidate.from, candidate.to)) continue
    if (edgeIntersectsOtherEdge(nodes, edges, candidate.from, candidate.to)) continue
    edges.push([candidate.from, candidate.to])
  }

  return edges
}

const makeAdjacency = (nodes, edges) => {
  const adjacency = Object.fromEntries(nodes.map((node) => [node.id, []]))
  for (const [u, v] of edges) {
    adjacency[u].push(v)
    adjacency[v].push(u)
  }
  for (const id of Object.keys(adjacency)) {
    adjacency[id].sort((left, right) => {
      const leftNode = nodes[left]
      const rightNode = nodes[right]
      if (leftNode.y !== rightNode.y) return leftNode.y - rightNode.y
      if (leftNode.x !== rightNode.x) return leftNode.x - rightNode.x
      return left - right
    })
  }
  return adjacency
}

const buildFrames = (adjacency, startNode) => {
  const visited = new Set([startNode])
  const usedEdges = new Set()
  const queue = [startNode]
  const distanceMap = { [startNode]: 0 }
  const frames = [{ current: startNode, visited: [startNode], usedEdges: [], currentDistance: 0, startNode }]

  while (queue.length > 0) {
    const current = queue.shift()

    for (const neighbor of adjacency[current]) {
      const key = `${Math.min(current, neighbor)}-${Math.max(current, neighbor)}`
      if (usedEdges.has(key)) continue

      usedEdges.add(key)

      if (visited.has(neighbor)) {
        frames.push({
          current,
          visited: [...visited],
          usedEdges: [...usedEdges],
          currentDistance: distanceMap[current],
          startNode,
        })
        continue
      }

      visited.add(neighbor)
      distanceMap[neighbor] = distanceMap[current] + 1
      queue.push(neighbor)
      frames.push({
        current,
        visited: [...visited],
        usedEdges: [...usedEdges],
        currentDistance: distanceMap[current],
        startNode,
      })
    }

    if (queue.length > 0) {
      frames.push({
        current: queue[0],
        visited: [...visited],
        usedEdges: [...usedEdges],
        currentDistance: distanceMap[queue[0]],
        startNode,
      })
    }
  }

  return frames
}

const makeGraph = () => {
  const nodeCount = 8 + Math.floor(Math.random() * 13)
  for (let attempt = 0; attempt < 200; attempt += 1) {
    const nodes = makeNodes(nodeCount)
    const edges = makeEdges(nodes)
    if (!edges) continue
    const adjacency = makeAdjacency(nodes, edges)
    const startNode = Math.floor(Math.random() * nodes.length)
    const frames = buildFrames(adjacency, startNode)
    return { nodes, edges, frames }
  }

  const nodes = makeNodes(nodeCount)
  const edges = []
  const adjacency = makeAdjacency(nodes, edges)
  const startNode = Math.floor(Math.random() * nodes.length)
  const frames = buildFrames(adjacency, startNode)
  return { nodes, edges, frames }
}

const graph = ref(makeGraph())
const frameIndex = ref(0)
const isPlaying = ref(false)
const baseInterval = 580
const speedOptions = [
  { label: '0.5x', interval: baseInterval * 2 },
  { label: '1x', interval: baseInterval },
  { label: '1.5x', interval: Math.round(baseInterval / 1.5) },
  { label: '2x', interval: Math.round(baseInterval / 2) },
]
const speedIndex = ref(1)
const frame = computed(() => graph.value.frames[frameIndex.value])

let timer = null

const stopPlayback = () => {
  if (timer !== null) {
    window.clearInterval(timer)
    timer = null
  }
}

const startPlayback = () => {
  stopPlayback()
  timer = window.setInterval(() => {
    if (frameIndex.value >= graph.value.frames.length - 1) {
      frameIndex.value = graph.value.frames.length - 1
      isPlaying.value = false
      stopPlayback()
      return
    }
    frameIndex.value += 1
  }, speedOptions[speedIndex.value].interval)
}

const play = () => {
  if (frameIndex.value >= graph.value.frames.length - 1) frameIndex.value = 0
  isPlaying.value = true
  startPlayback()
}

const pause = () => {
  isPlaying.value = false
  stopPlayback()
}

const regenerate = () => {
  graph.value = makeGraph()
  frameIndex.value = 0
  if (isPlaying.value) startPlayback()
}

const cycleSpeed = () => {
  speedIndex.value = (speedIndex.value + 1) % speedOptions.length
  if (isPlaying.value) startPlayback()
}

const prevFrame = () => {
  pause()
  if (frameIndex.value > 0) frameIndex.value -= 1
}

const nextFrame = () => {
  pause()
  if (frameIndex.value < graph.value.frames.length - 1) frameIndex.value += 1
}

const prevLayer = () => {
  pause()
  const currentDistance = frame.value.currentDistance
  for (let i = frameIndex.value - 1; i >= 0; i -= 1) {
    if (graph.value.frames[i].currentDistance < currentDistance) {
      while (i > 0 && graph.value.frames[i - 1].currentDistance === graph.value.frames[i].currentDistance) i -= 1
      frameIndex.value = i
      return
    }
  }
  frameIndex.value = 0
}

const nextLayer = () => {
  pause()
  const currentDistance = frame.value.currentDistance
  for (let i = frameIndex.value + 1; i < graph.value.frames.length; i += 1) {
    if (graph.value.frames[i].currentDistance > currentDistance) {
      frameIndex.value = i
      return
    }
  }
  frameIndex.value = graph.value.frames.length - 1
}

onBeforeUnmount(() => {
  stopPlayback()
})

const nodeState = (id) => {
  if (frame.value.current === id) return 'current'
  if (frame.value.visited.includes(id)) return 'visited'
  return 'unvisited'
}

const nodeById = (id) => graph.value.nodes.find((node) => node.id === id)
const edgeState = (from, to) => {
  const key = `${Math.min(from, to)}-${Math.max(from, to)}`
  return frame.value.usedEdges.includes(key) ? 'used' : 'idle'
}

const isStartNode = (id) => frame.value.startNode === id
</script>

<template>
  <div class="simple-bfs-preview">
    <div class="simple-bfs-controls">
      <div class="simple-bfs-controls-left">
        <button class="simple-bfs-button simple-bfs-arrow-button" @click="prevLayer" :disabled="frameIndex === 0">⏮</button>
        <button class="simple-bfs-button simple-bfs-arrow-button" @click="prevFrame" :disabled="frameIndex === 0">◀</button>
        <button class="simple-bfs-button simple-bfs-arrow-button" @click="nextFrame" :disabled="frameIndex === graph.frames.length - 1">▶</button>
        <button class="simple-bfs-button simple-bfs-arrow-button" @click="nextLayer" :disabled="frameIndex === graph.frames.length - 1">⏭</button>
      </div>
      <div class="simple-bfs-controls-right">
        <button class="simple-bfs-button simple-bfs-icon-button" @click="play" :disabled="isPlaying" aria-label="再生">▶</button>
        <button class="simple-bfs-button simple-bfs-icon-button" @click="pause" :disabled="!isPlaying" aria-label="停止">■</button>
        <button class="simple-bfs-button simple-bfs-icon-button" @click="regenerate" aria-label="再生成">↻</button>
        <button class="simple-bfs-button simple-bfs-button-active" @click="cycleSpeed">
          速度 {{ speedOptions[speedIndex].label }}
        </button>
      </div>
    </div>
    <svg viewBox="0 0 400 215" class="simple-bfs-graph" role="img" aria-label="幅優先探索の簡略図">
      <line
        v-for="edge in graph.edges"
        :key="`simple-bfs-edge-${edge[0]}-${edge[1]}`"
        :x1="nodeById(edge[0]).x"
        :y1="nodeById(edge[0]).y"
        :x2="nodeById(edge[1]).x"
        :y2="nodeById(edge[1]).y"
        class="simple-bfs-edge"
        :class="`simple-bfs-edge-${edgeState(edge[0], edge[1])}`"
      />

      <circle
        v-for="node in graph.nodes"
        :key="node.id"
        :cx="node.x"
        :cy="node.y"
        :r="nodeRadius"
        class="simple-bfs-node"
        :class="`simple-bfs-node-${nodeState(node.id)}`"
      />
      <circle
        v-for="node in graph.nodes.filter((item) => isStartNode(item.id))"
        :key="`start-${node.id}`"
        :cx="node.x"
        :cy="node.y"
        :r="nodeRadius + 3.5"
        class="simple-bfs-start-ring"
      />
    </svg>
  </div>
</template>

<style scoped>
.simple-bfs-preview {
  margin: 0.55rem 0 0.35rem 0;
}

.simple-bfs-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 0.4rem;
  margin-bottom: 0.35rem;
}

.simple-bfs-controls-left,
.simple-bfs-controls-right {
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

.simple-bfs-button {
  border: 1px solid #003366;
  border-radius: 6px;
  background: white;
  color: #003366;
  padding: 0.14rem 0.58rem;
  font-size: 0.76rem;
  font-weight: 600;
}

.simple-bfs-button:disabled {
  opacity: 0.45;
}

.simple-bfs-button-active {
  background: #003366;
  color: white;
}

.simple-bfs-arrow-button,
.simple-bfs-icon-button {
  min-width: 2.2rem;
  padding: 0.14rem 0.38rem;
  font-size: 0.9rem;
  line-height: 1;
}

.simple-bfs-graph {
  width: 100%;
  height: 15.6rem;
  border: 1px solid #d6e0ea;
  border-radius: 10px;
  background: #fcfdff;
}

.simple-bfs-edge {
  stroke: #aab8c8;
  stroke-width: 2.1;
  stroke-linecap: round;
}

.simple-bfs-edge-used {
  stroke: #4c6175;
  stroke-width: 2.5;
}

.simple-bfs-node {
  stroke-width: 1.7;
}

.simple-bfs-node-current {
  fill: #ffde59;
  stroke: #d17a00;
}

.simple-bfs-node-visited {
  fill: #d9ecff;
  stroke: #003366;
}

.simple-bfs-node-unvisited {
  fill: white;
  stroke: #7f8fa0;
}

.simple-bfs-start-ring {
  fill: none;
  stroke: #003366;
  stroke-width: 1.5;
}
</style>

<template>
  <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
    <div class="flex items-center justify-between mb-3">
      <h3 class="text-sm font-bold text-slate-400">回溯热点分析</h3>
      <span v-if="analysis.totalBacktracks > 0" class="text-xs px-2 py-0.5 bg-orange-900 text-orange-300 rounded-full">
        {{ analysis.totalBacktracks }} 次回溯
      </span>
    </div>

    <div v-if="!store.matchResult" class="text-slate-500 text-sm">
      等待执行匹配...
    </div>

    <div v-else-if="analysis.totalBacktracks === 0" class="text-green-400 text-sm">
      ✓ 无回溯，匹配效率优秀
    </div>

    <div v-else class="space-y-3">
      <div class="bg-slate-900 rounded-lg p-3">
        <div class="flex justify-between text-xs text-slate-500 mb-2">
          <span>回溯率</span>
          <span>{{ (analysis.backtrackRate * 100).toFixed(1) }}%</span>
        </div>
        <div class="w-full bg-slate-700 rounded-full h-2">
          <div 
            class="h-2 rounded-full transition-all duration-300"
            :class="analysis.backtrackRate > 0.3 ? 'bg-red-500' : analysis.backtrackRate > 0.1 ? 'bg-orange-500' : 'bg-yellow-500'"
            :style="{ width: Math.min(100, analysis.backtrackRate * 100) + '%' }"
          ></div>
        </div>
      </div>

      <div v-if="analysis.worstHotspot" class="bg-orange-900/30 border border-orange-700 rounded-lg p-3">
        <div class="flex items-center gap-2 mb-2">
          <span class="text-orange-400">🔥</span>
          <span class="text-sm font-bold text-orange-300">最耗时热点</span>
        </div>
        <div class="text-xs text-slate-400 mb-2">位置: 索引 {{ analysis.worstHotspot.charIndex }}</div>
        <div class="font-mono text-sm bg-slate-900 rounded p-2 overflow-x-auto">
          <span class="text-slate-500">{{ sampleBefore(analysis.worstHotspot) }}</span>
          <span class="bg-red-600 text-white px-0.5 rounded">{{ analysis.worstHotspot.char || ' ' }}</span>
          <span class="text-slate-500">{{ sampleAfter(analysis.worstHotspot) }}</span>
        </div>
        <div class="flex justify-between mt-2 text-xs">
          <span class="text-slate-500">回溯次数</span>
          <span class="text-orange-400 font-bold">{{ analysis.worstHotspot.backtrackCount }}</span>
        </div>
      </div>

      <div class="space-y-2">
        <div class="text-xs font-bold text-slate-500">热点列表 ({{ analysis.hotspots.length }})</div>
        <div class="space-y-2 max-h-64 overflow-y-auto">
          <div 
            v-for="hotspot in analysis.hotspots" 
            :key="hotspot.id"
            class="bg-slate-900 rounded-lg overflow-hidden border border-slate-700"
          >
            <div 
              class="flex items-center justify-between p-2 cursor-pointer hover:bg-slate-800 transition-colors"
              @click="toggleHotspot(hotspot.id)"
            >
              <div class="flex items-center gap-2">
                <span class="text-slate-500 text-xs w-4">{{ expandedIds.has(hotspot.id) ? '▼' : '▶' }}</span>
                <span class="text-cyan-400 font-mono text-sm">'{{ hotspot.char || ' ' }}'</span>
                <span class="text-slate-500 text-xs">索引 {{ hotspot.charIndex }}</span>
              </div>
              <div class="flex items-center gap-2">
                <span class="text-xs text-orange-400 font-bold">{{ hotspot.backtrackCount }} 次</span>
              </div>
            </div>

            <div v-if="expandedIds.has(hotspot.id)" class="border-t border-slate-700 p-2">
              <div class="text-xs font-bold text-slate-500 mb-2">样例文本</div>
              <div class="font-mono text-xs bg-slate-950 rounded p-2 mb-3 overflow-x-auto">
                <span class="text-slate-500">{{ sampleBefore(hotspot) }}</span>
                <span class="bg-red-600 text-white px-0.5 rounded">{{ hotspot.char || ' ' }}</span>
                <span class="text-slate-500">{{ sampleAfter(hotspot) }}</span>
              </div>

              <div class="text-xs font-bold text-slate-500 mb-2">触发链路 ({{ hotspot.chain.length }} 步)</div>
              <div class="space-y-1 max-h-40 overflow-y-auto">
                <div 
                  v-for="node in hotspot.chain" 
                  :key="node.depth"
                  class="flex items-center gap-2 text-xs font-mono"
                >
                  <span class="text-slate-600 w-6">{{ node.depth }}.</span>
                  <span class="text-yellow-400 w-4">'{{ node.char }}'</span>
                  <span class="text-slate-500">→</span>
                  <span class="text-green-400">S{{ node.state }}</span>
                  <span class="text-slate-500">→</span>
                  <span class="text-purple-400">{{ node.transition }}</span>
                  <span v-if="node.isBacktrack" class="text-orange-400 ml-auto">⚠ 回溯</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useRegexStore } from '../store/regex'
import type { BacktrackHotspot } from '../types'

const store = useRegexStore()
const expandedIds = ref<Set<number>>(new Set())

const analysis = computed(() => store.backtrackAnalysis)

function toggleHotspot(id: number) {
  if (expandedIds.value.has(id)) {
    expandedIds.value.delete(id)
  } else {
    expandedIds.value.add(id)
  }
}

function sampleBefore(hotspot: BacktrackHotspot): string {
  const posInSample = hotspot.charIndex - hotspot.sampleStart
  return hotspot.sampleText.substring(0, posInSample)
}

function sampleAfter(hotspot: BacktrackHotspot): string {
  const posInSample = hotspot.charIndex - hotspot.sampleStart
  return hotspot.sampleText.substring(posInSample + 1)
}
</script>

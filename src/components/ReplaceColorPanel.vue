<script setup lang="ts">
import { computed, ref } from 'vue'
import { useEditorStore, type ReplaceScope } from '../editor/useEditorStore'

const store = useEditorStore()

const targetColor = ref('#ff004d')
const replacementColor = ref('#29adff')
const toTransparent = ref(false)
const scope = ref<ReplaceScope>('layer')
const lastChanged = ref<number | null>(null)

const scopes: { id: ReplaceScope; label: string }[] = [
  { id: 'layer', label: '当前图层' },
  { id: 'frame', label: '当前帧' },
  { id: 'all', label: '全部帧' },
]

const affected = computed(() => store.countReplaceablePixels(targetColor.value, scope.value))

const isSameColor = computed(
  () =>
    !toTransparent.value &&
    targetColor.value.toLowerCase() === replacementColor.value.toLowerCase(),
)

const canApply = computed(() => affected.value > 0 && !isSameColor.value)

function useActiveAsTarget() {
  targetColor.value = store.prefs.activeColor
}

function useActiveAsReplacement() {
  replacementColor.value = store.prefs.activeColor
  toTransparent.value = false
}

function apply() {
  if (!canApply.value) return
  const changed = store.replaceColor(
    targetColor.value,
    toTransparent.value ? null : replacementColor.value,
    scope.value,
  )
  lastChanged.value = changed
}
</script>

<template>
  <section class="panel replace-panel">
    <h3>换色</h3>
    <p class="hint">把范围内所有目标色像素一次性替换</p>

    <div class="replace-row">
      <span class="row-label">目标色</span>
      <input v-model="targetColor" type="color" aria-label="目标色" />
      <code>{{ targetColor }}</code>
      <button type="button" class="mini" title="使用当前颜色" @click="useActiveAsTarget">
        用当前色
      </button>
    </div>

    <div class="replace-row">
      <span class="row-label">替换为</span>
      <template v-if="!toTransparent">
        <input v-model="replacementColor" type="color" aria-label="替换色" />
        <code>{{ replacementColor }}</code>
        <button type="button" class="mini" title="使用当前颜色" @click="useActiveAsReplacement">
          用当前色
        </button>
      </template>
      <template v-else>
        <span class="swatch-preview transparent" title="透明" />
        <code>透明</code>
      </template>
    </div>

    <label class="replace-check">
      <input v-model="toTransparent" type="checkbox" />
      替换成透明（擦除该颜色）
    </label>

    <div class="replace-row">
      <span class="row-label">范围</span>
      <div class="scope-list">
        <button
          v-for="s in scopes"
          :key="s.id"
          type="button"
          class="size-btn"
          :class="{ active: scope === s.id }"
          @click="scope = s.id"
        >
          {{ s.label }}
        </button>
      </div>
    </div>

    <p class="hint replace-count">
      将改动 <strong>{{ affected }}</strong> 格
      <span v-if="isSameColor" class="warn">（目标色与替换色相同）</span>
    </p>

    <button
      type="button"
      class="action primary"
      :disabled="!canApply"
      @click="apply"
    >
      确认替换
    </button>
    <p v-if="lastChanged !== null" class="hint replace-done">已替换 {{ lastChanged }} 格</p>
  </section>
</template>

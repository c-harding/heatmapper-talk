<script setup lang="ts">
import { computed, ref, watch } from 'vue';

interface Activity { label: string, start: number, end: number, uploaded: number, showAt?: number }
interface Sync {step: number, time: number}
type SyncInput = Sync | [step: number, time: number]

type FilterMode = 'lastSync' | 'newestActivity'

const {step = 0, activities, syncs: rawSyncs, maxTime, filterMode = 'lastSync'} = defineProps<{
  step?: number
  activities: Activity[]
  syncs: SyncInput[]
  maxTime: number
  filterMode?: FilterMode
}>()

const syncs = computed(() => rawSyncs.map(s => Array.isArray(s) ? { step: s[0], time: s[1] } : s))

const pct = (v: number) => (v / maxTime * 100) + 'cqh'

const syncIndex = computed(() => syncs.value.findLastIndex(s => step >= s.step))

// Track step direction
const dStep = ref(0)
watch(() => step, (newStep, oldStep) => { dStep.value = newStep - oldStep }, { flush: 'sync' })

// newestActivity: highest start among activities uploaded before the sync
const newestActivityStart = (syncTime: number) => {
  const starts = activities.filter(a => a.uploaded <= syncTime).map(a => a.start)
  return Math.max(...starts, 0)
}

// Compute the lastFetch time based on filterMode
const filterTime = (syncTime: number) => {
  if (filterMode === 'lastSync') return syncTime
  return newestActivityStart(syncTime)
}

// One last-fetch line per transition (offset: sync i's time shown at sync i+1's step)
const lastSyncs = computed<Sync[]>(() => syncs.value.slice(0, -1).map((s, i) => ({
  step: syncs.value[i + 1].step,
  time: filterTime(s.time),
})))

// activity is included if its start >= lastSync and uploaded <= sync
const includedInSync = (activity: Activity, sync: Sync, lastSync?: Sync) =>
  (lastSync ? lastSync.time : 0) < activity.start && activity.uploaded <= sync.time

// valid if caught by any sync
const inValidSync = (activity: Activity) =>
  syncs.value.some((s, i) => s.step <= step && includedInSync(activity, s, lastSyncs.value[i - 1]))

// caught in the current (latest visible) sync
const inCurrentSync = (activity: Activity) =>
  syncIndex.value >= 0 && includedInSync(activity, syncs.value[syncIndex.value], lastSyncs.value[syncIndex.value - 1])

const lastSyncVisible = (i: number) =>
  step >= lastSyncs.value[i].step && (i + 1 >= lastSyncs.value.length || step < lastSyncs.value[i + 1].step)

// Forward appear: instant. Forward disappear: fade. Backward appear: fade. Backward disappear: instant+delay.
const lastSyncTransition = (i: number) => {
  const visible = lastSyncVisible(i)
  if (dStep.value > 0) {
    return visible ? 'all 500ms, opacity 0s' : 'all 500ms, opacity 500ms'
  } else {
    return visible ? 'all 500ms, opacity 500ms' : 'all 500ms, opacity 0s 500ms'
  }
}

// Day labels using Intl (de-DE, short weekday)
const dayLabels = Array.from({ length: maxTime }, (_, i) => {
  // 2024-01-01 was a Monday
  const date = new Date(2024, 0, 1 + i)
  return new Intl.DateTimeFormat('de-DE', { weekday: 'short' }).format(date)
})
</script>

<template>
  <div class="pb-1px grid h-full text-sm select-none" style="grid-template-columns: 3em 1fr; container-type: size">
    <!-- Left: time axis with day ticks -->
    <div class="relative border-r border-black">
      <template v-for="(label, i) in dayLabels" :key="i">
        <!-- Tick mark -->
        <div
          class="absolute right-0 w-2 h-px bg-black mr--1px"
          :style="{ top: pct(i) }"
        />
        <!-- Day label -->
        <div
          class="absolute right-3 text-xs text-gray-500 -translate-y-1/2"
          :style="{ top: pct(i + 0.5) }"
        >
          {{ label }}
        </div>
      </template>
      <!-- Tick after last day -->
      <div
        class="absolute right-0 w-2 h-px bg-black mr--1px"
        :style="{ top: pct(maxTime) }"
      />
    </div>

    <!-- Right: activities -->
    <div class="relative flex flex-col h-full">
      <!-- Day separator lines -->
      <div  v-for="(,i) in maxTime + 1" :key="'sep-' + i"
        class="absolute left-0 right-0 h-px bg-gray-200"
        :style="{ top: pct(i) }"
      />
      <!-- Cropped part -->
      <div
        class="relative inset-0 overflow-hidden transition-all duration-500 z-0 flex gap-3 px-3 border-b border-blue-500 box-content pr-12"
        :style="{ height: pct(syncIndex >= 0 ? syncs[syncIndex].time : 0) }"
      >
        <!-- Activity columns -->
        <div
          v-for="(a, i) in activities"
          :key="i"
          class="relative flex-1 transition-all duration-500"
          :style="{ opacity: step >= (a.showAt ?? 0) ? 1 : 0 }"
        >
          <!-- Activity bar -->
          <div
            class="absolute left-0 right-0 rounded-sm flex items-center justify-center px-1 text-xs text-white font-600 overflow-hidden transition-all"
            :class="inCurrentSync(a) ? 'bg-green-500' : inValidSync(a) ? 'bg-green-300' : 'bg-red-500'"
            :style="{
              top: pct(a.start),
              height: pct(a.end - a.start),
            }"
          >
            <span class="text-center leading-tight">
              {{ a.label }}
            </span>
          </div>

          <!-- Reverse T: vertical stem down from bar end to upload point -->
          <div
            v-if="a.uploaded > a.end"
            class="absolute left-1/2 w-px border-l-2 -translate-x-1/2 transition-all"
            :class="inCurrentSync(a) ? 'border-green-400' : inValidSync(a) ? 'border-green-200' : 'border-red-400'"
            :style="{ top: pct(a.end), height: pct(a.uploaded - a.end) }"
          />

          <!-- Reverse T: horizontal crossbar at upload point -->
          <div
            v-if="a.uploaded > a.end"
            class="absolute left-0 right-0 border-t-2 transition-all"
            :class="inCurrentSync(a) ? 'border-green-400' : inValidSync(a) ? 'border-green-200' : 'border-red-400'"
            :style="{ top: pct(a.uploaded) }"
          />
        </div>

        <!-- lastSync lines -->
        <div
          v-for="(sync, i) in lastSyncs" :key="i"
          class="absolute left-0 right-0 border-t border-dotted border-blue-400 z-10 flex items-start justify-end"
          :style="{ top: pct(sync.time), opacity: lastSyncVisible(i) ? 1 : 0, transition: lastSyncTransition(i) }"
        >
          <span class="text-xs text-blue-400 font-mono pr-1">after:</span>
        </div>
      </div>
      <!-- Sync label -->
      <div
        class="text-xs text-blue-500 z-10 transition-all duration-500 text-right"
        :class="step === undefined || step >= 0 ? 'opacity-100' : 'opacity-0'"
      >
        Synchronisation
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { configs, sharedState, useNav } from '@slidev/client';
import type { SlideRoute } from '@slidev/types';
import { useInterval, useSessionStorage } from '@vueuse/core';
import { computed, nextTick, ref, toRef, watch } from 'vue';

const { slides, currentPage, clicks, clicksTotal, isPresenter, go } = useNav();

const timerState = toRef(sharedState, 'timer');
const interval = useInterval(100, { controls: true });

const elapsed = computed(() => {
  void interval.counter.value; // trigger reactivity
  if (timerState.value.status === 'stopped' || !timerState.value.startedAt)
    return 0;
  if (timerState.value.status === 'paused')
    return (timerState.value.pausedAt - timerState.value.startedAt) / 1000;
  return (Date.now() - timerState.value.startedAt) / 1000;
});

const frontmatter = (slide: SlideRoute) =>
  (slide.meta?.slide as { frontmatter?: Record<string, any> } | undefined)
    ?.frontmatter;

function secondsToString(seconds: number): string {
  const h = Math.floor(seconds / 3600);
  const m = Math.floor((seconds % 3600) / 60);
  const s = Math.round(seconds % 60);
  const parts = [
    h > 0 ? h + 'h' : '',
    m > 0 ? m + 'm' : '',
    s > 0 ? s + 's' : '',
  ].filter(Boolean);
  return parts.slice(0, 2).join(' ') || '0s';
}

function parseDuration(input: string): number | null {
  const n = String.raw`[\d][\d_,]*(?:\.[\d_,]+)?`;
  const re = new RegExp(
    String.raw`^\s*` +
      String.raw`(?:(?<h>${n})\s*h\s*)?` +
      String.raw`(?:(?<m>${n})\s*(?:m|min)\s*)?` +
      String.raw`(?:(?<s>${n})\s*(?:s|sec)\s*)?` +
      String.raw`(?:(?<rest>${n})\s*)?$`,
  );
  const match = String(input).match(re);
  if (!match || !match[0].trim()) return null;
  const p = (v?: string) => (v ? parseFloat(v.replace(/[_,]/g, '')) : 0);
  const g = match.groups!;
  const lastExplicit = g.s ? 's' : g.m ? 'm' : g.h ? 'h' : undefined;
  const restUnit: Record<string, number> = { h: 60, m: 1, s: 1 / 60 };
  const restFactor = lastExplicit ? restUnit[lastExplicit] : 1; // bare number alone = minutes
  return p(g.h) * 60 + p(g.m) + p(g.s) / 60 + p(g.rest) * restFactor;
}

const timedSlides = computed(() =>
  slides.value.filter((s) => {
    const fm = frontmatter(s);
    return fm?.sectionDuration && !fm?.disabled;
  }),
);

const slideInfo = computed(() =>
  timedSlides.value.map((s) => {
    const fm = frontmatter(s)!;
    const durationSeconds =
      (parseDuration(String(fm.sectionDuration)) ?? 0) * 60;
    return {
      no: s.no,
      title: fm.shortTitle || fm.title || `Slide ${s.no}`,
      durationSeconds,
    };
  }),
);

const totalDuration = computed(() =>
  slideInfo.value.reduce((sum, s) => sum + s.durationSeconds, 0),
);

const durationOverride = useSessionStorage<number | null>(
  'slidev-timing-duration-override',
  null,
);
const editingDuration = ref(false);
const durationInput = ref('');
const effectiveDuration = computed(
  () => durationOverride.value ?? totalDuration.value,
);

// Scale factor for duration override
const durationScale = computed(() =>
  totalDuration.value > 0 ? effectiveDuration.value / totalDuration.value : 1,
);

const currentSection = computed(() => {
  const page = currentPage.value;
  const past = timedSlides.value.filter((s) => s.no <= page);
  return past.length > 0 ? past[past.length - 1].no : -1;
});

// Progress through current section (0-100%)
// Each slide gets an equal slice. Within the current slide, subdivide by clicks.
// E.g. 3 slides, current is slide 1 with 3 clicks (4 states):
// slide 1 owns [0, 1/3). At click 0: 1/24, click 1: 3/24, click 2: 5/24, click 3: 7/24
const sectionProgress = computed(() => {
  const idx = timedSlides.value.findIndex((s) => s.no === currentSection.value);
  if (idx < 0) return 0;
  const sectionStart = timedSlides.value[idx].no;
  const sectionEnd =
    idx + 1 < timedSlides.value.length
      ? timedSlides.value[idx + 1].no
      : slides.value.length + 1;
  const totalSlides = sectionEnd - sectionStart;
  if (totalSlides <= 0) return 0;

  const slideIdx = currentPage.value - sectionStart; // 0-based slide index within section
  const clickStates = clicksTotal.value + 1; // total states for current slide
  const clickIdx = clicks.value; // 0-based current click

  // Odd fraction within current slide's slot, then offset by completed slides
  const posInSlide = (2 * clickIdx + 1) / (2 * clickStates);
  return ((slideIdx + posInSlide) / totalSlides) * 100;
});

// Expected progress range for the current section (scaled in endTime mode)
const expectedRange = computed(() => {
  const scale = timeDisplay.value === 'endTime' ? endTimeScale.value : 1;
  let start = 0;
  for (const s of slideInfo.value) {
    const end = start + s.durationSeconds * scale;
    if (s.no === currentSection.value) return { start, end };
    start = end;
  }
  return { start: 0, end: 0 };
});

const arrowColor = computed(() => {
  const e = elapsed.value;
  const { start, end } = expectedRange.value;
  if (e >= start && e <= end) return '#000'; // on track
  if (e < start) return '#22c55e'; // ahead (green-500)
  return '#ef4444'; // behind (red-500)
});

type TimeDisplay = 'duration' | 'endTime';
const timeDisplay = ref<TimeDisplay>('duration');

const configEndTime = computed(
  () =>
    ((configs as any)['end-time'] ?? (configs as any).endTime) as
      | string
      | undefined,
);
const endTimeOverride = useSessionStorage<string | null>(
  'slidev-timing-endtime-override',
  null,
);
const endTime = computed(() => endTimeOverride.value ?? configEndTime.value);
const editingEndTime = ref(false);
const endTimeInput = ref('');

// Parse "HH:MM" to seconds since midnight
function parseClockTime(s: string): number | null {
  const m = s.match(/^(\d{1,2}):(\d{2})$/);
  return m ? parseInt(m[1]) * 3600 + parseInt(m[2]) * 60 : null;
}

function formatClockTime(seconds: number): string {
  const h = Math.floor((((seconds % 86400) + 86400) % 86400) / 3600);
  const m = Math.floor((seconds % 3600) / 60);
  return `${h}:${String(m).padStart(2, '0')}`;
}

const endTimeSeconds = computed(() =>
  endTime.value ? parseClockTime(endTime.value) : null,
);

// Wall clock time when the timer started (seconds since midnight)
const startClockSeconds = computed(() => {
  if (timerState.value.status === 'stopped' || !timerState.value.startedAt)
    return null;
  const d = new Date(timerState.value.startedAt);
  return d.getHours() * 3600 + d.getMinutes() * 60 + d.getSeconds();
});

// Total span from timer start to end time (wall clock)
const endTimeSpan = computed(() => {
  if (startClockSeconds.value == null || endTimeSeconds.value == null)
    return null;
  let span = endTimeSeconds.value - startClockSeconds.value;
  if (span <= 0) span += 86400; // wrap past midnight
  return span;
});

const totalDurationString = computed(() =>
  secondsToString(effectiveDuration.value),
);

const progressPct = computed(() => {
  if (timeDisplay.value === 'endTime' && endTimeSpan.value) {
    return Math.min((elapsed.value / endTimeSpan.value) * 100, 100);
  }
  return effectiveDuration.value > 0
    ? Math.min((elapsed.value / effectiveDuration.value) * 100, 100)
    : 0;
});

// Scale factor: how much to stretch section durations to fill the end-time span
const endTimeScale = computed(() => {
  if (!endTimeSpan.value || totalDuration.value === 0) return 1;
  return endTimeSpan.value / totalDuration.value;
});

// Seconds until the configured end time
const timeUntilEnd = computed(() => {
  if (endTimeSeconds.value == null) return null;
  void interval.counter.value; // trigger reactivity
  const now = new Date();
  const nowSeconds =
    now.getHours() * 3600 + now.getMinutes() * 60 + now.getSeconds();
  let diff = endTimeSeconds.value - nowSeconds;
  if (diff <= 0) diff += 86400;
  return diff;
});

// Show play-to-end only when end time is less than duration away
const showPlayToEnd = computed(() => {
  if (timeUntilEnd.value == null) return false;
  return timeUntilEnd.value < effectiveDuration.value;
});

function playToEnd() {
  if (endTimeSeconds.value == null) return;
  const now = new Date();
  const todayMidnight = new Date(
    now.getFullYear(),
    now.getMonth(),
    now.getDate(),
  ).getTime();
  let endTimeMs = todayMidnight + endTimeSeconds.value * 1000;
  if (endTimeMs <= now.getTime()) endTimeMs += 86400 * 1000;

  timerState.value = {
    ...timerState.value,
    status: 'paused',
    startedAt: endTimeMs - effectiveDuration.value * 1000,
    pausedAt: now.getTime(),
  };
  timeDisplay.value = 'endTime';

  // play via Slidev's UI button so the timer is triggered correctly
  nextTick(() => {
    const pauseIcon = document.querySelector<HTMLElement>(
      '.slidev-presenter > .grid-container > .grid-section.bottom > :last-child .i-carbon\\:play',
    );
    pauseIcon?.click();
  });
}

// --- Recorded slide durations ---
interface RecordedVisit {
  start: number;
  end: number;
}
interface RecordedSlideDuration {
  visits: RecordedVisit[];
}
interface RecordedSlideDurations {
  slides: Record<number, RecordedSlideDuration>;
  currentSlide: number;
  currentStart: number;
}

const recordedDurations = useSessionStorage<RecordedSlideDurations>(
  'slidev-recorded-durations',
  {
    slides: {},
    currentSlide: -1,
    currentStart: -1,
  },
);

function currentElapsed(): number {
  if (!timerState.value.startedAt) return 0;
  if (timerState.value.status === 'paused')
    return (timerState.value.pausedAt - timerState.value.startedAt) / 1000;
  return (Date.now() - timerState.value.startedAt) / 1000;
}

function endCurrentVisit() {
  const rec = recordedDurations.value;
  if (rec.currentSlide < 0 || rec.currentStart < 0) return;
  const end = currentElapsed();
  if (!rec.slides[rec.currentSlide]) {
    rec.slides[rec.currentSlide] = { visits: [] };
  }
  rec.slides[rec.currentSlide].visits.push({ start: rec.currentStart, end });
  rec.currentStart = -1;
}

function beginVisit() {
  recordedDurations.value.currentSlide = currentPage.value;
  recordedDurations.value.currentStart = currentElapsed();
}

function logRecordedDurations() {
  const rec = recordedDurations.value;
  const lines: string[] = [];
  for (const section of slideInfo.value) {
    // Find all slide numbers in this section
    const idx = timedSlides.value.findIndex((s) => s.no === section.no);
    const sectionStart = section.no;
    const sectionEnd =
      idx + 1 < timedSlides.value.length
        ? timedSlides.value[idx + 1].no
        : slides.value.length + 1;

    let sectionTotal = 0;
    const slideLines: string[] = [];
    for (let no = sectionStart; no < sectionEnd; no++) {
      const visits = rec.slides[no]?.visits ?? [];
      if (visits.length === 0) continue;
      const durations = visits.map((v) => v.end - v.start);
      const total = durations.reduce((s, d) => s + d, 0);
      sectionTotal += total;
      const parts = durations.map((d) => secondsToString(d));
      const title =
        frontmatter(slides.value.find((s) => s.no === no)!)?.title ??
        `Slide ${no}`;
      slideLines.push(
        `  ${no}. ${title}: ${parts.join(' + ')} = ${secondsToString(total)}`,
      );
    }
    if (slideLines.length > 0) {
      lines.push(
        `${section.title} (${secondsToString(sectionTotal)} / ${secondsToString(section.durationSeconds)}):`,
      );
      lines.push(...slideLines);
    }
  }
  console.log('Recorded slide durations:\n' + lines.join('\n'));
}

// Reset when timer transitions from stopped → running/paused
watch(
  () => timerState.value.status,
  (newStatus, oldStatus) => {
    if (
      oldStatus === 'stopped' &&
      (newStatus === 'running' || newStatus === 'paused')
    ) {
      recordedDurations.value = {
        slides: {},
        currentSlide: -1,
        currentStart: -1,
      };
      if (newStatus === 'running') beginVisit();
    } else if (oldStatus === 'running' && newStatus === 'paused') {
      endCurrentVisit();
    } else if (oldStatus === 'paused' && newStatus === 'running') {
      beginVisit();
    } else if (newStatus === 'stopped') {
      endCurrentVisit();
    }
  },
);

// Track slide transitions while running
watch(currentPage, () => {
  if (timerState.value.status !== 'running') return;
  endCurrentVisit();
  beginVisit();
});
</script>

<template>
  <Teleport
    v-if="isPresenter"
    defer
    to=".slidev-presenter > .grid-container > .grid-section.bottom > nav + div"
  >
    <div class="flex flex-row items-stretch gap-2">
      <div class="flex flex-col flex-1 min-w-0">
        <!-- Progress marker -->
        <div class="relative h-3 mx-2">
          <div
            class="absolute w-3 h-3 -translate-x-1/2"
            :style="{
              left: progressPct + '%',
              clipPath: 'polygon(50% 100%, 0 0, 100% 0)',
              backgroundColor: arrowColor,
            }"
            @dblclick="logRecordedDurations()"
          />
        </div>
        <div class="flex text-xs rounded p-2 z-100 w-100%">
          <div
            v-for="(slide, i) in slideInfo"
            :key="slide.no"
            class="py-0.5 truncate border-white border-2 bg-gray-300 cursor-pointer hover:bg-gray-400 text-center relative overflow-hidden"
            :class="
              currentSection === slide.no ? 'bg-blue-300! pt-2 -mt-1.5' : ''
            "
            :style="{ flexBasis: 0, flexGrow: slide.durationSeconds }"
            @click="go(slide.no)"
          >
            <div
              v-if="currentSection === slide.no"
              class="absolute top-0 left-0 h-1.5 bg-blue-600"
              :style="{ width: sectionProgress + '%' }"
            />
            {{ slide.title }}<br />{{
              secondsToString(
                slide.durationSeconds *
                  (timeDisplay === 'endTime' ? endTimeScale : durationScale),
              )
            }}
          </div>
        </div>
      </div>
      <fieldset class="flex flex-col text-xs gap-1 justify-center pl-2 w-16">
        <label class="flex items-center gap-1 cursor-pointer">
          <input type="radio" value="duration" v-model="timeDisplay" />
          <span
            v-if="!editingDuration"
            @dblclick="
              editingDuration = true;
              durationInput = String(effectiveDuration / 60) + 'm';
            "
          >
            {{ totalDurationString }}
          </span>
          <input
            v-else
            v-model="durationInput"
            class="w-8 bg-transparent border-b border-current outline-none"
            @keydown.enter="
              durationOverride =
                (parseDuration(durationInput) ?? 0) * 60 || null;
              editingDuration = false;
            "
            @keydown.escape="editingDuration = false"
            @blur="
              durationOverride =
                (parseDuration(durationInput) ?? 0) * 60 || null;
              editingDuration = false;
            "
            autofocus
          />
        </label>
        <label
          class="flex items-center gap-1"
          :class="endTime ? 'cursor-pointer' : 'opacity-40'"
        >
          <input
            type="radio"
            value="endTime"
            :disabled="!endTime"
            v-model="timeDisplay"
          />
          <span
            v-if="!editingEndTime"
            @dblclick="
              editingEndTime = true;
              endTimeInput = endTime ?? '';
            "
          >
            {{ endTime ?? 'End time' }}
          </span>
          <input
            v-else
            v-model="endTimeInput"
            class="w-8 bg-transparent border-b border-current outline-none"
            @keydown.enter="
              endTimeOverride = endTimeInput || null;
              editingEndTime = false;
              timeDisplay = 'endTime';
            "
            @keydown.escape="editingEndTime = false"
            @blur="
              endTimeOverride = endTimeInput || null;
              editingEndTime = false;
            "
            autofocus
          />
        </label>
      </fieldset>
      <button
        :disabled="!showPlayToEnd"
        class="text-xs p-1.5 my-auto mr--2 cursor-pointer not-disabled:hover:bg-gray-300 disabled:opacity-30 rounded flex items-center"
        title="Start timer to finish at end time"
        @click="playToEnd"
      >
        <svg
          viewBox="0 0 16 16"
          class="w-4 h-4"
          fill="none"
          stroke="currentColor"
          stroke-width="1"
        >
          <polygon points="1,2 11,8 1,14" />
          <line x1="13.25" y1="2" x2="13.25" y2="14" />
        </svg>
      </button>
    </div>
  </Teleport>
</template>

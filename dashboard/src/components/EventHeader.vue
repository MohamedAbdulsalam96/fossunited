<template>
  <div class="flex flex-col gap-3 mt-5 md:mt-0">
    <div v-if="chapter.doc">
      <FossClubBranding v-if="chapter.doc.chapter_type == 'FOSS Club'">{{
        chapter.doc.chapter_name
      }}</FossClubBranding>
      <CityCommunityBranding v-else>{{ chapter.doc.chapter_name }}</CityCommunityBranding>
    </div>
    <div class="flex gap-3 items-center">
      <div class="prose">
        <h1>{{ event.event_name }}</h1>
      </div>
      <Badge v-if="formExists && form.data" :theme="getBadgeTheme()" size="md">
        <div class="font-medium flex items-center gap-1">
          <LivePing v-if="shouldShowStatusDot()" />
          <span>{{ getBadgeText() }}</span>
        </div>
      </Badge>
      <Badge v-if="!formExists && form" :theme="'gray'" size="md">
        <div class="font-medium">
          <span> Form Not Created </span>
        </div>
      </Badge>
    </div>
    <div class="flex gap-2 items-center text-base text-gray-700">
      <div>
        {{ event.event_type }}
      </div>
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        fill="currentColor"
        class="fill-gray-600 w-2 h-2 icon icon-tabler icons-tabler-filled icon-tabler-circle"
      >
        <path stroke="none" d="M0 0h24v24H0z" fill="none" />
        <path
          d="M7 3.34a10 10 0 1 1 -4.995 8.984l-.005 -.324l.005 -.324a10 10 0 0 1 4.995 -8.336z"
        />
      </svg>
      <div>
        <span>
          {{ getFormattedDate(event.event_start_date) }}
        </span>
        <span v-if="event.event_start_date != event.event_end_date">
          - {{ getFormattedDate(event.event_end_date) }}
        </span>
      </div>
    </div>
  </div>
</template>
<script setup>
import { defineProps } from 'vue'
import { createDocumentResource, Badge } from 'frappe-ui'
import LivePing from '@/components/animation/LivePing.vue'
import CityCommunityBranding from '@/components/CityCommunityBranding.vue'
import FossClubBranding from '@/components/FossClubBranding.vue'

import { computed } from 'vue'

const props = defineProps({
  event: {
    type: Object,
    default: null,
  },
  formExists: {
    type: Boolean,
    default: false,
  },
  form: {
    type: Object,
    default: null,
  },
})

function getFormattedDate(date) {
  return new Date(date).toLocaleDateString('en-IN', {
    day: 'numeric',
    month: 'long',
    year: 'numeric',
  })
}

const chapter = createDocumentResource({
  doctype: 'FOSS Chapter',
  name: props.event.chapter,
  fields: ['*'],
  auto: true,
})

const isEvent = computed(() => props.form.data?.doctype === 'Event')

function getBadgeTheme() {
  if (isEvent.value) {
    switch (props.event.status) {
      case 'Live':
        return 'green'
      case 'Cancelled':
        return 'red'
      case 'Draft':
        return 'orange'
      default:
        return 'gray'
    }
  }
  return props.form.data.is_published ? 'green' : 'gray'
}

function getBadgeText() {
  if (isEvent.value) {
    return `${props.event.status}`
  }
  const formType = props.form.data.doctype.split(' ').pop()
  return `${formType} ${props.form.data.is_published ? 'Live' : 'Unpublished'}`
}

function shouldShowStatusDot() {
  return (
    (isEvent.value && props.event.status === 'Live') ||
    (!isEvent.value && props.form.data.is_published)
  )
}
</script>

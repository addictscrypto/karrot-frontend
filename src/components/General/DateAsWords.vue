<template>
  <div :title="tooltipContent">
    {{ dateInWords }}
  </div>
</template>

<script>
import dateFnsHelper from '@/services/dateFnsHelper'

export default {
  props: {
    date: {
      type: Date | String, // TODO remove string, always convert to date before
      required: true,
    },
  },
  computed: {
    tooltipContent () {
      return this.$d(new Date(this.date), 'long')
    },
    dateInWords () {
      return dateFnsHelper.distanceInWordsToNow(this.date, { addSuffix: true, disallowFuture: true })
    },
  },
}
</script>

<script>
import ContextMenu from '@/components/shared/contextMenu/ContextMenu'
import ContextMenuItem from '@/components/shared/contextMenu/ContextMenuItem'
import Date from '@/components/shared/date/Date'
import Dropdown from '@/components/shared/dropdown/Dropdown'
import StatCustomizeMenuMobile from '@/components/stat/StatCustomizeMenuMobile'

export default {
  components: {
    ContextMenu,
    ContextMenuItem,
    Date,
    Dropdown,
    StatCustomizeMenuMobile
  },

  data () {
    return {
      visiblePeriodMenu: false
    }
  }
}
</script>

<template lang="pug">
.periodNav
  .periodNav__link
    .d-button-cirle(
      @click="$store.dispatch('setPeriodPrev')"
      :class="{ _disable: $store.state.filter.period === 'all' || $store.getters.isFirstPeriodSelected }")
      .mdi.mdi-chevron-left

  .periodNav__link._grow
    ContextMenu(
      :position="{ left: true, bottom: true }"
      :visible="visiblePeriodMenu"
      v-on:onClickOpener="visiblePeriodMenu = !visiblePeriodMenu")
      template(slot="opener")
        Dropdown(:active="visiblePeriodMenu")
          template(slot="title"): Date

      template(slot="content")
        ContextMenuItem(
          icon="mdi mdi-weather-sunset-up"
          :title="$lang.dates.day.simple"
          :selected="$store.state.filter.period === 'day'"
          v-on:onClick="$store.dispatch('setPeriod', 'day')"
          v-on:onClose="visiblePeriodMenu = !visiblePeriodMenu")
        ContextMenuItem(
          icon="mdi mdi-calendar-week"
          :title="$lang.dates.week.simple"
          :selected="$store.state.filter.period === 'week'"
          v-on:onClick="$store.dispatch('setPeriod', 'week')"
          v-on:onClose="visiblePeriodMenu = !visiblePeriodMenu")
        ContextMenuItem(
          icon="mdi mdi-calendar"
          :title="$lang.dates.month.simple"
          :selected="$store.state.filter.period === 'month'"
          v-on:onClick="$store.dispatch('setPeriod', 'month')"
          v-on:onClose="visiblePeriodMenu = !visiblePeriodMenu")
        ContextMenuItem(
          icon="mdi mdi-calendar-star"
          :title="$lang.dates.year.simple"
          :selected="$store.state.filter.period === 'year'"
          v-on:onClick="$store.dispatch('setPeriod', 'year')"
          v-on:onClose="visiblePeriodMenu = !visiblePeriodMenu")
        ContextMenuItem(
          icon="mdi mdi-database"
          :title="$lang.dates.all"
          :selected="$store.state.filter.period === 'all'"
          v-on:onClick="$store.dispatch('setPeriod', 'all')"
          v-on:onClose="visiblePeriodMenu = !visiblePeriodMenu")

    .customize
      StatCustomizeMenuMobile(
        icon="mdi mdi-tune"
        :position="{ right: true, bottom: true }")

  .periodNav__link
    .d-button-cirle(
      @click="$store.dispatch('setPeriodNext')"
      :class="{ _disable: $store.state.filter.period === 'all' || $store.getters.isLastPeriodSelected }")
      .mdi.mdi-chevron-right
</template>

<style lang="stylus" scoped>
@import "~@/stylus/variables/margins"
@import "~@/stylus/variables/media"

.periodNav
  display flex
  align-items stretch
  justify-content space-between
  width 100%
  padding 0 $m7

  &__link
    display flex
    align-items center
    justify-content center
    padding 0 $m3
    font-size 24px

    @media $media-phone
      padding 0

.customize
  margin-left 20px
  @media $media-phone-sm
    display none
</style>

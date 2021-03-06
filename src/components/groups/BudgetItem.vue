<script>
import Amount from '@/components/amount/Amount'
import TrnItem from '@/components/groups/BudgetItemTrn'
import TrnsList from '@/components/trns/list/TrnsList2'
import { formatDate } from '@/utils/formatDate'

export default {
  components: {
    Amount,
    TrnItem,
    TrnsList
  },

  props: {
    budget: {
      type: Object,
      required: true
    }
  },

  data () {
    return {
      isTrnsVisible: false
    }
  },

  computed: {
    formatedDate () {
      return formatDate(this.budget.date, 'number')
    },

    totalAmount () {
      return this.gotAmount.incomes - this.gotAmount.expenses || 0
    },

    trnsIds () {
      if (this.budget.trnsIds) {
        return Object.keys(this.budget.trnsIds)
          .sort((a, b) => {
            if (this.$store.state.trns.items[a].date > this.$store.state.trns.items[b].date) return -1
            if (this.$store.state.trns.items[a].date < this.$store.state.trns.items[b].date) return 1
            return 0
          })
      }
    },

    gotAmount () {
      if (this.trnsIds) {
        return this.$store.getters.getTotalOfTrnsIds(this.trnsIds)
      }
      return 0
    },

    styles () {
      return {
        width: `${Math.abs(this.gotAmount.incomes - this.gotAmount.expenses) / Math.abs(this.gotAmount.incomes) * 100}%`
      }
    }
  },

  methods: {
    async removeBudgetId (id) {
      // await this.$store.dispatch('removeBudget', id)
    },

    handleTrnItemClick (trnId) {
      this.$store.commit('hideCategoryModal')
      this.$store.commit('showTrnModal')
      this.$store.commit('setTrnModalId', trnId)
    }
  }
}
</script>

<template lang="pug">
.budgetItem(@click="removeBudgetId(budget.id)")
  .budgetItem__top
    .budgetItem__meta
      .budgetItem__name {{ budget.name }}
      .budgetItem__description(v-if="budget.description") {{ budget.description }}

    //- total
    .budgetItem__total
      .sum._right
        .sum__title {{ $lang.groups.stat.total }}
        .sum__amount
          Amount(:currency="budget.currency" :value="totalAmount")

  //- info
  .budgetItem__info(@click="isTrnsVisible = !isTrnsVisible")
    .budgetItem__amounts
      //- expenses
      .sum
        .sum__title {{ $lang.groups.stat.expenses }}
        .sum__amount
          Amount(:currency="budget.currency" :type="0" :value="gotAmount.expenses")

      //- incomes
      .sum._right
        .sum__title {{ $lang.groups.stat.incomes }}
        .sum__amount
          Amount(:currency="budget.currency" :type="1" :value="gotAmount.incomes")

    .budgetItem__graph: .budgetItem__graph__in(:style="styles")

  .budgetItem__trns(v-if="budget.trnsIds" v-show="isTrnsVisible")
    TrnsList(:ids="trnsIds" v-slot="{ trns }")
      TrnItem(
        v-for="trnId in trnsIds"
        v-on:onClick="handleTrnItemClick"
        :category="trns[trnId].category"
        :id="trns[trnId].id"
        :key="trns[trnId].id"
        :trn="trns[trnId].trn"
        :wallet="trns[trnId].wallet")
</template>

<style lang="stylus" scoped>
@import "~@/stylus/variables/media"

.budgetItem
  overflow hidden
  font-size 14px
  background var(--c-bg-4)
  border 1px solid var(--c-bg-5)
  border-radius 3px

  &:last-child
    margin-bottom 0

  &__top
    display flex
    align-items center
    justify-content space-between
    padding 12px

    +media-laptop()
      padding 16px

  &__name
    font-size 14px

  &__description
    padding-top 5px
    font-size 11px
    color var(--c-font-4)

  &__info
    padding 16px
    background var(--c-bg-3)
    border-top 1px solid var(--c-bg-2)

    +media-laptop()
      padding 16px

  &__amounts
    display flex
    justify-content space-between
    padding-bottom 8px

  &__graph
    overflow hidden
    background var(--c-bg-7)
    border-radius 3px

    &__in
      height 6px
      min-width 0px
      background rgba(44, 173, 34, 0.7)

  &__trns
    margin-top 1px
    padding 0 12px
    border-top 1px solid var(--c-bg-2)

    +media-laptop()
      padding 0 16px
</style>

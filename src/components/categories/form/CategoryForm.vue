<script>
import { focus } from 'vue-focus'
import { db } from '@/firebase'
import { generateSimpleId } from '@/utils/id'
import colors from '@/components/ui/store/colors'
import icons from '@/components/ui/store/icons'

import Button from '@/components/shared/button/Button'
import CategoriesView from '@/components/categories/list/CategoriesView'
import Checkbox from '@/components/shared/inputs/Checkbox'
import ComponentWrap from '@/components/layout/component/Component'
import ModalBottom from '@/components/modal/ModalBottom'
import ModalButton from '@/components/modal/ModalButton'

export default {
  components: {
    Button,
    CategoriesView,
    Checkbox,
    ComponentWrap,
    ModalBottom,
    ModalButton
  },

  directives: { focus },

  data () {
    return {
      showParents: false,
      showColors: false,
      showIcons: false,
      filter: null,
      category: {
        color: '',
        icon: '',
        name: null,
        parentId: 0,
        showInLastUsed: true,
        showInQuickSelector: false,
        showStat: true
      }
    }
  },

  computed: {
    categoryId () {
      return this.$store.state.categories.editId
    },

    selectedIcons () {
      if (this.filter) return icons.filter(icon => icon.includes(this.filter.toLowerCase()))
      return icons
    }
  },

  watch: {
    categoryId: {
      handler (categoryId) {
        if (categoryId) {
          this.category = {
            ...this.category,
            ...this.$store.state.categories.items[this.categoryId]
          }
        }
      },
      immediate: true
    }
  },

  created () {
    this.colors = colors
    this.icons = icons
    if (!this.$store.state.categories.editId) {
      this.category.icon = `mdi ${icons[Math.floor(Math.random() * icons.length)]}`
      this.category.color = colors[Math.floor(Math.random() * colors.length)]
    }
  },

  beforeDestroy () {
    this.$store.commit('setCategoryEditId', null)
  },

  methods: {
    handleColorSelect (color) {
      this.category.color = color
      this.showColors = false
    },

    handleIconSelect (icon) {
      this.category.icon = `mdi ${icon}`
      this.showIcons = false
    },

    handleParenCategorySelect (categoryId) {
      const parentCategory = this.$store.state.categories.items[categoryId]
      if (parentCategory) {
        this.category.color = parentCategory.color
      }
      this.category.parentId = categoryId
      this.showParents = false
    },

    handleSubmit () {
      if (this.validateForm()) {
        const uid = this.$store.state.user.user.uid
        const id = this.categoryId || generateSimpleId()

        const categoriesValues = {
          color: this.category.color,
          icon: this.category.icon,
          name: this.category.name,
          parentId: this.category.parentId,
          showInLastUsed: this.category.showInLastUsed,
          showInQuickSelector: this.category.showInQuickSelector,
          showStat: true
        }

        db.ref(`users/${uid}/categories/${id}`).set(categoriesValues)

        this.$store.commit('setCategoryEditId', null)
        this.$store.dispatch('setActiveTab', 'categories')

        if (this.$listeners.callback) this.$listeners.callback()
      }
    },

    validateForm () {
      const categories = this.$store.state.categories.items
      if (!this.category.name) {
        this.$notify({
          group: 'main',
          title: '😮',
          text: this.$lang.categories.form.name.error
        })
        return false
      }

      for (const categoryId in categories) {
        if (categories[categoryId].name === this.category.name && categories[categoryId].parentId === this.category.parentId) {
          if (this.categoryId) {
            if (this.categoryId !== categoryId) {
              this.$notify({
                group: 'main',
                title: '😮',
                text: this.$lang.categories.form.name.exist
              })
              return false
            }
          } else {
            this.$notify({
              group: 'main',
              title: '😮',
              text: this.$lang.categories.form.name.exist
            })
            return false
          }
        }
      }

      return true
    }
  }
}
</script>

<template lang="pug">
ComponentWrap
  template(slot="headerLeft")
    template(v-if="!categoryId")
      div Categories
      div {{ $lang.categories.createNewTitle }}
    template(v-else) Categories __ {{ $lang.categories.editTitle }}

  template(slot="content")
    .form
      .form-line._text
        .inputText
          input(
            type="text"
            :placeholder="$lang.categories.form.name.placeholder"
            v-model="category.name"
            v-focus.lazy="$store.state.ui.pc"
          ).inputText__value
          .inputText__label {{ $lang.categories.form.name.label }}

      //- can not change root category if inside this category already has some categories
      template(v-if="$store.getters.getChildCategoriesIds(categoryId).length === 0 && $store.getters.hasCategories")
        .form__btns__i._full
          .form-line(@click="showParents = true")
            .inputModal._flex
              .inputModal__content
                template(v-if="category.parentId !== 0")
                  .inputModal__icon
                    div(
                      :class="$store.state.categories.items[category.parentId].icon"
                      :style="{ color: $store.state.categories.items[category.parentId].color }"
                    )
                  .inputModal__name {{ $store.state.categories.items[category.parentId].name }}
                template(v-else)
                  .inputModal__icon: .mdi.mdi-chart-bubble
                  .inputModal__name {{ $lang.categories.form.parent.no }}
              .inputModal__label {{ $lang.categories.form.parent.label }}

      .form__btns
        .form__btns__i
          .form-line(@click="showColors = true")
            .inputModal._flex
              .inputModal__value: .inputModal__color(:style="{ background: category.color }")
              .inputModal__label {{ $lang.categories.form.color.label }}

        .form__btns__i
          .form-line(@click="showIcons = true")
            .inputModal._flex
              .inputModal__icon: div(:class="category.icon", :style="{ color: category.color }")
              .inputModal__label {{ $lang.categories.form.icon.label }}

      template(v-if="$store.getters.getChildCategoriesIds(categoryId).length === 0")
        .form-line._p0._clean
          Checkbox(
            v-model="category.showInLastUsed"
            :title="$lang.categories.form.lastUsed"
            :alt="true")
        .form-line._p0._clean
          Checkbox(
            v-model="category.showInQuickSelector"
            :title="$lang.categories.form.quickSelector"
            :alt="true")

    //- colors
    ModalBottom(
      :center="true"
      :show="showColors"
      :title="$lang.categories.form.color.placeholder"
      v-on:onClose="showColors = false")
      .inputText
        .inputText__colors
          .colors
            .colorItem(
              :class="{ _active: category.color === color }"
              :style="{ background: color }"
              @click="handleColorSelect(color)"
              v-for="color in colors"
            )
      .customColor
        .customColor__title {{ $lang.categories.form.color.custom }}
        input.customColor__value(v-model="category.color" type="color")

    //- parent
    ModalBottom(
      :center="true"
      :show="showParents"
      :title="$lang.categories.form.parent.label"
      v-on:onClose="showParents = false")
      .padding-bottom
        ModalButton(
          :name="$lang.categories.form.parent.no"
          icon="mdi mdi-chart-bubble"
          v-on:onClick="() => handleParenCategorySelect(0)")
      CategoriesView(
        :noPadding="true"
        :ids="$store.getters.categoriesForBeParent.filter(cId => cId !== categoryId)"
        v-on:onClick="handleParenCategorySelect")

    //- icons
    ModalBottom(
      :center="true"
      :show="showIcons"
      :title="$lang.categories.form.color.placeholder"
      v-on:onClose="showIcons = false")
      .form-line._text
        .inputText
          input.inputText__value(
            type="text"
            placeholder="Filter icons..."
            v-model="filter"
            v-focus.lazy="showIcons")
          .inputText__label Filter
      .inputIcons
        .inputIcons__item(
          :class="{ _active: category.icon === `mdi ${icon}` }"
          @click="handleIconSelect(icon)"
          v-for="icon in selectedIcons")
          div(:class="`mdi ${icon}`")

  template(slot="bottom")
    .col
        Button(
          :class="['_text-center _blue _ml-big', { _inline: $store.state.ui.pc }]"
          :title="$lang.categories.form.save"
          v-on:onClick="handleSubmit")
</template>

<style lang="stylus" scoped>
@import "~@/stylus/variables/margins"
@import "~@/stylus/variables/media"

.customColor
  margin (- $m7)
  margin-top 0
  padding $m7
  background var(--c-bg-1)

  @media $media-laptop
    margin (- $m9)
    margin-top 0
    padding $m9

  &__title
    padding-bottom $m6
    color var(--c-font-4)

  &__value
    width 100%
    height 40px
    margin 0
    padding 0
    border 0

.padding-bottom
  padding-bottom $m8

.form
  @media $media-laptop
    max-width 380px

.form__actions
  @media $media-phone
    text-align center

.form__btns
  display grid
  grid-template-columns repeat(2, 1fr)
  grid-column-gap $m9
  grid-row-gap $m9
  padding-bottom $m9

  @media $media-laptop
    grid-column-gap $m10
    grid-row-gap $m10
    padding-bottom $m10

  &__i._full
    grid-column 1 / -1

  .form-line
    display flex
    align-items center
    justify-content center
    height 100%
    height 56px
    margin-bottom 0

.inputModal
  &._flex
    flex 1
    display flex
    align-items center
    justify-content space-between

  &__content
    display flex

  &__label
    margin 0
    padding 0
    font-size 10px
</style>

<style lang="stylus">
@import "~@/stylus/variables/media"

.dashboard .component
  .categories__list
    @media $media-laptop
      grid-template-columns repeat(3,  1fr)
</style>

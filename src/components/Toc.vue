<template>
  <ul class="sidebar-headings">
    <li
      class="sidebar-heading"
      :class="{'has-children': hasChildren(heading.index), visible: isVisible(heading.level, heading.parent)}"
      v-for="heading in headings"
      :data-level="heading.level">
      <router-link
        exact
        @click.native="navigate(heading.slug)"
        class="sidebar-heading-anchor"
        :class="{active: activeId === heading.slug}"
        :to="{query: {id: heading.slug}}"
        v-html="heading.text">
      </router-link>
    </li>
  </ul>
</template>

<script>
  import {mapState, mapActions} from 'vuex'
  import {$, isMobile} from 'utils/dom'

  export default {
    props: {
      headings: {
        type: Array,
        required: true
      }
    },
    computed: {
      ...mapState(['activeId']),
      visibleBlockIndexes() {
        if (!this.activeId) return []
        const indexes = []

        const active = this.headings.filter(heading => {
          return this.activeId === heading.slug
        })[0]
        if (!active) return []
        indexes.push(active.index)

        const getParent = heading => {
          indexes.push(heading.parent)
          const parent = this.headings.filter(h => {
            return h.index === heading.parent
          })[0]
          if (parent) getParent(parent)
        }

        const parent = this.headings[active.index]
        if (parent) getParent(parent)

        return indexes.filter(i => i >= 0)
      }
    },
    methods: {
      ...mapActions(['jumpToId']),
      hasChildren(index) {
        return this.headings.filter(heading => {
          return heading.parent === index
        }).length > 0
      },
      isVisible(level, index) {
        if (level <= 4) return true
        return this.visibleBlockIndexes.indexOf(index) !== -1
      },
      navigate(slug) {
        this.jumpToId(slug)
      }
    }
  }
</script>

<style>
  .sidebar-headings {
    list-style: none;
    padding: 0 20px;
    margin: 0;
    margin-top: 20px;
    .sidebar-heading {
      line-height: 1.4;
      margin-bottom: 3px;
      &:not(.visible) {
        display: none;
      }
      &[data-level="2"] {
        font-weight: bold;
        margin-bottom: 5px;
        &:not(:first-child) {
          margin-top: 5px;
          padding-top: 5px;
        }
        .sidebar-heading-anchor {
          color: #333;
        }
      }
      &[data-level="4"] {
        padding-left: 15px;
        font-size: 13px;
      }
      &[data-level="5"] {
        padding-left: 30px;
        font-size: 13px;
      }

      .sidebar-heading-anchor {
        color: #666;
        &.active {
          color: #42b983;
        }
      }
    }
  }
  @media screen and (max-width: 768px) {
    .sidebar-headings {
      border-top: 1px solid #e2e2e2;
      padding: 0 10px;
      padding-top: 10px;
      margin-top: 10px;
    }
  }
</style>

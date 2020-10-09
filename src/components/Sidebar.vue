<template>
  <aside :class="{'sidebar--open' : this.$store.state.sidebarOpen}" class="sidebar">
    <nav>
      <ul>
        <li v-for="{ node: menuItem } in $static.menu.edges" :key="menuItem.id" class="section">
          <h3 class="section-title">{{ menuItem.section }}</h3>
          <ul>
            <li v-for="{node: post} in $static.posts.edges" v-if="menuItem.section === 'Blog Stuff'" :key="menuItem.id">
              <a :href="post.path" class="topic">{{ post.title }}</a>
            </li>
          </ul>
        </li>
      </ul>
    </nav>
  </aside>
</template>

<static-query>
query {
menu: allMenu(order:ASC) {
edges {
node {
section
}
}
}
posts: allPost {
edges {
node {
path
categories
title
}
}
}
}
</static-query>

<script>
import GitLink from '~/components/GitLink.vue'
import throttle from 'lodash/throttle'

export default {
  components: {
    GitLink
  },
  watch: {
    '$route' () {
      this.$store.commit('closeSidebar')
    }
  },
  methods: {
    checkAnchors (slug, item) {
      if (slug == item) {
        return true
      }
    },
    stateFromSize: function () {
      if (window.getComputedStyle(document.body, ':before').content == '"small"') {
        this.$store.commit('closeSidebar')
      } else {
        this.$store.commit('openSidebar')
      }
    },
    sidebarScroll: function () {
      let mainNavLinks = document.querySelectorAll('.topic.active + ul .sub-topic')
      let fromTop = window.scrollY

      mainNavLinks.forEach(link => {
        let section = document.querySelector(link.hash)
        let allCurrent = document.querySelectorAll('.current'), i

        if (section.offsetTop <= fromTop) {
          for (i = 0; i < allCurrent.length; ++i) {
            allCurrent[i].classList.remove('current')
          }
          link.classList.add('current')
        } else {
          link.classList.remove('current')
        }
      })
    }
  },
  beforeMount () {
    this.stateFromSize()
  },
  mounted () {
    window.addEventListener('scroll', throttle(this.sidebarScroll, 50))
  }
}
</script>

<style lang="scss" scoped>
.sidebar {
  transition: background .15s ease-in-out, transform .15s ease-in-out, border-color .15s linear;
  padding: 100px 30px 30px;
  width: 300px;
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  z-index: 9;
  will-change: transform;
  transform: translateX(-300px);
  border-right: 1px solid transparent;
  overflow: auto;

  @include respond-above(sm) {
    transform: translateX(0);
  }

  &--open {
    transform: translateX(0);
  }

  .bright & {
    background: $sidebarBright;
    border-color: shade($sidebarBright, 10%);
  }

  .dark & {
    background: $sidebarDark;
    border-color: shade($sidebarDark, 40%);
  }
}

nav {
  position: relative;
  min-height: 100%;
  border: 1px solid transparent;
  padding-bottom: 40px;
}

ul {
  list-style: none;
  padding: 0;
  margin: 0;

  a {
    text-decoration: none;
    color: inherit;
    padding: 5px 0;
    display: block;

    &.active {
      color: $brandPrimary;
    }
  }
}

.section {
  margin-bottom: 30px;
}

.section-title {
  text-transform: uppercase;
  font-size: 12px;
  margin-bottom: 20px;
  opacity: .3;
  letter-spacing: .15em;
  font-weight: 700;
}

.topic {
  font-weight: 600;
}

.sub-topic {
  font-size: .875rem;
  position: relative;
  opacity: .8;

  &::after {
    content: '';
    transition: opacity .15s ease-in-out;
    width: 6px;
    height: 6px;
    background: $brandPrimary;
    border-radius: 100%;
    display: block;
    opacity: 0;
    position: absolute;
    top: 13px;
    left: -15px;
  }

  &.current {
    &::after {
      opacity: 1;
    }
  }
}

.git {
  position: absolute;
  bottom: 0;
  left: 0;
}
</style>



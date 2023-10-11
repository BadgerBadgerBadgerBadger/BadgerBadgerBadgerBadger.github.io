<template>
  <Layout :sidebar="false">
    <div class="content">
      <h1>{{ $static.metadata.siteName }} - {{ this.description }}</h1>
      <nav>
        <Shortcut :link="latestPost.path" icon="book-icon" text="Blog"/>
        <Shortcut :link="latestPost.path" icon="users-icon" text="Hire Me!"/>
      </nav>
      <GitLink class="git" size="large"/>
    </div>
  </Layout>
</template>

<static-query>
query {
metadata: metadata {
siteName
}
topPost: allPost(limit: 1) {
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
import Shortcut from '~/components/Shortcut.vue'

export default {
  components: {
    Shortcut
  },
  data () {
    return {
      description: 'badger badger badger badger'
    }
  },
  computed: {
    latestPost () {
      return this.$static.topPost.edges[0].node
    }
  },
  metaInfo () {
    return {
      title: this.description,
      meta: [
        {
          key: 'description',
          name: 'description',
          content: 'badger badger badger badger'
        }
      ]
    }
  }
}
</script>

<style lang="scss" scoped>
.content {
  display: flex;
  flex-direction: column;
}

h1 {
  text-align: center;
  max-width: 600px;
  margin: 1.5em auto 1.5em;

  @include respond-above(md) {
    max-width: 1000px;
  }
}

h2 {
  font-size: 1.4rem;
  margin: 0;
}

nav {
  display: flex;
  justify-content: space-between;
  flex-direction: column;

  @include respond-above(sm) {
    flex-direction: row;
  }
}

.git {
  margin: 3em 0 0;
  align-self: center;

  @include respond-above(md) {
    margin: 5em 0 0;
  }
}
</style>

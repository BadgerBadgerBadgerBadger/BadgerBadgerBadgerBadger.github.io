<template>
  <Layout>
    <h1>
      {{ $page.post.title }}
    </h1>
    <h3> {{ $page.post.tags.join(' ') }} </h3>
    <div class="markdown" v-html="$page.post.content"/>
  </Layout>
</template>

<page-query>
query Post ($path: String!) {
post: post (path: $path) {
title
path
categories
content
tags
}
}
</page-query>

<script>
export default {
  metaInfo () {
    return {
      title: this.$page.post.title,
      meta: [
        { key: 'description', name: 'description', content: this.$page.post.description }
      ]
    }
  }
}
</script>


<style lang="scss" scoped>
/deep/ > p {
  opacity: .8;
}

/deep/ > h2 {
  padding-top: 100px;
  margin-top: -80px;

  @include respond-above(md) {
    font-size: 2rem;
  }
}

/deep/ > p > img {
  max-width: 100%;
}

.markdown {
  padding-bottom: 50vh;
}
</style>

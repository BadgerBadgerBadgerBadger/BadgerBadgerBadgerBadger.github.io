<template>
  <g-link class="logo" title="Back to home" to="/">
    <transition name="theme">
      <div v-if="color == 'dark'">
        <g-image alt="logo" src="~/assets/img/badger-logo.png"/>
      </div>
    </transition>
    <transition name="theme">
      <div v-if="color == 'bright'">
        <g-image alt="logo" src="~/assets/img/badger-logo.png"/>
      </div>
    </transition>
  </g-link>
</template>

<script>
export default {
  props: {
    color: {
      type: String,
      required: true
    }
  }
}
</script>

<style lang="scss" scoped>
.logo {
  height: 40px;
  display: block;
  transition: color .15s ease-in-out;
  color: $textBright;
  position: relative;
  perspective: 200px;

  div {
    height: 40px;
    position: absolute;
    top: 0;
    left: 0;
  }

  img {
    border-radius: 50%;
    height: 100%;
    width: auto;
  }
}

.theme-enter-active, .theme-leave-active {
  transition: opacity .15s ease-in-out;
}

.theme-enter, .theme-leave-to /* .fade-leave-active below version 2.1.8 */
{
  opacity: 0;
}
</style>



<template>
  <div class="title-faded">
    <h1 v-if="!size || size === 'large'" class="xs-pb2 bold fill-white">{{text}}</h1>
    <h4 v-else-if="size === 'small'" class="xs-pb1 xs-mb0 bold fill-white">{{text}}</h4>
  </div>
</template>

<script>
  export default {
    props: ["text", "size"]
  }
</script>

<style>
  .title-faded {
    background: linear-gradient(to right, rgba(0,0,0,.2), rgba(0,0,0,0));
    padding-bottom: 1px;
  }
</style>
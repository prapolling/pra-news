<template>
  <div class="title-faded">
    <h1 class="xs-pb2 bold fill-white">{{text}}</h1>
  </div>
</template>

<script>
  export default {
    props: ["text"]
  }
</script>

<style>
.title-faded {
  background: linear-gradient(to right, rgba(0,0,0,.2), rgba(0,0,0,0));
  padding-bottom: 1px;
}
</style>
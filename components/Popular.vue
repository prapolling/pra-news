<template>
  <div class='grid-popular'>
    <Title text='Popular' />
    <div v-for='s in popular' :key='s._path'>
      <h3 class='xs-mb2 bold'>{{s.title}}</h3>
      <div v-html='s.body'></div>
      <img v-if='s.featuredImage' class='xs-py1' :src='`${s.featuredImage}`' />
    </div>
  </div>
</template>

<script>
  import Title from '~/components/Title';

  export default {
    data() {
      return {
        popular: this.$store.state.popular,
      };
    },
    components: {
      Title,
    }
  }
</script>

<style>
</style>
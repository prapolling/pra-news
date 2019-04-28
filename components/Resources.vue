<template>
  <div class="grid-resource">
    <Title text="Resources" />
    <div v-for="r in resources" :key="r._path">
      <nuxt-link :to="r._path">
        <h3 class="xs-mb2 bold">{{r.title}}</h3>
      </nuxt-link>
      <div v-if="r.description" v-html="r.description"></div>
      <div v-else v-html="r.body"></div>
    </div>
  </div>
</template>

<script>
  import Title from '~/components/Title';

  export default {
    data() {
      return {
        resources: this.$store.state.resources,
      };
    },
    components: {
      Title,
    }
  }
</script>

<style>
</style>
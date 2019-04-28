<template>
  <div class='xs-text-6 md-text-5' :style='`margin-top:${navbarheight}px`'>
    <div v-if='items2[0]' class='xs-py4 table grid clearfix'>
      <Resources />
      <div class='xs-px4 browse grid-recent'>
        <Title :text='`Recent${type ? ` ${type} ` : ` `}Posts`' />
        <div v-if='items2[pi]' v-for='(p,pi) in items2' :key='p._path' class='item' >
          <h2 class='xs-my2 bold'>{{p.title}}</h2>
          <div v-html='items2[pi].blurb'></div>
          <nuxt-link class='xs-my1 xs-pr2 xs-text-right button button--transparent' :to='p._path'>
            <h4 class='text-red bold'>READ MORE</h4>
          </nuxt-link>
        </div>
      </div>
      <Popular />
    </div>
    <div v-else class='r full-height browse'>
      <div class='xs-p2 c-100 xs-flex xs-flex-align-center xs-flex-justify-center xs-text-center'
        :style='`height:calc(100vh - ${navbarheight}px);margin-top:${navbarheight}px`'>
        <div v-if='total < 1 && !busy'>No Results.</div>
      </div>
    </div>
  </div>
</template>

<script>
  import _ from 'lodash';
  import remark from 'remark';
  import strip from 'strip-markdown';

  import Popular from '~/components/Popular';
  import Resources from '~/components/Resources';
  import Title from '~/components/Title';

  export default {
    props: ['items', 'allitems', 'type'],
    data() {
      return {
        currentPage: null,
        pageNumbers: [],
        pageNumberCount: 0,
        items2: [],
        query: 1,
        busy: false,
        count: 0,
      };
    },
    components: {
      Popular,
      Resources,
      Title,
    },
    methods: {
      pageCheck() {
        if (this.allitems.length > 6) {
          this.$store.commit('paginateOn', true);
          this.$store.commit('resultsLength', this.allitems.length);
        } else if (this.allitems.length < 6) {
          this.$store.commit('paginateOff', false);
        } else {
          this.$store.commit('paginateOff', false);
        }
      },

      loadMore() {
        this.count = this.offset;
        if (this.total > this.count && this.busy == false) {
          this.busy = true;

          const newItems = [];
          for (var i = 0, j = 6; i < j; i++) {
            const api = this.allitems[this.count];

            if (api) {
              newItems.push(api);
            }
            
            this.count++;
          }

          Promise.all(
            _.map(newItems, item => {
              return (
                remark()
                  .use(strip)
                  .process(item.body)
                  .then(stripped => {
                    const blurb = stripped.contents.length > 497
                      ? `${stripped.contents.substr(0, 497)}...`
                      : stripped.contents;

                    return { ...item, blurb };
                  })
              );
            })
          ).then(items2 => {
            this.items2 = items2;
            this.busy = false;
          });
        }
      },

      onResize(event) {
        this.navHeight();
      },

      navHeight() {
        if (process.browser) {
          var height = document.getElementById('navbar').clientHeight;

          this.$store.commit('SET_NAVHEIGHT', height - 1);
        }
      }
    },
    watch: {
      // whenever question changes, this function will run
      $route({
        params,
        query
      }) {
        if (this.$route.query.page > 1) {
          this.loadMore();
          this.navHeight();
          this.pageCheck();
        } else if (this.$route.query.page == null) {
          this.$route.query.page = 1;
          this.loadMore();
          this.navHeight();
          this.pageCheck();
        } else {
          this.loadMore();
          this.navHeight();
          this.pageCheck();
        }
      },
      queryParam: function () {
        this.loadMore();
      }
    },
    computed: {
      offset() {
        if (this.queryParam > 1) {
          return Number(this.queryParam - 1) * 6;
        } else {
          return 0;
        }
      },
      prevpage() {
        const prev = Number(this.queryParam) - 1;
        return prev <= 1 ? 1 : prev;
      },
      nextpage() {
        const next = Number(this.queryParam) + 1;
        return next;
      },
      navbarheight() {
        return this.$store.state.navheight;
      },
      total() {
        return this.allitems.length;
      },

      queryParam() {
        if (this.$route.query.page == null) {
          return 1;
        } else {
          return Number(this.$route.query.page);
        }
      }
    },

    updated() {
      this.$nextTick(() => {
        this.pageCheck();
        this.navHeight();
        this.$store.commit('SET_GRIDOFFSET', this.offset);
      });
    },
    mounted() {
      if (process.browser) {
        this.loadMore();

        this.$nextTick(() => {
          this.navHeight();
          this.pageCheck();
          window.addEventListener('resize', this.onResize);
        });
      }
    },
    beforeDestroy() {
      // Unregister the event listener before destroying this Vue instance
      window.removeEventListener('resize', this.onResize);
    }
  };
</script>

<style>
  .grid {
    width: 1200px;
    max-width: 100%;
    margin: 0 auto;
    display: grid !important;
    grid-template-areas: 'resource recent popular';
    grid-gap: 0;
    align-items: start;
    grid-template-columns: 25% auto 25%;
  }

  .grid-resource {
    grid-area: resource;
  }

  .grid-recent {
    grid-area: recent;
  }

  .grid-popular {
    grid-area: popular;
  }

  @media only screen and (max-width: 80rem) {
    .grid {
      grid-template-areas: 'resource recent' 'popular popular';
      grid-template-columns: 33% auto;
    }

    .grid-resource {
      padding-left: 2rem;
    }

    .grid-popular {
      padding: 0 2rem;
    }
  }

  @media only screen and (max-width: 60rem) {
    .grid {
      grid-template-areas: 'recent' 'popular' 'resource';
      grid-template-columns: 100%;
    }
  }
</style>
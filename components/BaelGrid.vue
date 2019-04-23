<template>
  <div class="xs-text-6 md-text-5" :style="`margin-top:${navbarheight}px`">
    <div v-if="items2[0]" class="xs-py4 table grid clearfix">
      <div class="xs-col-3 xs-float-left">
        <div v-for="r in resources" :key="r.pi">
          <a href="https://www.wikipedia.com" target="_blank"><h3 class="xs-mb2 bold">{{r.title}}</h3></a>
          <div>{{r.body}}</div>
        </div>
      </div>
      <div class="xs-col-6 browse xs-float-left">
        <div v-for="(p,pi) in items2" :key="p.pi" >
          <div v-if="items2[pi]" class="xs-px4 item">
            <h2 class="xs-my2 bold">{{p.title}}</h2>
            <div v-html="p.body"></div>
            <nuxt-link class="xs-my1 xs-pr5 xs-text-right button button--transparent bold" :to="p._path">
              <span class="text-red">READ MORE</span>
            </nuxt-link>
          </div>
        </div>
      </div>
      <div class="xs-col-3 xs-float-left">
        <div v-for="s in stickyPosts" :key="s.pi">
          <h3 class="xs-mb2 bold">{{s.title}}</h3>
          <div>{{s.body}}</div>
          <img class="xs-py1" src={{s.featuredImage}} />
        </div>
      </div>
    </div>
    <div v-else class="r full-height browse">
      <div class="xs-p2 c-100 xs-flex xs-flex-align-center xs-flex-justify-center xs-text-center"
        :style="`height:calc(100vh - ${navbarheight}px);margin-top:${navbarheight}px`">
        <div v-if="total < 1 && !busy">No Results.</div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    props: ["items", "allitems"],
    data() {
      return {
        currentPage: null,
        pageNumbers: [],
        pageNumberCount: 0,
        items2: [],
        query: 1,
        busy: false,
        count: 0,
        resources: this.$store.state.resources,
        stickyPosts: this.$store.state.stickyPosts,
      };
    },
    methods: {
      pageCheck() {
        if (this.allitems.length > 6) {
          this.$store.commit("paginateOn", true);
          this.$store.commit("resultsLength", this.allitems.length);
        } else if (this.allitems.length < 6) {
          this.$store.commit("paginateOff", false);
        } else {
          this.$store.commit("paginateOff", false);
        }
      },

      loadMore() {
        this.count = this.offset;
        if (this.total > this.count && this.busy == false) {
          this.busy = true;
          this.items2.splice(0);

          for (var i = 0, j = 6; i < j; i++) {
            const api = this.allitems[this.count];

            this.items2.push(api);
            this.count++;
          }

          this.busy = false;
        }
      },

      onResize(event) {
        this.navHeight();
      },

      navHeight() {
        if (process.browser) {
          var height = document.getElementById("navbar").clientHeight;

          this.$store.commit("SET_NAVHEIGHT", height - 1);
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
        this.$store.commit("SET_GRIDOFFSET", this.offset);
      });
    },
    mounted() {
      if (process.browser) {
        this.loadMore();

        this.$nextTick(() => {
          this.navHeight();
          this.pageCheck();
          window.addEventListener("resize", this.onResize);
        });
      }
    },
    beforeDestroy() {
      // Unregister the event listener before destroying this Vue instance
      window.removeEventListener("resize", this.onResize);
    }
  };
</script>

<style>
.grid {
  width: 1200px;
  max-width: 100%;
  margin: 0 auto;
}
</style>
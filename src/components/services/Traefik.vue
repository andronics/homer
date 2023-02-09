<template>
  <Generic :item="item">
    <template #content>
      <p class="title is-4">{{ item.name }}</p>
      <p class="subtitle is-6">
        <template v-if="item.subtitle"> {{ item.subtitle }} </template>
        <template v-else-if="item.displayCount">
          {{ entrypointsCount }} Entrypoint<template v-if="entrypointsCount > 1">s</template>,
          {{ routersCount }} Router<template v-if="routersCount > 1">s</template>,
          {{ middlewaresCount }} Middleware<template v-if="middlewaresCount > 1">s</template> &
          {{ servicesCount }} Service<template v-if="servicesCount > 1">s</template>
        </template>
      </p>
    </template>
    <template #indicator>
      <div class="notifs">
        <strong v-if="errorCount > 0" class="notif error" title="Errors">
          {{ errorCount }}
        </strong>
        <strong v-if="warningCount > 0" class="notif warning" title="Warning">
          {{ warningCount }}
        </strong>
      </div>
    </template>
  </Generic>
</template>

<script>
import service from "@/mixins/service.js";
import Generic from "./Generic.vue";

export default {
  name: "Traefik",
  mixins: [service],
  props: {
    item: Object,
  },
  components: {
    Generic,
  },
  data: () => ({
    entrypoints: null,
    overview: null,
  }),
  computed: {
    errorCount: function() {
      
      if (!this.overview) {
        return 0;
      };
      
      return ['http', 'tcp', 'udp'].reduce((pv, i) => {
        return ["middlewares", "routers", "services"].reduce((sv, j) => {
          if (this.overview[i].hasOwnProperty(j)) {
            return sv + this.overview[i][j].errors;
          } else {
            return sv;
          }
        }, pv);
      }, 0);

    },
    warningCount: function() {
      
      if (!this.overview) {
        return 0;
      };
      
      return ['http', 'tcp', 'udp'].reduce((pv, i) => {
        return ["middlewares", "routers", "services"].reduce((sv, j) => {
          if (this.overview[i].hasOwnProperty(j)) {
            return sv + this.overview[i][j].warnings;
          } else {
            return sv;
          }
        }, pv);
      }, 0);

    },
    entrypointsCount: function() {
      
      if (!this.entrypoints) {
        return 0;
      } else {
        return this.entrypoints.length
      };

    },
    middlewaresCount: function() {
      
      if (!this.overview) {
        return 0;
      };
      
      return ['http', 'tcp'].reduce((v, i) => {
        return v + this.overview[i].middlewares.total;
      }, 0);
    
    },
    routersCount: function() {
      
      if (!this.overview) {
        return 0;
      };
    
      return ['http', 'tcp', 'udp'].reduce((v, i) => {
        return v + this.overview[i].routers.total;
      }, 0);
    
    },
    servicesCount: function() {
      
      if (!this.overview) {
        return 0;
      }
      
      return ['http', 'tcp', 'udp'].reduce((v, i) => {
        return v + this.overview[i].services.total;
      }, 0);

    },
  },
  created() {
    
    const updateInterval = parseInt(this.item.updateInterval, 10) || 0;
    
    if (updateInterval > 0) {
      setInterval(() => {
        this.fetchEntrypoints();
        this.fetchOverview();
      }, updateInterval);
    }

    this.fetchEntrypoints();
    this.fetchOverview();

  },
  methods: {
    fetchEntrypoints: async function () {
      this.entrypoints = await this.fetch("/api/entrypoints").catch(
        (e) => {
          console.error(e);
        }
      );
    },
    fetchOverview: async function () {
      this.overview = await this.fetch("/api/overview").catch(
        (e) => {
          console.error(e);
        }
      );
    },
  },
};
</script>

<style scoped lang="scss">
.notifs {
  position: absolute;
  color: white;
  font-family: sans-serif;
  top: 0.3em;
  right: 0.5em;

  .notif {
    display: inline-block;
    padding: 0.2em 0.35em;
    border-radius: 0.25em;
    position: relative;
    margin-left: 0.3em;
    font-size: 0.8em;

    &.success {
      background-color: #4fd671;
    }

    &.warning {
      background-color: #e51111;
    }

    &.error {
      background-color: #2ed0c8;
    }
  }
}
</style>

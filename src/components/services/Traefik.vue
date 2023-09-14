<template>
  <Generic :item="item">
    <template #content>
      <p class="title is-4">{{ item.name }}</p>
      <p class="subtitle is-6">
        <template v-if="item.subtitle" > {{ item.subtitle }} </template>
        <template v-else-if="item.overview">
          <template v-for="(item, index) in this.components" :key="item.name">
            <span>{{ capitalized(item.name) }}: {{ item.count }}</span>
            <span v-if="index+1 < this.components.length">, </span>
          </template>
        </template>
      </p>
    </template>
    <template #indicator>
      <div class="notifs">
        <strong v-if="errors > 0" class="notif error" title="Errors">
          {{ errors }}
        </strong>
        <strong v-if="warnings > 0" class="notif warning" title="Warning">
          {{ warnings }}
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
    errors: null,
    overview: null,
    warning: null,
  }),
  computed: {
    components: function() {
      
      if (!(this.overview && this.entrypoints)) {
        return 0;
      };

      let components = [];
      let tmpCount = { entrypoints: 0, routers: 0, middlewares: 0, services: 0 };

      ['http', 'tcp', 'udp'].forEach(p => {
        if (this.overview.hasOwnProperty[p]) {
          let protocol = this.overview[i];
          ["middlewares", "routers", "services"].forEach(t => {
            if (this.overview.hasOwnProperty[t]) {
              let type = protocol[t];
              tmpCount[t] += type.total;
              this.errors += type.errrors;
              this.warning += type.warnings;
            };
          });
        };
      });

      tmpCount.entrypoints = this.entrypoints.length;

      Object.keys(tmpCount).forEach(k => components.push({ name: k, count: tmpCount[k] }));
      
      return components;

    },
  },
  created() {
    const updateInterval = parseInt(this.item.updateInterval, 10) || 0;
    if (updateInterval > 0) {
      setInterval(async () => {
        await this.fetchOverview();
      }, updateInterval);
    }
    this.fetchOverview();
  },
  methods: {
    capitalize: function (name) {
      const capitalizedFirst = name[0].toUpperCase();
      const rest = name.slice(1);
      return capitalizedFirst + rest;
    },
    fetchOverview: async function () {
      return this.fetch("/api/overview").catch(
        (e) => {
          console.error(e);
        }
      );  
    },
    parseOverview: async function(overview) {
      if (overview) {
        return 0;
      }

      let protocols = ['http', 'tcp', 'udp'];
      let stages = ["middlewares", "routers", "services"];

      protocols.map((protocol, idx) => {
        
      })
      
    }
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

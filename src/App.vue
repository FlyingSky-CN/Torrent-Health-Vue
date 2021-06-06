<template>
  <v-app style="background-color: #eee">
    <v-main>
      <v-container>
        <div style="background: none; margin-bottom: 32px; margin-top: 128px;">
          <div class="text-h3">Torrent Health Tracker</div>
          <div class="text-h5" style="padding-top: 8px;">Sci-Hub / Libgen / <a style="text-decoration: none"
              href="https://phillm.net/torrent-health-frontend/stats-filtered-table.php?propname%5B%5D=seeders&comp%5B%5D=%3C&value%5B%5D=7&propname%5B%5D=type&comp%5B%5D===&value%5B%5D=scimag">Data Upstream</a> / <a href="https://zhuanlan.zhihu.com/p/376398943" style="text-decoration: none">About</a> / Count: {{table.length}}
          </div>
        </div>
        <v-card class="elevation-6" style="margin-bottom: 32px; border-radius: 8px;">
          <v-tabs v-model="tab" centered dense>
            <v-tabs-slider></v-tabs-slider>
            <v-tab href="#tab-1">
              Link 1
            </v-tab>
            <v-tab href="#tab-2">
              Link 2
            </v-tab>
            <v-tab href="#tab-3">
              Hash
            </v-tab>
          </v-tabs>
          <v-tabs-items v-model="tab" style="min-height: 128px;">
            <v-tab-item key="1" value="tab-1">
              <v-card flat>
                <v-card-text v-html="listData(1)" style="color: rgba(0, 0, 0, 0.87);"></v-card-text>
              </v-card>
            </v-tab-item>
            <v-tab-item key="2" value="tab-2">
              <v-card flat>
                <v-card-text v-html="listData(2)" style="color: rgba(0, 0, 0, 0.87);"></v-card-text>
              </v-card>
            </v-tab-item>
            <v-tab-item key="3" value="tab-3">
              <v-card flat>
                <v-card-text v-html="listData(3)" style="color: rgba(0, 0, 0, 0.87);"></v-card-text>
              </v-card>
            </v-tab-item>
          </v-tabs-items>
        </v-card>
        <v-card class="elevation-6" style="margin-bottom: 32px; border-radius: 8px;padding: 16px;" v-if="selected.length == 0">
          Select items below to get start.
        </v-card>
        <v-data-table v-model="selected" :headers="headers" :items="table" item-key="name" :loading="loading"
          show-select calculate-widths fixed-header class="elevation-6 text-truncate"
          style="margin-bottom: 32px; border-radius: 8px;">
          <template v-slot:item.size_bytes="{ item }">
            {{ (item.size_bytes/1073741824).toFixed(2) + ' GB'}}
          </template>
          <template v-slot:item.infohash="{ item }">
            <a :href="'https://phillm.net/torrent-health-frontend/stats-hash-table.php?hash='+item.infohash"
              target="_blank" style="text-decoration: none"><code>{{ item.infohash.substr(0,6) }}</code></a>
          </template>
          <template v-slot:item.link="{ item }">
            <a :href="item.link.replace('gen.lib.rus.ec','libgen.st')" target="_blank"
              style="text-decoration: none">libgen.st</a>
            &nbsp;&nbsp;
            <a :href="item.link" target="_blank" style="text-decoration: none">gen.lib.rus.ec</a>
          </template>
        </v-data-table>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
  export default {
    name: 'App',

    data: () => ({
      loading: false,
      error: false,
      table: [],
      tab: '',
      headers: [
        { text: 'Name', value: 'name' },
        { text: 'Size', value: 'size_bytes', align: 'right' },
        { text: 'Hash', value: 'infohash' },
        { text: 'Torrent Link', value: 'link' },
        { text: 'Seeders', value: 'seeders', align: 'right' },
        { text: 'Leechers', value: 'leechers', align: 'right' },
        { text: 'Scraped', value: 'scraped_date', align: 'right' },
        { text: 'DHT Peers', value: 'dht_peers', align: 'right' },
        { text: 'DHT Scraped', value: 'dht_scraped', align: 'right' },
        { text: 'Type', value: 'type', align: 'right' },
        { text: 'Last Updated', value: 'created_unix', align: 'right' },
      ],
      selected: []
    }),

    methods: {
      updateData() {
        this.loading = true
        this.$axios
          .get('https://phillm.net/torrent-health-frontend/stats-filtered.php?propname%5B%5D=seeders&comp%5B%5D=%3C&value%5B%5D=7&propname%5B%5D=type&comp%5B%5D===&value%5B%5D=scimag')
          .then(response => {
            this.table = []
            for (var i in response.data) {
              this.table.push(response.data[i])
            }
            this.loading = false
            this.error = false
            console.info('[Data Updated]', this.table)
          })
          .catch(error => (
            this.loading = false,
            this.error = true,
            console.error('[Axios Exception]', error)
          ));
      },
      listData(n) {
        var text = ''
        for (var i in this.selected) {
          switch (n) {
            case 1:
              text = text + this.table[i].link.replace('gen.lib.rus.ec', 'libgen.st') + "<br/>"
              break;
            case 2:
              text = text + this.table[i].link + "<br/>"
              break;
            case 3:
            default:
              text = text + this.table[i].infohash + "<br/>"
              break;
          }
        }
        return text
      }
    },

    mounted() {
      this.updateData()
    }
  };
</script>

<style>
  body::-webkit-scrollbar {
    width: 0;
    height: 0;
  }
</style>
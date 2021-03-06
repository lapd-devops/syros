<template>
<div>
  <div>
    <ol class="breadcrumb">
      <li><router-link class="text-uppercase" :to="{ name: 'home'}">home</router-link></li>
      <li>Docker Hosts</li>
    </ol>
  </div>
  <div class="stats">
    <div class="row">
      <div class="col-md-3 text-center">
        <h2>{{ stats.hosts }}</h2><small class="text-uppercase">Docker Hosts</small></div>
      <div class="col-md-3 text-center">
        <h2>{{ stats.containers }}</h2><small class="text-uppercase">Containers up</small></div>
      <div class="col-md-3 text-center">
        <h2>{{ stats.cpus }}</h2><small class="text-uppercase">vCPUs</small></div>
      <div class="col-md-3 text-center">
        <h2>{{ stats.ram }}</h2><small class="text-uppercase">Memory</small></div>
    </div>
  </div>
  <v-client-table ref="hostsTabel" :data="tableData" :columns="columns" :options="options"></v-client-table>  
</div>
</template>

<script>
  import Vue from 'vue'
  import bus from 'components/bus.vue'
  import rowTemplate from 'components/hosts/row.template.jsx'
  import rowChild from 'components/hosts/row-child.template.jsx'

  export default {
    name: 'hosts',
    data () {
      return {
        timer: null,
        stats: {hosts: '0', containers: '0', cpus: '0', ram: '0 MB'},
        columns: ['name', 'status', 'containers_running', 'ncpu', 'mem_total', 'system_time'],
        tableData: [],
        options: {
          skin: 'table-hover',
          sortable: ['name', 'status', 'containers_running', 'ncpu', 'mem_total', 'system_time'],
          dateColumns: ['system_time'],
          toMomentFormat: 'YYYY-MM-DDTHH:mm:ssZ',
          uniqueKey: 'id',
          orderBy: {column: 'name', ascending: true},
          perPage: 10,
          perPageValues: [10, 20, 30, 50],
          childRow: rowChild,
          templates: rowTemplate
        }
      }
    },
    methods: {
      loadData () {
        this.$Progress.start()
        Vue.$http.get('/docker/hosts')
          .then((response) => {
            if (response != null) {
              this.tableData = response.data
              var statsHosts = 0
              var statsContainers = 0
              var statsCpus = 0
              var statsRam = 0
              for (var i = 0, len = response.data.length; i < len; i++) {
                statsHosts += 1
                statsContainers += response.data[i].containers_running
                statsCpus += response.data[i].ncpu
                statsRam += response.data[i].mem_total
              }
              this.stats = {
                hosts: statsHosts.toString(),
                containers: statsContainers.toString(),
                cpus: statsCpus.toString(),
                ram: parseInt(parseFloat((statsRam / Math.pow(1024, 3))).toFixed(0)) + 'GB'
              }
              this.$Progress.finish()
            } else {
              this.$Progress.fail()
            }
          })
          .catch((error) => {
            if (!error.response) {
              bus.$emit('flashMessage', {
                type: 'warning',
                message: 'Network error! Could not connect to the server'
              })
            } else {
              bus.$emit('flashMessage', {
                type: 'warning',
                message: `${error.response.statusText}! ${error.response.data}`
              })
            }
            this.$Progress.fail()
          })
      },
      refreshData () {
        this.loadData()
        console.log('Refresh data: ' + this.$options.name)
        // enqueue new call after 30 seconds
        if (this.timer) clearTimeout(this.timer)
        this.timer = setTimeout(this.refreshData, 30000)
      }
    },
    created: function () {
      console.log('Created: ' + this.$options.name)
    },
    mounted: function () {
      console.log('Mounted: ' + this.$options.name)
      this.refreshData()

      // setTimeout(
      //   () => {
      //     bus.$emit('flashMessage', {
      //       type: 'warning',
      //       message: 'testing a very loooooooong warning message'
      //     })
      //   },
      //   2500
      // )
    },
    destroyed: function () {
      if (this.timer) {
        clearTimeout(this.timer)
        console.log('Destroyed: ' + this.$options.name)
      }
    }
}

</script>

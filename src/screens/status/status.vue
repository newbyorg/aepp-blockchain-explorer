<template>
  <div class="status-screen screen">
    <div>
      <h1>Status</h1>
      <p>
      Explorer connected to: <strong>{{node}}</strong>
      </p>
      <h2>Node and Peers</h2>
      <table>
        <tr>
          <th>address</th>
          <th>top.height</th>
          <th>top.hash </th>
          <th>top.time </th>
          <th>version.revision</th>
          <th>version.genesis_hash</th>
        </tr>
        <tr v-if='apiNodeTop && apiNodeVersion'>
          <td><strong>{{node}}</strong></td>
          <td>{{apiNodeTop.height}}</td>
          <td>{{apiNodeTop.hash | startAndEnd }}</td>
          <td>{{apiNodeTop.time}}</td>
          <td>{{apiNodeVersion.revision | startAndEnd }}</td>
          <td>{{apiNodeVersion.genesis_hash | startAndEnd}}</td>
        </tr>
        <tr v-for='p in apiPeers'>
          <template v-if='p !== null'>
          <td>{{p.address}}</td>
          <td>{{p.top.height}}</td>
          <td>{{p.top.hash | startAndEnd }}</td>
          <td>{{p.top.time}}</td>
          <td>{{p.version.revision | startAndEnd }}</td>
          <td>{{p.version.genesis_hash | startAndEnd}}</td>
          </template>
        </tr>
      </table>
      <h1>Detail</h1>
      <h2>{{node}} </h2>
      <!--<h3>epoch.yaml</h3>-->
      <!--<pre>{{apiNodeConf}}</pre>-->
      <h3>version</h3>
      <pre>{{apiNodeVersion}}</pre>
      <h3>top</h3>
      <pre>{{apiNodeTop}}</pre>

      <h2>Peers</h2>
      <pre>{{apiPeers}}</pre>
    </div>
  </div>
</template>
<script>
export default {
  data () {
    return {
      apiNodeConf: null,
      apiNodeVersion: null,
      apiNodeTop: null,
      apiPeers: []
    }
  },
  computed: {
    node () {
      return this.$http.options.root
    }
  },
  methods: {
    getPeerTop (i) {
      return new Promise((resolve, reject) => {
        this.$http.get(`peer/${i + 1}/v2/top`).then(resp => {
          resolve({
            address: resp.headers.get('X-Upstream'),
            body: resp.body
          })
        }, resp => {
          reject(new Error('fail'))
        })
      })
    },
    getPeerVersion (i) {
      return new Promise((resolve, reject) => {
        this.$http.get(`peer/${i + 1}/v2/version`).then(resp => {
          resolve({
            address: resp.headers.get('X-Upstream'),
            body: resp.body
          })
        }, resp => {
          reject(new Error('fail'))
        })
      })
    },
    getPeers (n) {
      this.apiPeers = new Array(n).fill(null)
      for (var i = 0; i < n; i++) {
        Promise.all([i, this.getPeerTop(i), this.getPeerVersion(i)])
          .then((values) => {
            if (values[1].address !== values[2].address) {
              return
            }
            this.$set(this.apiPeers, values[0], {
              address: values[1].address,
              top: values[1].body,
              version: values[2].body
            })
          })
      }
    }
  },
  created () {
    this.getPeers(3)
    this.$http.get('node/epoch.yaml').then(resp => {
      this.apiNodeConf = resp.body
    }, resp => {
    })
    this.$http.get('external/v2/top').then(resp => {
      this.apiNodeTop = resp.body
    }, resp => {
    })
    this.$http.get('external/v2/version').then(resp => {
      this.apiNodeVersion = resp.body
    }, resp => {
    })
  }
}
</script>
<style src='./status.scss' lang='scss' />

<template>
  <div class="row">
    <div class="col-md">
      <div class="card">
      <div class="card-body">
        <div class="row">
          <div class="col-md-6">
            <form v-on:submit.prevent="onSubmit">
              <div class="form-row">
                <div class="col-md-8 mb-3">
                  <label for="keyword">Keyword</label>
                  <input type="text" class="form-control" id="keyword" v-model="keyword" placeholder="Keyword" required>
                </div>
              </div>
              <button class="btn btn-primary" type="submit">Crawl Tweets</button>
            </form>
          </div>
          <div class="col-md">
              <div class="progress"  v-show="getTweetResponse.isStreaming">
                <div class="progress-bar progress-bar-striped progress-bar-animated" role="progressbar" aria-valuemax="100" style="width: 100%"></div>
              </div>
              <p></p>
              <button type="button" class="btn btn-secondary" v-on:click="stopCrawling()"  v-show="getTweetResponse.isStreaming">STOP</button>
              <a :href="getTweetResponse.link" class="btn btn-outline-primary" target="_blank" v-show="getTweetResponse.link">Download Full Tweet CSV</a>
          </div>
        </div>
        <div class="row mt-4">
          <div class="col-md-12">
            <div class="table-responsive">
              <table id="table-tweet" class="table table-striped display">
                <caption>Tweets</caption>
                  <thead>
                    <tr>
                      <th scope="col">#</th>
                      <th scope="col">ID</th>
                      <th scope="col">UserName</th>
                      <th scope="col">CreatedAt</th>
                      <th scope="col">Language</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(item, index) in getTweets" v-bind:key="index">
                      <th scope="row">{{index}}</th>
                      <td>{{item.ID}}</td>
                      <td>{{item.UserName}}</td>
                      <td>{{item.CreatedAt}}</td>
                       <td>{{item.Lang}}</td>
                    </tr>
                  </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HomePage',
  data () {
    return {
      keyword: ''
    }
  },
  methods: {
    onSubmit () {
      this.$store.dispatch('tweet/crawlTweets', {keyword: this.keyword})
      this.interval = setInterval(function () {
        this.loadCsv()
      }.bind(this), 800)
    },
    stopCrawling () {
      clearInterval(this.interval)
      this.$store.dispatch('tweet/stopCrawling')
      $('#table-tweet').DataTable({
        dom: 'Bfrtip',
        buttons: [
          'copy', 'csv', 'excel', 'pdf', 'print'
        ]
      })
    },
    loadCsv () {
      this.$store.dispatch('tweet/loadCsv')
    }
  },
  computed: {
    getTweetResponse () {
      return this.$store.getters['tweet/getTweetResponse']
    },
    getTweets () {
      return this.$store.getters['tweet/getTweets']
    }
  }
}
</script>

<style scoped>
  a:hover {
    color: white!important;
  }
</style>

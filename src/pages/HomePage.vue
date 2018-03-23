<template>
  <div class="row">
    <div class="col-md-12">
       <form v-on:submit.prevent="onSubmit">
      <div class="row">
          <div class="col-md-6">
                <div>
                  <label for="keyword">Keyword</label>
                  <input type="text" class="form-control" id="keyword" v-model="payload.keyword" placeholder="Keyword" />
                </div>
                <div>
                  <label for="language">Language</label>
                  <input type="text" class="form-control" id="language" v-model="payload.language" placeholder="Language" />
                </div>
                <div>
                  <label for="userId">UserID</label>
                  <input type="text" class="form-control" id="userId" v-model="payload.userId" placeholder="UserID" />
                </div>
          </div>
          <div class="col-md-6">
              <div id="googleMap"></div>
              <input type="text" hidden class="form-control" id="location" v-model="payload.location" placeholder="Location" />
          </div>
        </div>
           <div class="row mt-4">
            <div class="col-md-12">
              <p>
                 <button class="btn btn-primary" type="submit">Crawl Tweets</button>
              </p>
                  <div>
                      <div class="progress"  v-show="getTweetResponse.isStreaming">
                        <div class="progress-bar progress-bar-striped progress-bar-animated" role="progressbar" aria-valuemax="100" style="width: 100%"></div>
                      </div>
                      <p></p>
                      <button type="button" class="btn btn-secondary" v-on:click="stopCrawling()"  v-show="getTweetResponse.isStreaming">STOP</button>
                      <a :href="getTweetResponse.link" class="btn btn-outline-primary" target="_blank" v-show="getTweetResponse.link">Download Full Tweet CSV</a>
                  </div>
            </div>
           </div>
        </form>
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
</template>

<script>
export default {
  name: 'HomePage',
  data () {
    return {
      payload: {
        keyword: '',
        language: '',
        location: '',
        userId: ''
      },
      isHasData: false
    }
  },
  mounted () {
    this.initMap()
  },
  methods: {
    onSubmit () {
      this.$store.dispatch('tweet/crawlTweets', this.payload)
      this.interval = setInterval(function () {
        this.loadCsv()
      }.bind(this), 800)
      if (this.table) {
        this.table.destroy()
      }
    },
    stopCrawling () {
      clearInterval(this.interval)
      this.$store.dispatch('tweet/stopCrawling')
      this.table = $('#table-tweet').DataTable({
        dom: 'Bfrtip',
        buttons: [
          'copy', 'csv', 'excel', 'pdf', 'print'
        ]
      })
    },
    loadCsv () {
      this.$store.dispatch('tweet/loadCsv')
    },
    initMap () {
      let mapProp = {
        center: new google.maps.LatLng(1.3413, 103.9638),
        zoom: 10
      }
      let map = new google.maps.Map(document.getElementById('googleMap'), mapProp)
      let marker = null
      google.maps.event.addListener(map, 'click', function (event) {
        console.log(event.latLng.lat() + ',' + event.latLng.lng())
        if (marker) {
          marker.setMap(null)
        }
        marker = new google.maps.Marker({
          position: event.latLng,
          map: map
        })
        map.panTo(event.latLng)
      })
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
  #googleMap {
    width: 100%;
    height: 200px;
  }
</style>

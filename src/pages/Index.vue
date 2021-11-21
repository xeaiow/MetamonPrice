<template>
  <q-page>
    <q-header>
      <q-toolbar>
        <q-toolbar-title>元獸挑選系統</q-toolbar-title>
        <q-btn glossy icon="sort" :label="sort === 'score' ? '按總分排序' : '按價格排序'" @click="sortBy" />
      </q-toolbar>
    </q-header>
    <div class="row">
      <div class="col-md-4 offset-md-4 col-sm-12 col-xs-12">
        <q-list bordered class="rounded-borders">
          <q-item clickable v-ripple v-for="(info, index) in sortByscore" :key="index" @click="open(info)">
            <q-item-section avatar>
              <q-avatar size="128px">
                <img :src="info.image_url">
              </q-avatar>
            </q-item-section>
            <q-item-section class="q-ml-lg">
              <q-item-label caption lines="2">
                <q-icon size="sm" name="img:icons/raca.png" />
                <span class="text-weight-bold text-h6 q-ml-sm">{{ formatPrice(info.fixed_price) }}</span>
              </q-item-label>
              <q-item-label>
                <span class="text-h6">Score: {{ info.score }}</span>
              </q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'
import _ from 'lodash'

export default defineComponent({
  name: 'Index',

  data () {
    return {
      metamon: [],
      sort: 'score'
    }
  },
  methods: {
    getMetamon () {
      this.$axios.get('https://market-api.radiocaca.com/nft-sales?pageNo=1&pageSize=10&sortBy=created_at&name=&order=desc&saleType&category=13&tokenType')
        .then(res => {
          if (res.data.code !== 200) return
          this.metamon = res.data.list
          this.getScore()
        })
    },
    formatPrice (value) {
      return new Intl.NumberFormat().format(value)
    },
    getScore () {
      this.metamon.forEach((item, index) => {
        this.$axios.get(`https://market-api.radiocaca.com/nft-sales/${item.id}`)
          .then(res => {
            if (res.data.code !== 200) return
            this.metamon[index].score = res.data.data.properties[9].value
          })
      })
      console.log(this.score)
    },
    open (info) {
      window.open(`https://market.radiocaca.com/#/market-place/${info.id}`)
    },
    sortBy () {
      if (this.sort === 'score') {
        this.sort = 'price'
      } else {
        this.sort = 'score'
      }
    }
  },
  computed: {
    sortByscore () {
      if (this.sort === 'price') {
        return _.sortBy(this.metamon, function (obj) {
          return parseInt(obj.fixed_price, 10)
        })
      }

      return _.orderBy(this.metamon, ['score'], ['desc'])
    }
  },
  created () {
    this.getMetamon()
  }
})
</script>

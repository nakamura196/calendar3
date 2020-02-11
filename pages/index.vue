<template>
  <div>
    <v-container class="mt-2 mb-5">
      {{ data }}
      <!--
      <SearchForm
        :q="q"
        :collections="collections"
        :collections_query="index.collections ? Object.keys(index.collections) : []"
        :u="u"
        :search_place_holder="search_place_holder"
      />
      -->

      <v-card class="mt-5">
        <v-card-text>
          <v-simple-table>
            <template v-slot:default>
              <tbody>
                <tr v-for="(item, index2) in items" :key="index2">
                  <th>{{ item.year }} ({{ item.wareki }})</th>
                  <td v-for="index in 12" :key="index" class="text-xs-right">
                    <!-- <a :href="'list?q='+q+'&date=' + props.item.year + '-' + index + '&type=' + type" v-if="props.item.month[index-1] > 0">{{index}}月 ({{ props.item.month[index-1] }})</a> -->
                    <router-link
                      v-if="item.month[index-1] > 0"
                      :to="{ path : 'item', query : { param : JSON.stringify({q: q, collections: collections}), u: u, date: item.year + '-' + index }}"
                    >
                      {{ index }}月 ({{ item.month[index-1] }})
                    </router-link>
                    <span v-else>{{ index }}月</span>
                  </td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
        </v-card-text>
      </v-card>
    </v-container>
  </div>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator'

function toWareki (y: number): string {
  let s: string = ''
  if (y > 1988) {
    s = '平成' + (y - 1988)
  } else if (y > 1925) {
    s = '昭和' + (y - 1925)
  } else if (y > 1911) {
    s = '大正' + (y - 1911)
  } else if (y > 1867) {
    s = '明治' + (y - 1867)
  }
  return s
}

@Component({
  components: {
  },
  /*
  async asyncData (context) {
    const res = await context.$axios.get(
      'https://webpark5032.sakura.ne.jp/tmp/calendar.json',
      {
        headers: {
          Authorization: 'Bearer 準備で発行した自分のトークン'
        }
      }
    )
    return {
      data: res.data
    }
  },
  */
  head (): object {
    return {
      titleTemplate: null,
      title: 'なかむらさとる'
    }
  }
})
export default class IndexPage extends Vue {
  /*
  async asyncData (context: any) {
    const res = await context.$axios.get('https://webpark5032.sakura.ne.jp/tmp/calendar.json', {
      headers: {
        // Authorization: 'Bearer xxxxxxxx'
      }
    })
    return {
      aaa: 'uuu',
      data: res.data
    }
  }
  */

  returnUrl: string = '';
  returnLabel: string = ''
  header: string = ''
  footer: string = ''
  items: Array<string> =[]
  dataAll: Array<string> = []
  q: string = ''
  u: string = ''
  description: string = 'test'
  searchPlaceHolder: string = ''
  index: Object = {}
  collections: Array<string> = []
  test:Array<number> = [1, 3, 4]
  ip: string = ''
  data: any = {}

  async fetchSomething (url: string) {
    const data = await this.$axios.$get(url)
    console.log(data)
    this.data = data
  }

  created () {
    const param: any = Object.assign({}, this.$route.query)
    console.log(param)

    this.u = param.u ? param.u : 'https://webpark5032.sakura.ne.jp/tmp/calendar.json'

    console.log(toWareki(2020))

    this.fetchSomething(this.u)

    /*
    this.$axios.$get(this.u).then((response: any) => {
      const result: Object = response.data
      console.log(result)
    })
    */

    /*
    const data:any = this.fetchSomething()
    console.log(data)
    */

    // console.log(this.data)
    /*
    console.log(this.data)
    console.log($data)
    console.log(data)
    console.log(this.description)
    */

    /*
    if (!param.u) {
      location.href = 'https://github.com/nakamura196/calendar'
    }
    this.u = param.u

    if (param.param) {
      const query = JSON.parse(param.param)

      this.q = query.q ? query.q : this.q
      this.collections = query.collections
        ? query.collections
        : this.collections
    }
    */
  }

  mounted () {
    // 何か処理
  }

  updated () {
    // 何か処理
  }

  destroyed () {
    // 何か処理
  }
}
</script>

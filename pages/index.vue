<template>
  <div>
    <v-container class="mt-2 mb-5">
      <!--
      <SearchForm
        :q="q"
        :collections="collections"
        :collections_query="index.collections ? Object.keys(index.collections) : []"
        :u="u"
        :searchPlaceHolder="searchPlaceHolder"
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
  }
})
export default class IndexPage extends Vue {
  head () {
    return {
      titleTemplate: null,
      title: 'なかむらさとる'
    }
  }

  returnUrl: string = '';
  returnLabel: string = ''
  header: string = ''
  footer: string = ''
  items: any[] =[]
  dataAll: Array<string> = []
  q: string = ''
  u: string = ''
  description: string = 'test'
  searchPlaceHolder: string = ''
  index: any = {}
  collections: Array<string> = []
  test:Array<number> = [1, 3, 4]
  ip: string = ''
  data: any = {}

  search () {
    const data = this.filter()

    const items: any[] = []
    this.items = items

    let minYear = 2000
    let maxYear = 0
    const map: any = {}

    for (let i = 0; i < data.length; i++) {
      const obj: any = data[i]

      const date: string = obj.date
      const c = 1

      const tmp = date.split('-')
      const year = Number(tmp[0])

      if (year < minYear) {
        minYear = year
      }

      if (year > maxYear) {
        maxYear = year
      }

      const month = Number(tmp[1])

      if (!map[year]) {
        map[year] = {}
      }

      const tmp2 = map[year]

      if (!tmp2[month]) {
        tmp2[month] = 0
      }

      tmp2[month] += c
    }

    for (let year = minYear; year <= maxYear; year++) {
      const monthes: number[] = []
      const obj = {
        year,
        wareki: toWareki(year),
        month: monthes
      }

      const tmp = map[year]

      for (let month = 1; month <= 12; month++) {
        const value: any = tmp != null && tmp[month] != null ? tmp[month] : 0
        obj.month.push(value)
      }

      items.push(obj)
    }
  }

  filter () {
    const index = this.index

    console.log(index)

    let collectionIndex: any[] = []
    let fulltextIndex: any[] = []

    const collections = this.collections

    if (collections.length === 0) {
      for (const key in index.collections) {
        collectionIndex = collectionIndex.concat(index.collections[key])
      }
    } else {
      for (let i = 0; i < collections.length; i++) {
        const collection = collections[i]
        const indexArr = index.collections[collection]
        collectionIndex = collectionIndex.concat(indexArr)
      }
    }

    const q = this.q
    if (!q) {
      for (const key in index.fulltext) {
        fulltextIndex = fulltextIndex.concat(index.fulltext[key])
      }
    } else {
      for (const key in index.fulltext) {
        if (key.includes(q)) {
          fulltextIndex = fulltextIndex.concat(index.fulltext[key])
        }
      }
    }

    const x = new Set(collectionIndex)
    const y = new Set(fulltextIndex)
    const intersection = new Set([...x].filter(e => y.has(e))) // 2, 3

    const data = []
    for (const value of intersection) {
      data.push(this.dataAll[value])
    }

    return data
  }

  async main (url: string) {
    const result = await this.$axios.$get(url)

    this.header = result.header
    this.footer = result.footer
    this.returnUrl = result.returnUrl
    this.returnLabel = result.returnLabel

    this.description = result.description
    this.searchPlaceHolder = result.searchPlaceHolder

    const data = result.data
    this.dataAll = data

    const index: any = {
      fulltext: {}
    }

    for (let i = 0; i < data.length; i++) {
      const obj = data[i]
      let fulltext = ''

      for (const key in obj) {
        if (!index[key]) {
          index[key] = {}
        }
        let values = obj[key]
        if (!(Array.isArray(values))) {
          values = [values]
        }
        for (let j = 0; j < values.length; j++) {
          const value = values[j]

          // URIの場合は無視
          if (value.startsWith('http')) {
            continue
          }

          fulltext += value + ' '
          if (!index[key][value]) {
            index[key][value] = []
          }
          index[key][value].push(i)
        }
      }
      index.fulltext[fulltext] = [i]
    }

    this.index = index

    this.search()
  }

  created () {
    const param: any = Object.assign({}, this.$route.query)

    this.u = param.u ? param.u : 'https://webpark5032.sakura.ne.jp/tmp/calendar.json'

    if (param.param) {
      const query: any = JSON.parse(param.param)

      this.q = query.q ? query.q : this.q
      this.collections = query.collections
        ? query.collections
        : this.collections
    }

    console.log(toWareki(2020))

    this.main(this.u)

    /*
    this.$axios.$get(this.u).then((response: any) => {
      const result: Object = response.data
      console.log(result)
    })
    */

    /*
    const data:any = this.main()
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

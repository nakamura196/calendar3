
<template>
  <div>
    <Header
      :header="header"
      :url="returnUrl"
      :label="returnLabel"
      :u="u"
      :description="description"
    />

    <v-container class="mt-2 mb-5">
      <SearchForm
        :q="q"
        :collections="collections"
        :collections_query="index.collections ? Object.keys(index.collections) : []"
        :u="u"
        :search-place-holder="searchPlaceHolder"
      />

      <v-row class="fill-height mt-5">
        <v-col>
          <v-sheet height="64">
            <v-toolbar flat color="white">
              <v-btn fab text small @click="prev">
                <v-icon small>
                  mdi-chevron-left
                </v-icon>
              </v-btn>
              <v-btn fab text small @click="next">
                <v-icon small>
                  mdi-chevron-right
                </v-icon>
              </v-btn>
              <v-toolbar-title class="ml-5">
                {{ getTitle }}
              </v-toolbar-title>
              <v-spacer />
              <v-menu bottom right>
                <template v-slot:activator="{ on }">
                  <v-btn outlined v-on="on">
                    <span>{{ typeToLabel[type] }}</span>
                    <v-icon right>
                      mdi-menu-down
                    </v-icon>
                  </v-btn>
                </template>
                <v-list>
                  <v-list-item @click="type = 'day'">
                    <v-list-item-title>Day</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="type = 'week'">
                    <v-list-item-title>Week</v-list-item-title>
                  </v-list-item>
                  <v-list-item @click="type = 'month'">
                    <v-list-item-title>Month / 月</v-list-item-title>
                  </v-list-item>
                  <!--
                  <v-list-item @click="type = '4day'">
                    <v-list-item-title>4 days</v-list-item-title>
                  </v-list-item>
                  -->
                </v-list>
              </v-menu>
            </v-toolbar>
          </v-sheet>
          <v-sheet height="600">
            <v-calendar
              ref="calendar"
              v-model="focus"
              color="primary"
              :events="events"
              :event-margin-bottom="3"
              :type="type"
              locale="ja"
              @click:event="showEvent"
              @click:more="viewDay"
              @click:date="viewDay"
              @change="updateRange"
            />
            <v-menu
              v-model="selectedOpen"
              :close-on-content-click="false"
              :activator="selectedElement"
              offset-x
            >
              <v-card>
                <v-card-text>
                  <v-row>
                    <v-col>
                      <a target="_blank" :href="selectedEvent.url">
                        <v-img :src="selectedEvent.thumbnail" :lazy-src="selectedEvent.thumbnail" />
                      </a>
                    </v-col>
                    <v-col>
                      <p>
                        <a target="_blank" :href="selectedEvent.url">
                          {{ selectedEvent.name }}
                          <i class="fas fa-external-link-alt" />
                        </a>
                      </p>
                      {{ selectedEvent.description }}
                    </v-col>
                  </v-row>
                </v-card-text>
              </v-card>
            </v-menu>
          </v-sheet>
        </v-col>
      </v-row>
    </v-container>

    <Footer :footer="footer" />
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

  type: string = 'month'
  events: any = []
  typeToLabel: any = {
    month: 'Month / 月',
    week: 'Week',
    day: 'Day'
    // "4day": "4 Days"
  }

  focus: string = '2000-01-01'
  start: any = null
  end: any = null
  selectedEvent: any = {}
  selectedElement: any = null
  selectedOpen: boolean = false
  yearAndMonth: string = ''

  data: any = {}

  private get calendarInstance (): Vue & { prev: () => void, next: () => void,
    getFormatter: (format: any) => any } {
    return this.$refs.calendar as Vue & { prev: () => void, next: () => void, getFormatter: (format: any) => any }
  }

  get fullTitle () {
    const { start, end } = this
    if (!start || !end) {
      return ''
    }

    const startMonth = this.monthFormatter(start)
    const endMonth = this.monthFormatter(end)
    const suffixMonth = startMonth === endMonth ? '' : endMonth

    const startYear = start.year
    const endYear = end.year
    const suffixYear = startYear === endYear ? '' : endYear

    const startDay = start.day + this.nth(start.day)
    const endDay = end.day + this.nth(end.day)

    switch (this.type) {
      case 'month':
        return `${startMonth} ${startYear}年`
      case 'week':
      case '4day':
        return `${startMonth} ${startDay} ${startYear} - ${suffixMonth} ${endDay} ${suffixYear}`
      case 'day':
        return `${startMonth} ${startDay} ${startYear}`
    }
    return ''
  }

  get monthFormatter () {
    return this.calendarInstance.getFormatter({
      timeZone: 'UTC',
      month: 'long'
    })
  }

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

  nth (d: number) {
    return d > 3 && d < 21
      ? 'th'
      : ['th', 'st', 'nd', 'rd', 'th', 'th', 'th', 'th', 'th', 'th'][d % 10]
  }

  /*
  viewDay({ date: any }) {
    this.focus = date;
    this.type = "day";
  }

  prev() {
    this.$refs.calendar.prev();
    this.updatePath();
  }

  next() {
    this.$refs.calendar.next();
    this.updatePath();
  }

  showEvent({ nativeEvent: any, event: any }) {
    const open = () => {
      this.selectedEvent = event;
      this.selectedElement = nativeEvent.target;
      setTimeout(() => (this.selectedOpen = true), 10);
    };

    if (this.selectedOpen) {
      this.selectedOpen = false;
      setTimeout(open, 10);
    } else {
      open();
    }

    nativeEvent.stopPropagation();
  }

  updateRange({ start, end }) {
    // You could load events from an outside source (like database) now that we have the start and end dates on the calendar
    this.start = start;
    this.end = end;
  }

  handleDateClick(arg: any) {
    window.open(arg.event.id, "_blank");
  }

  updatePath() {
    let focus_arr = this.focus.split("-");
    let yearAndMonth = focus_arr[0] + "-" + focus_arr[1];
    this.$router.push({
      name: "item",
      query: {
        param: JSON.stringify({
          q: this.q,
          collections: this.collections
        }),
        date: yearAndMonth,
        u: this.u
      }
    });
  }
  */

  created () {
    const param: any = Object.assign({}, this.$route.query)

    this.u = param.u ? param.u : 'https://webpark5032.sakura.ne.jp/tmp/calendar.json'

    this.focus = param.date ? param.date + '-01' : '2020-01-01'
    const focusArr = this.focus.split('-')
    this.yearAndMonth = focusArr[0] + '-' + focusArr[1]

    if (param.param) {
      const query: any = JSON.parse(param.param)

      this.q = query.q ? query.q : this.q
      this.collections = query.collections
        ? query.collections
        : this.collections
    }

    console.log(toWareki(2020))

    this.main(this.u)
  }

  mounted () {
    // this.calendarInstance.checkChange()
  }
}
</script>

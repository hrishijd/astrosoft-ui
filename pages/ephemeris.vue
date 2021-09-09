<template>
  <section class="center">
    <AstroInput
      @change="resetData"
      @submit="calculate"
    />
    <b-loading :active.sync="isLoading" />
    <div v-if="ephData.length > 0" class="p-4 mt-10 mb-10 content max-w-xl">
      <h1>
        Planetary Ephemeris on {{ formattedDateTime }} at {{ placeName }}
      </h1>
      <svg id="chart" width="1200" height="800" />
      <table class="table is-bordered is-hoverable card">
        <thead>
          <tr class="bg-green-400 text-gray-900">
            <th>
              Planet
            </th>
            <th>
              Position
            </th>
          </tr>
        </thead>
        {{ buildChart }}
        <tbody>
          <tr v-for="(eph, index) in ephData" v-bind:key="index">
            <td :class="planetStyle(eph.planet)">
              {{ eph.planet }}
              <span v-show="eph.isRetro"> ( R ) </span>
            </td>
            <td class="planet-pos font-semibold text-blue-700">
              {{ eph.position }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </section>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'
import AstroInputData from '../astro/AstroInputData'
import { formatDateTime, formatDegMinSec } from '../mixins/FormatUtils'
import { Ephemeris } from '../astro/Ephemeris'
import { getCurrentPageUrl } from '../mixins/AppUtils'
import AstroChart from '../astro/AstroChart'
import AstroInput from '@/components/AstroInput.vue'

@Component({
  components: {
    AstroInput
  }
})

export default class EphemerisVue extends Vue {
  astroInputData = new AstroInputData();
  ephData: Array<Ephemeris> = []
  isLoading = false

  head () {
    return {
      titleTemplate: 'Planetary Ephemeris ' + this.titleSuffix + ' | %s',
      meta: [
        { name: 'og:url', content: getCurrentPageUrl('/ephemeris') },
        { name: 'twitter:url', content: getCurrentPageUrl('/ephemeris') }
      ]
    }
  }

  resetData () {
    this.ephData = []
  }

  buildChart () {
      console.log('hello')
    const options = {
        title: ['Rasi', '14/04/2014 07:00AM', 'Erode, Tamil Nadu, India'],
         showHouseNumbers: true,
            width: 600,
            height: 400
    }
    const _AstroChart: any = AstroChart
    const astroChart = new _AstroChart('#chart')
    astroChart.draw({
        1: ['Su', 'Ke'],
        3: ['Ju'],
        6: ['Ma', 'Asc'],
        7: ['Mo', 'Sa', 'Ra'],
        11: ['Ve'],
        12: ['Me']
    }, options)
  }

  get titleSuffix () {
    return this.astroInputData.isPlaceSet ? 'at  ' + this.astroInputData.place.placeFormatted : ''
  }

  get formattedDateTime () {
    return formatDateTime(this.astroInputData.dateTimeValue)
  }

  get placeName () {
    return this.astroInputData.place.placeName
  }

  calculate (value: AstroInputData) {
    this.astroInputData = value
    this.isLoading = true
    this.fetchData().then((data) => {
      this.ephData = data
      this.isLoading = false
    })
  }

  async fetchData () {
    const dateTime = this.astroInputData.dateTimeValue
    const body = {
      name: 'Astrosoft UI',
      place: {
        name: this.astroInputData.place.placeName,
        longitude: this.astroInputData.place.lng,
        latitude: this.astroInputData.place.lat,
        timeZoneId: this.astroInputData.timezone.timeZoneId
      },
      year: dateTime.getFullYear(),
      month: dateTime.getMonth() + 1,
      date: dateTime.getDate(),
      hour: dateTime.getHours(),
      minutes: dateTime.getMinutes(),
      seconds: dateTime.getSeconds(),
      options: {
        Ayanamsa: 'LAHARI'
      }
    }
    const resp = await this.$axios.$post('https://api.innovativeastrosolutions.com/v0/horoscope', body)
    return this.parseData(resp)
  }

  parseData (resp: any): Array<Ephemeris> {
    const result: Array<Ephemeris> = []
    let value: any
    for (value of Object.values(resp.planetaryInfo)) {
      result.push({ planet: value.planet, position: formatDegMinSec(value.position), isRetro: value.isRetro })
    }
    return result
  }

  planetStyle (planet: string) {
    const fontStyle = 'font-semibold '
    switch (planet) {
      case 'Sun' : return fontStyle + 'text-orange-800'
      case 'Moon' : return fontStyle + 'text-green-800'
      case 'Ascendant' : return fontStyle + 'text-red-500'
      default : return 'text-gray-800'
    }
  }
}

</script>
<style scoped>

.planet-pos {
  font-family: 'Share Tech Mono', Courier, monospace
}

h1 {
  font-family: 'KoHo', sans-serif;
}
/* <![CDATA[rect.house {
    stroke: #FFA500;
    fill:   #FFFFFF;
}

rect.chart {
    fill: #FFFFFF;
}

text.house {
    font-family : "Courier New";
    font-weight: bold;
    font-size: 16px;
    fill       : #0000CC;
}

text.title {
    font-family: Arial, Helvetica, sans-serif;
    font-weight: 700;
    font-size  : 16px;
    fill       : #C80082;
    text-anchor: middle;
}

text.retrograde {
    fill:#00468C;
}

text.houseNumber {
    font-family : Consolas;
    font-size: 14px;
    fill       : #B6B4AD;
    font-style: italic;
}

#Asc {
    fill:red;
}

#Mo {
    fill:#FF00FF
}
        ]]>
 */</style>

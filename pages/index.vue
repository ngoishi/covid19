<template>
  <div class="MainPage">
    <page-header
      :icon="headerItem.icon"
      :title="headerItem.title"
      :date="headerItem.date"
    />
    <static-info
      class="mb-4"
      title="このホームページは個人が作成したものです。大阪府が作成したものではありません。"/>
    <whats-new class="mb-4" :items="newsItems" />
    <static-info
      class="mb-4"
      :url="'http://www.pref.osaka.lg.jp/iryo/osakakansensho/corona-denwa.html'"
      :text="'自分や家族の症状に不安や心配があればまずは電話相談をどうぞ'"
    />
    <v-row class="DataBlock">
      <v-col cols="12" md="6" class="DataCard">
        <svg-card
          title="検査陽性者の状況"
          :title-id="'details-of-confirmed-cases'"
          :date="headerItem.date"
        >
          <confirmed-cases-table v-bind="confirmedCases" />
        </svg-card>
      </v-col>
      <v-col cols="12" md="6" class="DataCard">
        <data-table
          :title="'陽性患者の属性'"
          :title-id="'attributes-of-confirmed-cases'"
          :chart-data="patientsTable"
          :chart-option="{}"
          :date="Data.patients.date"
          :info="sumInfoOfPatients"
          :url="
            'https://catalog.data.metro.tokyo.lg.jp/dataset/t000010d0000000068'
          "
        />
      </v-col>    
      <v-col cols="12" md="6" class="DataCard">
        <time-stacked-bar-chart
          title="検査実施数"
          :title-id="'number-of-tested'"
          :chart-id="'time-stacked-bar-chart-inspections'"
          :chart-data="inspectionsGraph"
          :date="Data.inspections_summary.date"
          :labels="inspectionsLabels"
          :unit="'件'"
        />
      </v-col>      
      <v-col cols="12" md="6" class="DataCard">
        <time-bar-chart
          title="陽性患者数"
          :title-id="'number-of-confirmed-cases'"
          :chart-id="'time-bar-chart-patients'"
          :chart-data="patientsGraph"
          :date="Data.patients.date"
          :unit="'人'"
          :url="
            'https://catalog.data.metro.tokyo.lg.jp/dataset/t000010d0000000068'
          "
        />
      </v-col>
   <v-col cols="12" md="6" class="DataCard">
        <time-bar-chart2
          title="退院者数"
          :title-id="'number-of-discharges-cases'"
          :chart-id="'time-bar-chart-discharges'"
          :chart-data="dischargesGraph"
          :date="Data.discharges_summary.date"
          :unit="'人'"
          :url="
            'https://catalog.data.metro.tokyo.lg.jp/dataset/t000010d0000000068'
          "
        />
      </v-col>
    </v-row>
  </div>
</template>

<script>
import PageHeader from '@/components/PageHeader.vue'
import TimeBarChart from '@/components/TimeBarChart.vue'
import TimeBarChart2 from '@/components/TimeBarChart2.vue'
import MetroBarChart from '@/components/MetroBarChart.vue'
import TimeStackedBarChart from '@/components/TimeStackedBarChart.vue'
import WhatsNew from '@/components/WhatsNew.vue'
import StaticInfo from '@/components/StaticInfo.vue'
import Data from '@/data/data.json'
import MetroData from '@/data/metro.json'
import DataTable from '@/components/DataTable.vue'
import formatGraph from '@/utils/formatGraph'
import formatTable from '@/utils/formatTable'
import formatConfirmedCases from '@/utils/formatConfirmedCases'
import News from '@/data/news.json'
import SvgCard from '@/components/SvgCard.vue'
import ConfirmedCasesTable from '@/components/ConfirmedCasesTable.vue'

export default {
  components: {
    PageHeader,
    TimeBarChart,
    TimeBarChart2,
    MetroBarChart,
    TimeStackedBarChart,
    WhatsNew,
    StaticInfo,
    DataTable,
    SvgCard,
    ConfirmedCasesTable
  },
  data() {
    // 感染者数グラフ
    const patientsGraph = formatGraph(Data.patients_summary.data)
    // 感染者数
    const patientsTable = formatTable(Data.patients.data)
    // 退院者グラフ
    const dischargesGraph = formatGraph(Data.discharges_summary.data)

    // 相談件数
    const contactsGraph = formatGraph(Data.contacts.data)
    // 帰国者・接触者電話相談センター相談件数
    const querentsGraph = formatGraph(Data.querents.data)
    // 都営地下鉄の利用者数の推移
    const metroGraph = MetroData
    // 検査実施日別状況
    const inspectionsGraph = [
     　Data.inspections_summary.data['府内'],
      Data.inspections_summary.data['その他']
    ]
     const inspectionsItems = [
     '府内発生'
    ]
 
    const inspectionsLabels = Data.inspections_summary.labels
    // 死亡者数
    // #MEMO: 今後使う可能性あるので一時コメントアウト
    // const fatalitiesTable = formatTable(
    //   Data.patients.data.filter(patient => patient['備考'] === '死亡')
    // )
    // 検査陽性者の状況
    const confirmedCases = formatConfirmedCases(Data.main_summary)

    const sumInfoOfPatients = {
      lText: patientsGraph[
        patientsGraph.length - 1
      ].cumulative.toLocaleString(),
      sText: patientsGraph[patientsGraph.length - 1].label + 'の累計',
      unit: '人'
    }

    const data = {
      Data,
      patientsTable,
      patientsGraph,
      dischargesGraph,
      contactsGraph,
      querentsGraph,
      metroGraph,
      inspectionsGraph,
      inspectionsLabels,
      confirmedCases,
      sumInfoOfPatients,
      headerItem: {
        icon: 'mdi-chart-timeline-variant',
        title: '大阪府内の最新感染動向',
        date: Data.lastUpdate
      },
      newsItems: News.newsItems,
      metroGraphOption: {
        responsive: true,
        legend: {
          display: false
        },
        scales: {
          xAxes: [
            {
              position: 'bottom',
              stacked: false,
              gridLines: {
                display: true
              },
              ticks: {
                fontSize: 10,
                maxTicksLimit: 20,
                fontColor: '#808080'
              }
            }
          ],
          yAxes: [
            {
              stacked: false,
              gridLines: {
                display: true
              },
              ticks: {
                fontSize: 12,
                maxTicksLimit: 10,
                fontColor: '#808080',
                callback(value) {
                  return value.toFixed(2) + '%'
                }
              }
            }
          ]
        },
        tooltips: {
          displayColors: false,
          callbacks: {
            title(tooltipItems, _) {
              const label = tooltipItems[0].label
              return `期間: ${label}`
            },
            label(tooltipItem, data) {
              const currentData = data.datasets[tooltipItem.datasetIndex]
              const percentage = `${currentData.data[tooltipItem.index]}%`

              return `${metroGraph.base_period}の利用者数との相対値: ${percentage}`
            }
          }
        }
      }
    }
    return data
  },
  head() {
    return {
      title: '大阪府内の最新感染動向'
    }
  }
}
</script>

<style lang="scss" scoped>
.MainPage {
  .DataBlock {
    margin: 20px -8px;
    .DataCard {
      @include largerThan($medium) {
        padding: 10px;
      }
      @include lessThan($small) {
        padding: 4px 8px;
      }
    }
  }
}
</style>

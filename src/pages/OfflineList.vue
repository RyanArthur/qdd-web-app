<template>
  <v-table
    is-vertical-resize
    style="width:100%"
    is-horizontal-resize
    :min-height='200'
    :columns="columns"
    :table-data="tableData"
    :show-vertical-border="false"
    :row-click='showDetails'
  ></v-table>
</template>

<script>
  import {Indicator, Toast} from 'mint-ui'
  export default {
    name: 'offline-list',
    data () {
      return {
        dealerId: null,
        storeId: null,
        tableData: [],
        columns: [
          {
            field: 'ccuSn',
            title: '序列号',
            width: 100,
            titleAlign: 'center',
            columnAlign: 'center',
            isResize: true,
            formatter: function (rowData, rowIndex, pagingIndex, field) {
              return rowData.ccuSn
            }
          },
          {
            field: 'storeName',
            title: '驿站',
            width: 100,
            titleAlign: 'center',
            columnAlign: 'center',
            isResize: true
          },
          // {
          //   field: 'realname',
          //   title: '姓名',
          //   width: 50,
          //   titleAlign: 'center',
          //   columnAlign: 'center',
          //   isResize: true,
          //   formatter: function (rowData, rowIndex, pagingIndex, field) {
          //     return rowData.owner.realname
          //   }
          // },
          {
            field: 'reportTime',
            title: '最后上报时间',
            width: 100,
            titleAlign: 'center',
            columnAlign: 'center',
            isResize: true
          }
          // {
          //   field: 'surplusTime',
          //   title: '有效期',
          //   width: 60,
          //   titleAlign: 'center',
          //   columnAlign: 'center',
          //   isResize: true,
          //   formatter: function (rowData, rowIndex, pagingIndex, field) {
          //     let tempIndex = rowData.surplusTime.indexOf('小时')
          //     let length = rowData.surplusTime.length
          //     if (tempIndex > 0) {
          //       length = tempIndex + 2
          //     }
          //     return rowData.surplusTime.substring(1, length)
          //   }
          // }
        ]
      }
    },
    methods: {
      loadStoreInfo () {
        Indicator.open('加载中...')
        this.axios.get('/api-user/v3.1/ebikestores/' + this.storeId).then((res) => {
          Indicator.close()
          let data = res.data
          if (data) {
            if (data.category === 0 || data.category === 1) {
              this.dealerId = data.id
              this.storeId = null
            }
            this.loadStatistics()
          }
        })
          .catch(error => {
            Indicator.close()
            if (error.data.error) {
              Toast(error.data.error.msg)
            }
          })
      },
      loadStatistics () {
        // http://cjl3.rokyinfo.net:7200/api-ebike/v3.1/ebikes/list?categories=1,2&exceptionFlag=1&bmsExceptionReason=0&sort=ebike.report_time,asc&limit=100&page=1
        Indicator.open('加载中...')
        this.axios.get('/api-ebike/v3.1/ebikes/list?exceptionFlag=1&bmsExceptionReason=0&sort=ebike.report_time,asc&limit=1000&page=1',
          {
            params: {
              dealerId: this.dealerId,
              storeId: this.storeId
            }
          }
        ).then((res) => {
          Indicator.close()
          this.tableData = res.data.list
        })
          .catch(error => {
            Indicator.close()
            if (error.data.error) {
              Toast(error.data.error.msg)
            }
          })
      },
      showDetails (rowIndex, rowData, column) {
        this.$router.push({
          name: 'EquipmentDetail',
          params: {ccusn: rowData.ccuSn},
          query: {reportTime: rowData.reportTime,
            voltageString: rowData.voltageString,
            storeName: rowData.storeName,
            usedDays: rowData.usedDays,
            surplusTime: rowData.surplusTime,
            owner: rowData.owner,
            bmsSoc: rowData.bmsSoc,
            odo: rowData.odo,
            dayOdo: rowData.dayOdo,
            guige: rowData.productEntity.name,
            gear: rowData.gear,
            status: rowData.status,
            lon: rowData.lon,
            lat: rowData.lat
          }
        })
      }
    },
    mounted () {
      document.title = '离线查询'
      if (this.$route.query) {
        this.$store.commit('setToken', this.$route.query.token)
        this.$store.commit('setFirm', this.$route.query.firm)
        this.axios.defaults.headers.common['firm'] = this.$route.query.firm
        if (this.$route.query.token) {
          this.axios.defaults.headers.common['Authorization'] = this.$route.query.token
        }
        this.storeId = this.$route.query.storeId
      }

      this.loadStoreInfo()
    }
  }
</script>

<style scoped>

</style>

<template>
  <div class="layout">
     <Layout>
       <oracle-menu :value="'2'" :tags="this.Tags"></oracle-menu>
            <Content :style="{margin: '10px 0 0', background: '#fff', minHeight: '500px'}">
               <Row>
          <Card>
            <DatePicker @on-change="handleDaterangechange" type="datetimerange" format="yyyy-MM-dd HH:mm:ss" placeholder="请选择时间范围" style="width: 250px"></DatePicker>
            <br>
            <chart-line1 style="height: 300px;" :title=chart1_title :data1="checktimeData" :data2="usedpctData"></chart-line1>
            <br>
            <chart-line1 style="height: 300px;" :title=chart2_title :data1="checktimeData" :data2="usedsizeData"></chart-line1>
          </Card>
    </Row>
         <row>
     <card>
       <p slot="title">会话使用UNDO表空间信息</p>
          <Table size="small"
               :columns="columns_sessionused"
               :data="sessionusedData">
        </Table>
     </Card>
         </Row>
            </Content>
        </Layout>
  </div>
</template>

<script>
import { OracleMenu } from '_c/top-menu'
import InforCard from '_c/info-card'
import { getOracleUndoTableSpaceHis, getOracleUndoTableSpaceSessionUsed } from '@/api/oracle'
import { ChartLine1 } from '_c/charts'

export default {
  name: 'oracle_tablespace',
  components: {
    OracleMenu,
    InforCard,
    ChartLine1
  },
  data () {
    return {
      Tags: '',
      end: 0,
      asynEndVal: 487,
      integratedEndVal: 3,
      data: [],
      checktimeData: [],
      usedsizeData: [],
      usedpctData: [],
      sessionusedData: [],
      chart1_title: ['UNDO表空间使用率', 'used percent'],
      chart2_title: ['UNDO表空间使用情况(MB)', 'used mb'],
      daterangeValue: null,
      startTime: '',
      endTime: '',
      columns_sessionused: [
        {
          title: 'sid',
          key: 'SID',
          width: 80
        },
        {
          title: '回滚段',
          key: 'RBS',
          width: 200
        },
        {
          title: '用户名',
          key: 'ORACLE_USER',
          width: 100
        },
        {
          title: '客户端用户',
          key: 'CLIENT_USER',
          width: 120
        },
        {
          title: 'program',
          key: 'PROGRAM',
          width: 150
        },
        {
          title: '进程号',
          key: 'UNIX_PID',
          width: 80
        },
        {
          title: '使用空间(MB)',
          key: 'UNDO_MB',
          width: 120,
          sortType: 'desc'
        },
        {
          title: '登录时间',
          key: 'LOGIN_TIME',
          width: 180
        },
        {
          title: '事务开始时间',
          key: 'TRANSACTION_STARTTIME',
          width: 180
        }
      ],
      columns_dayused: [
        {
          title: '日期',
          key: 'M_DATE',
          width: 100,
          sortType: 'asc'
        },
        {
          title: '表空间名称',
          key: 'NAME',
          width: 150
        },
        {
          title: '使用大小(MB)',
          key: 'USED_MB',
          width: 150
        }
      ]
    }
  },
  created () {
    this.get_oracle_undotablespace(`tags=${this.$route.params.tags}&undotablespace_name=${this.$route.params.tablespace_name} `)
    this.get_oracle_undotablespace_sessionused(`tags=${this.$route.params.tags}&undotablespace_name=${this.$route.params.tablespace_name} `)
    this.Tags = this.$route.params.tags
    this.timer = setInterval(() => {
      this.get_oracle_undotablespace(`tags=${this.$route.params.tags}&undotablespace_name=${this.$route.params.tablespace_name} `)
      this.get_oracle_undotablespace_sessionused(`tags=${this.$route.params.tags}&undotablespace_name=${this.$route.params.tablespace_name} `)
    }, 1000 * 20)
  },
  methods: {
    init () {
    },
    get_oracle_undotablespace (parameter) {
      getOracleUndoTableSpaceHis(parameter).then(res => {
        const tbsData = res.data.results
        this.checktimeData = tbsData.map(tbs => tbs.check_time)
        this.usedsizeData = tbsData.map(tbs => tbs.used_size)
        this.usedpctData = tbsData.map(tbs => tbs.percent_used)
        console.log(tbsData)
      }).catch(err => {
        this.$Message.error(`获取表空间信息错误 ${err}`)
      })
    },
    get_oracle_undotablespace_sessionused (parameter) {
      getOracleUndoTableSpaceSessionUsed(parameter).then(res => {
        this.sessionusedData = res.data
        console.log(this.sessionusedData)
      }).catch(err => {
        this.$Message.error(`获取UNDO表空间各会话使用空间信息错误 ${err}`)
      })
    },
    handleDaterangechange (daterange) {
      this.daterangeValue = daterange
      this.startTime = daterange[0]
      this.endTime = daterange[1]
      this.get_oracle_undotablespace(`tags=${this.$route.params.tags}&undotablespace_name=${this.$route.params.tablespace_name}&start_time=${this.startTime}&end_time=${this.endTime}`)
    }
  },
  mounted () {
    this.init()
  },
  destroyed () {
    clearInterval(this.timer)
  }
}
</script>

<style lang="less">
@baseColor: ~"#dc9387";
.countto-page-row{
  height: 200px;
}
.count-to-con{
  display: block;
  width: 100%;
  text-align: center;
}
.count-text{
  font-size: 50px;
  color: @baseColor;
}
.slot-text{
  font-size: 22px;
}
.unit-class{
  font-size: 30px;
  color: @baseColor;
}
.layout-footer-center{
  text-align: center;
}
.count-style{
  font-size: 50px;
}
</style>

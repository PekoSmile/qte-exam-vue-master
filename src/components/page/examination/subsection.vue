<template>
  <div>
    <div>
      <el-row :gutter="20">
        <el-col :span="6">

        </el-col>
        <el-col :span="4" style="margin-left: 10%">
          <el-select v-model="value" placeholder="请选择班级" clearable>
            <el-option
                v-for="item in options"
                :key="item.classId"
                :label="item.className"
                :value="item.classId">
            </el-option>
          </el-select>
        </el-col>

        <el-col :span="4" style="margin-left: 10%">
          <el-select v-model="value1" placeholder="请选择试卷" clearable>
            <el-option
                v-for="item in options1"
                :key="item.paperId"
                :label="item.paperName"
                :value="item.paperId">
            </el-option>
          </el-select>
        </el-col>
        <el-col :span="6" >
          <el-button type="primary" @click="queryLine">查询</el-button>
        </el-col>
      </el-row>
    </div>
    <div id="main" style="width: 80%;height: 600px;">

    </div>

  </div>

</template>

<script>
import echarts from 'echarts'

export default {
  name: "subsection",
  data() {
    return {
      value:'',
      value1:'',
      charts: '',
      opinionData: [],
      options:[],
      options1:[]
    }

  },
  //调用

  methods: {
    queryLine(){
      let param = {
        classId:this.value,
        paperId:this.value1
      };
      this.opinionData=[];
      this.axios
          .post('/qte/paper/queryAchievement.htm',param)
          .then(
              function (response) {
                response.data.result.filter((item, i) => {
                  this.opinionData.push(item)
                })
                // this.charts.setOption(true)
                this.drawLine('main')
              }.bind(this)
          )
          .catch(function (error) {
            console.log(error);
          });
    },


    drawLine(id) {
      this.charts = echarts.init(document.getElementById(id))
      this.charts.setOption({
        title: {
          text: '                     '+'试卷成绩分段'
        },

        tooltip: {},
        legend: {
          data: ['成绩分段']
        },
        // grid: {
        //     left: '3%',
        //     right: '4%',
        //     bottom: '3%',
        //     containLabel: true
        // },

        toolbox: {
          feature: {
            saveAsImage: {}
          }
        },
        xAxis: {
          data: ["0-30分", "30-60分", "60-80分", "80-100分"]
        },
        yAxis: {
          // 纵轴标尺固定
          type: 'value',
          scale: true,
          max: 15,
          min: 0,
          splitNumber: 20,
        },
        series: [{
          name: '总共人数',
          type: 'bar',
          stack: '总量',
          barCategoryGap: '70%',
          data: this.opinionData,
          itemStyle: {
            color: '#0772c4'  //设置柱子颜色
          },
          emphasis: {
            itemStyle: {
              color: '#004e8a',   //hover时改变柱子颜色
               shadowColor: 'rgba(102,102,102,0.50)',
              // shadowOffsetX: 0,
              // shadowOffsetY: 2,
              //shadowBlur: 6,
              borderWidth: 1,
              borderColor: '#008A17FF',
              borderType: 'solid'
            }
          },
        },

        ]
      })
    },
  },
  created() {
    this.axios
        .post('/qte/paper/queryAchievement.htm')
        .then(
            function (response) {
              response.data.result.filter((item, i) => {
                this.opinionData.push(item)
              })
              // this.charts.setOption(true)
              this.drawLine('main')
            }.bind(this)
        )
        .catch(function (error) {
          console.log(error);
        });

    this.axios
        .post('/qte/paper/queryClassIdList.htm')
        .then(
            function (response) {
              response.data.result.filter((item, i) => {
                this.options.push(item)
              })
              // this.charts.setOption(true)
            }.bind(this)
        )
        .catch(function (error) {
          console.log(error);
        });

    this.axios
        .post('/qte/paper/queryPaperIdList.htm')
        .then(
            function (response) {
              response.data.result.filter((item, i) => {
                this.options1.push(item)
              })
              // this.charts.setOption(true)
            }.bind(this)
        )
        .catch(function (error) {
          console.log(error);
        });
  },
  mounted() {
    this.$nextTick(function () {
      this.drawLine('main')
    })
  },
}
</script>

<style scoped>

</style>

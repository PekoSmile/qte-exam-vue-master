// 我的试卷页面
<template>
  <div id="myExam">
    <div class="title">我的试卷</div>
    <div class="wrapper">
      <ul class="paper">
        <li class="item" v-for="(item, index) in pagination" :key="index"  @click="toExam(item)">
          <h2>试卷名：{{ item.paperName }}</h2>
          <h2 v-if="item.status===1">考试成绩：{{item.fraction}}分，已考完</h2>
          <p class="name">班级：{{ item.className }} </p>
          <p class="name">科目：{{ item.subjectName }}</p>
          <div class="info">
            <i class="el-icon-message-solid"></i><span>{{ item.paperDate }}</span> <i
              class="el-icon-timer"></i
          ><span v-if="item.examDate != null">限时{{ item.examDate }}分钟</span>
          </div>
        </li>
      </ul>
      <div class="pagination">
        <Page
            :total="total"
            show-total
            show-sizer
            :page-size-opts="[10, 20, 30, 40, 10000]"
            v-show="tableDataShow"
            @on-change="changePage"
            @on-page-size-change="changePageSize"
            style="text-align: center"
        />
      </div>

      <el-dialog :visible.sync="dialogVisible" width="80%" :before-close="handleClose">
        <div class="all">
          <test :allTitle="allTitle" :title="title" @sendForm="sendForm" ref="form"
                v-if="dialogVisible"></test>
        </div>
        <span slot="footer" class="dialog-footer tc">
                    <div class="tc">
                        <el-button type="primary" @click="sendForm()">提交试卷</el-button>
                    </div>
                </span>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import test from '../../common/test';

export default {
  inject: ['reload'],
  data() {
    return {
      key: null, //搜索关键字
      allExam: null, //所有考试信息
      currentNum: 1,
      pageSize: 10,
      total: 0,
      tableDataShow: true,
      pagination: [],
      dialogVisible: false,
      allTitle: [],
      title: {
        examDate: '',
        paperName: '',
        userName: ''
      },
      jsonList: [],
      classId: '',
      paperId: ''
    };
  },
  components: {test},

  created() {
    this.getExamInfo();
  },
  methods: {
    closeDialog() {
      this.dialogVisible = false;
      this.allTitle = []
    },
    changePage(page) {
      //设置页码
      this.currentNum = page;
      this.getExamInfo();
    },
    changePageSize(pageSize) {
      //设置每页展示数量
      this.pageSize = pageSize;
      this.getExamInfo();
    },

    //获取当前所有考试信息
    getExamInfo() {
      let params = {
        currentNum: this.currentNum,
        pageSize: this.pageSize
      };
      this.axios
          .post('/qte/paper/queryPaperPage.htm', params)
          .then(
              function (response) {
                this.pagination = response.data.result.list;
                this.total = response.data.result.count;
              }.bind(this)
          )
          .catch(function (error) {
            console.log(error);
          });
    },
    handleClose(done) {
      this.$confirm('关闭页面即提交试卷，确定关闭吗').then(() => {
        this.sendForm();
        done();
        console.info("点击确定触发");
      }).catch(() => {
        console.log("点击右上角 'X' ，取消按钮或遮罩层时触发");
      });

    },
    sendForm(val) {
      if (val) {
        //提交的数据
        let jsonArr = [];
        let newArr = val.filter((item, i) => {
          console.log("循环==i==", item);
          let obj = {};
          obj.id = item.id;
          obj.val = item.val;
          jsonArr.push(JSON.stringify(obj))
        })
        this.jsonList.push(jsonArr)
        console.log(this.jsonList)
        let params = {
          paperList: this.jsonList,
          paperId: this.paperId,
          classId: this.classId
        };
        this.axios
            .post('/qte/paper/successPaper.htm', params)
            .then(
                function (response) {
                  if (response.data.code == '0000') {
                    this.$message.success('考试结束');
                    this.reload();

                  } else {
                    this.$message.error('提交失败');
                  }
                }.bind(this))
            .catch(function (error) {
              console.log(error);
            });

        this.allTitle = [];
      } else {
        this.$refs.form.checkForm();
        this.allTitle = [];
      }

      this.dialogVisible = false;
    },

    changeArray(one, val, name) {
      one.map((x) => {
        x[name] = val;
        x['val'] = '';
      });
      return one;
    },
    toExam(item) {
      if (item.status === 0) {
        this.classId = item.classId;
        this.paperId = item.paperId;
        let params = {
          paperId: item.paperId
        };
        this.allTitle=[];

        this.axios
            .post('/qte/paper/addPaperByStudent.htm', params)
            .then(
                function (response) {
                  if (response.data.code == '0000') {
                    this.$message.success('开始考试');
                    this.allTitle.push({
                      list: this.changeArray(response.data.result.oneList1, 1, 'type'),
                      fraction: response.data.result.fractionSum,
                      titleType:'单选题'

                    });
                    this.title.examDate = response.data.result.examDate,
                        this.title.paperName = response.data.result.paperName,
                        this.title.userName = response.data.result.userName,
                        this.allTitle.push({
                          list: this.changeArray(response.data.result.oneList2, 2, 'type'),
                          fraction: response.data.result.fractionSum1,
                          titleType:'填空题'
                        });
                    this.allTitle.push({
                      list: this.changeArray(response.data.result.oneList3, 3, 'type'),
                      fraction: response.data.result.fractionSum2,
                      titleType:'判断题'
                    });
                    console.log(this.allTitle);
                    this.dialogVisible = !this.dialogVisible;
                  } else {
                    this.$message.error('跳转失败');
                  }
                }.bind(this)
            )
            .catch(function (error) {
              console.log(error);
            });
      }

    }
  }
};
</script>

<style lang="scss" scoped>
.pagination {
  padding: 20px 0px 30px 0px;

  .el-pagination {
    display: flex;
    justify-content: center;
  }
}

.tc {
  text-align: center;
}

.paper {
  h4 {
    cursor: pointer;
  }
}

.paper .item a {
  color: #000;
}

.wrapper .top .order {
  cursor: pointer;
}

.wrapper .top .order:hover {
  color: #0195ff;
  border-bottom: 2px solid #0195ff;
}

.all {
  height: 600px;
  overflow: auto;
}

.wrapper .top .order:visited {
  color: #0195ff;
  border-bottom: 2px solid #0195ff;
}

.item .info i {
  margin-right: 5px;
  color: #0195ff;
}

.item .info span {
  margin-right: 14px;
}

.paper .item {
  width: 380px;
  border-radius: 4px;
  padding: 20px 30px;
  border: 1px solid #eee;
  box-shadow: 0 0 4px 2px rgba(217, 222, 234, 0.3);
  transition: all 0.6s ease;
}

.paper .item:hover {
  box-shadow: 0 0 4px 2px rgba(140, 193, 248, 0.45);
  transform: scale(1.03);
}

.paper .item .info {
  font-size: 16px;
  color: #88949b;
}

.paper .item .name {
  font-size: 16px;
  color: #88949b;
}

.paper * {
  margin: 20px 0;
}

.wrapper .paper {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
}

.top .el-icon-search {
  position: absolute;
  right: 10px;
  top: 10px;
}

.top .icon {
  position: relative;
}

.wrapper .top {
  border-bottom: 1px solid #eee;
  margin-bottom: 20px;
}

#myExam .search-li {
  margin-left: auto;
}

.top .search-li {
  margin-left: auto;
}

.top li {
  display: flex;
  align-items: center;
}

.top .search {
  margin-left: auto;
  padding: 10px;
  border-radius: 4px;
  border: 1px solid #eee;
  box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075);
  transition: border-color ease-in-out 0.15s, box-shadow ease-in-out 0.15s;
}

.top .search:hover {
  color: #0195ff;
  border-color: #0195ff;
}

.wrapper .top {
  display: flex;
}

.wrapper .top li {
  margin: 20px;
}

#myExam {
  width: 980px;
  margin: 0 auto;
}

#myExam .title {
  margin: 20px;
  font-size: 18px;
}

#myExam .wrapper {
  background-color: #fff;
}
</style>

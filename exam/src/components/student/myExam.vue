// 我的试卷页面
<template>
  <div id="myExam">
    <div class="title">我的试卷</div>
    <div class="wrapper">
      <ul class="top">
        <li class="order">
          <el-badge :value="this.allExam.length" class="item" type="primary">
            <span @click="handleCurrentChange(1)">全部</span>
          </el-badge>
        </li>
        <li class="order">
          <el-badge
            :value="this.notBeginExam.length"
            class="item"
            type="primary"
          >
            <span @click="examFilter(notBeginExam)">未开始</span>
          </el-badge>
        </li>
        <li class="order">
          <el-badge :value="this.beginExam.length" class="item" type="primary">
            <span @click="examFilter(beginExam)">已开始</span>
          </el-badge>
        </li>
        <li class="order">
          <el-badge :value="this.passExam.length" class="item">
            <span @click="examFilter(passExam)">已过期</span>
          </el-badge>
        </li>
        <li class="search-li">
          <div class="icon">
            <input
              type="text"
              placeholder="试卷名称"
              class="search"
              v-model="key"
            /><i class="el-icon-search"></i>
          </div>
        </li>
        <li>
          <el-button type="primary" @click="search()">搜索试卷</el-button>
        </li>
      </ul>
      <ul class="paper" v-loading="loading">
        <li
          class="item"
          v-for="(item, index) in pagination.records"
          :key="index"
        >
          <h4 @click="toExamMsg(item.examCode)">{{ item.source }}</h4>
          <p class="name">{{ item.source }}-{{ item.description }}</p>
          <div class="info">
            <i class="iconfont iconrili"></i><span>{{ item.examDate }}</span>
            <i class="iconfont iconjiluchoucha"></i
            ><span v-if="item.totalTime != null"
              >限时{{ item.totalTime }}分钟</span
            >
            <i class="iconfont iconkaoshixinxi"></i
            ><span>满分{{ item.totalScore }}分</span>
          </div>
        </li>
      </ul>
      <div class="pagination">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="pagination.current"
          :page-sizes="[6, 10, 20, 40]"
          :page-size="pagination.size"
          layout="total, sizes, prev, pager, next, jumper"
          :total="pagination.total"
        >
        </el-pagination>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  // name: 'myExam'
  data() {
    return {
      loading: false,
      key: null, //搜索关键字
      allExam: [{}], //所有考试信息
      notBeginExam: [{}],
      beginExam: [{}],
      passExam: [{}],
      firstPage: [{}],
      yearNow: null,
      monthNow: null,
      dayNow: null,
      timesNow: null,
      times: null,
      activeExam: this.allExam,
      pagination: {
        //分页后的考试信息
        current: 1, //当前页
        total: null, //记录条数
        size: 6, //每页条数
      },
    };
  },
  created() {
    this.getPageExamInfo();
    this.getExamInfo();
    this.getTimesNow();
    this.loading = true;
  },
  // watch: {

  // },
  methods: {
    //获取当前页面所有考试信息
    getPageExamInfo() {
      this.$axios(
        `/api/exams/${this.pagination.current}/${this.pagination.size}`
      )
        .then((res) => {
          this.firstPage = res.data.data;
          this.pagination = res.data.data;
          this.loading = false;
          console.log(this.pagination);
        })
        .catch((error) => {
          console.log(error);
        });
    },
    //获取当前所有考试信息
    getExamInfo() {
      this.$axios(`/api/exams`)
        .then((res) => {
          this.allExam = res.data.data;
          console.log(res.data.data);
          let i = 0;
          while (this.allExam[i] != null) {
            let beginTime = this.dealTime(this.allExam[i].examDate);
            let finishTime = this.dealTime(this.allExam[i].finishDate);
            if (this.compareDates(this.timesNow, beginTime, finishTime) == 0) {
              this.notBeginExam.unshift(this.allExam[i]);
            } else if (
              this.compareDates(this.timesNow, beginTime, finishTime) == 1
            ) {
              this.beginExam.unshift(this.allExam[i]);
            } else {
              this.passExam.unshift(this.allExam[i]);
            }
            i++;
          }
          this.notBeginExam.pop();
          this.beginExam.pop();
          this.passExam.pop();
        })
        .catch((error) => {
          console.log(error);
        });
    },
    //改变当前记录条数
    handleSizeChange(val) {
      this.pagination.size = val;
      this.getPageExamInfo();
    },
    //改变当前页码，重新发送请求
    handleCurrentChange(val) {
      this.pagination.current = val;
      this.getPageExamInfo();
    },
    //搜索试卷
    search() {
      let newPage = this.allExam.filter((item) => {
        return item.source.includes(this.key);
      });
      this.pagination.records = newPage;
    },
    //跳转到试卷详情页
    toExamMsg(examCode) {
      this.$router.push({ path: "/examMsg", query: { examCode: examCode } });
    },
    //跳转
    examFilter(exam) {
      this.pagination.records = exam;
    },
    //处理时间数据
    dealTime(time) {
      // let _time = time.split("-");
      // return _time[2] + "/" + _time[1] + "/" + _time[0];
      return time.replace(/-/g, "/");
    },
    //获取当前时间
    getTimesNow() {
      const date = new Date();
      if (date.getMonth() < 10) {
        this.timesNow =
          date.getFullYear() +
          "/0" +
          (date.getMonth() + 1) +
          "/" +
          date.getDate();
      } else {
        this.timesNow =
          date.getFullYear() +
          "/" +
          (date.getMonth() + 1) +
          "/" +
          date.getDate();
      }
    },
    compareDates(d1, d2, d3) {
      let date1 = new Date(d1).getTime();
      let date2 = new Date(d2).getTime();
      let date3 = new Date(d3).getTime();
      // console.log(d1, d2, d3);
      // console.log(date1, date2, date3);
      if (date1 < date2) {
        return 0;
      } else if (date1 >= date2 && date1 <= date3) {
        return 1;
      } else if (date1 > date3) {
        return 2;
      }
    },
  },
  beforeDestroy() {
    if (this.times) {
      clearInterval(this.getTimesInterval);
    }
  },
};
</script>

<style lang="less" scoped>
.pagination {
  padding: 20px 0px 30px 0px;
  .el-pagination {
    display: flex;
    justify-content: center;
  }
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
  font-size: 14px;
  color: #88949b;
}
.paper .item .name {
  font-size: 14px;
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
}
#myExam .wrapper {
  background-color: #fff;
}
</style>

<template>
  <div class="hello">
    <h2>秒表</h2>
    <div id="contentContainer">
      <div id='section'>{{latestTotalTimeStr}}</div>
      <div id="content">{{totalTimeStr}}</div>
    </div>
    <div id="watchContainer">
      <el-button round @click="resetBtnClicked">{{ watchOn?'计次':'复位' }}</el-button>
      <el-button round @click="startBtnClicked">{{ watchOn?'停止':'启动' }}</el-button>
    </div>

    <div id="listContainer">
      <el-table :data="listInfo" border>
        <el-table-column prop="index" label="计次" width="180">
        </el-table-column>
        <el-table-column prop="length" label="时长" width="180">
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
var msToTimeStr = function(time) {
  var minute = Math.floor(time / (60 * 1000));
  var second = Math.floor((time - 6000 * minute) / 1000);
  //var milSecond = Math.floor((time % 1000) / 10);
  var milSecond = time % 1000;
  return (
    (minute < 10 ? "0" + minute : minute) +
    ":" +
    (second < 10 ? "0" + second : second) +
    "." +
    (milSecond < 10
      ? "00" + milSecond
      : milSecond < 100 ? "0" + milSecond : milSecond)
  );
};

export default {
  name: "Watch",
  data() {
    return {
      watchOn: false,
      initialTime: null,
      totalSaved: 0,
      latestStart: null,
      interval: null,
      initialTime: null,
      lastRecordTime: null,
      latestTotalMss: 0,
      totalMss: 0,
      records: []
    };
  },
  computed: {
    totalTimeStr() {
      var str = msToTimeStr(this.totalMss);
      return str;
    },

    latestTotalTimeStr() {
      return msToTimeStr(this.latestTotalMss);
    },

    listInfo() {
      return this.records.map((recordTime, index) => ({
        index: index + 1,
        length: msToTimeStr(recordTime)
      }));
    }
  },
  watch: {},
  created() {},
  mounted() {},
  methods: {
    resetBtnClicked() {
      if (this.watchOn) {
        this.record();
      } else {
        this.reset();
      }
    },
    startBtnClicked() {
      this.watchOn = !this.watchOn;
      if (this.watchOn) {
        this.start();
      } else {
        this.stop();
      }
    },
    record() {
      var current = new Date().getTime();
      this.lastRecordTime = current;
      this.latestTotalMss = 0;

      this.totalMss = this.totalSaved + current - this.latestStart;

      var len = this.records.length;
      var total = 0;
      for (var index = 0; index < len; index++) {
        total += this.records[index];
      }

      var recordTime = this.totalMss - total;
      this.records.unshift(recordTime);
    },
    reset() {
      this.initialTime = null;
      this.lastRecordTime = null;
      this.latestTotalMss = 0;
      this.totalMss = 0;

      this.records = [];
    },
    start() {
      var tempTime = new Date().getTime();

      if (this.initialTime != null) this.initialTime = tempTime;

      this.latestStart = tempTime;
      this.totalSaved = this.totalMss;
      var totalLatestSaved = this.latestTotalMss;

      this.interval = setInterval(() => {
        var current = new Date().getTime();
        this.totalMss = this.totalSaved + current - this.latestStart;

        if (this.lastRecordTime == null) {
          this.latestTotalMss = this.totalMss;
        } else {
          if (this.lastRecordTime >= this.latestStart) {
            this.latestTotalMss = current - this.lastRecordTime;
          } else {
            this.latestTotalMss = totalLatestSaved + current - this.latestStart;
          }
        }
      }, 10);
    },
    stop() {
      clearInterval(this.interval);
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

#contentContainer {
  /* background-color: blue; */
  width: 360px;
  height: 100px;
  margin-left: auto;
  margin-right: auto;
  margin-top: 20px;
  margin-bottom: 20px;
  border: 1px solid red;
  padding: 10px;
}

#listContainer {
  /* background-color: blue; */
  width: 360px;
  height: 500px;
  margin-left: auto;
  margin-right: auto;
  margin-top: 20px;
}

#content {
  font-size: 60px;
  text-align: right;
}

#section {
  font-size: 30px;
  text-align: right;
}
</style>

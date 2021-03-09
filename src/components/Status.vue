<template>
  <div>
    <el-row>
      <h3>网络流量</h3>
    </el-row>
    <el-row :gutter="20">
      <el-col :span="12">
        <el-card shadow="hover">
          <div slot="header">
            <span>IPv4 Traffic</span>
          </div>
          <div v-if="chartdata_ipv4 !== null">
            <my-line :chart-data="chartdata_ipv4"></my-line>
          </div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card shadow="hover">
          <div slot="header">
            <span>IPv6 Traffic</span>
          </div>
          <div v-if="chartdata_ipv6 !== null">
            <my-line :chart-data="chartdata_ipv6"></my-line>
          </div>
        </el-card>
      </el-col>
    </el-row>
    <el-row>
      <h3>磁盘状态</h3>
    </el-row>
    <el-row>
      <el-col :span="20" :offset="2">
        <el-progress :text-inside="true" :stroke-width="26" :format="formatDiskUsage" :percentage="75"></el-progress>
      </el-col>
    </el-row>
    <el-row>
      <h3>CPU/内存</h3>
    </el-row>
    <el-row :gutter="20" style="margin-bottom:40px">
      <el-col :span="12">
        <div>
          <el-progress type="circle" :percentage="cpu_usage"></el-progress>
          CPU使用率
        </div>
      </el-col>
      <el-col :span="12">
        <div>
          <el-progress type="circle" :percentage="25"></el-progress>
          内存使用率
        </div>
      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-col :span="12">
        <el-card shadow="hover">
          <div slot="header">
            <span>CPU Usage</span>
          </div>
          <div v-if="chartdata_cpu !== null">
            <my-line :chart-data="chartdata_cpu"></my-line>
          </div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card shadow="hover">
          <div slot="header">
            <span>Memory Usage</span>
          </div>
          <div v-if="chartdata_memory !== null">
            <my-line :chart-data="chartdata_memory"></my-line>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import MyLine from '@/components/Charts/MyLine'
export default {
  name: 'Status',
  data () {
    return {
      chartdata_ipv4: null,
      chartdata_ipv6: null,
      chartdata_cpu: null,
      chartdata_memory: null,
      cpu_usage: 0
    }
  },
  methods: {
    formatDiskUsage: function (percentage) {
      return `30T/40T ${percentage}%`
    },
    generateTestY: function (multi) {
      let data = []
      var h = 0
      var m = 0
      var prev = Math.random() * multi
      for (h = 0; h < 24; h++) {
        for (m = 0; m < 60; m += 10) {
          prev += (Math.random() - 0.5) * 2 * multi * 0.05
          if (prev < 0) {
            prev = 0
          }
          if (prev > multi) {
            prev = multi
          }
          data.push(prev)
        }
      }
      return data
    },
    generateTestX: function () {
      let data = []
      var h = 0
      var m = 0
      for (h = 0; h < 24; h++) {
        for (m = 0; m < 60; m += 10) {
          let hStr = h.toString().padStart(2, '0')
          let mStr = m.toString().padStart(2, '0')
          data.push(hStr + ':' + mStr)
        }
      }
      return data
    },
    calculateX: function (data) {
      let result = []
      data.forEach((val, idx, arr) => {
        let date = new Date(val * 1000) // sec to millisec
        let hStr = date.getHours().toString().padStart(2, '0')
        let mStr = date.getMinutes().toString().padStart(2, '0')
        result.push(hStr + ':' + mStr)
      })
      return result
    },
    generateCpuData: function () {
      let that = this
      this.$axios.get('http://mirrors.pku.edu.cn/monitor_device/api/v1/data?chart=system.cpu&after=-86400&before=0&points=400&group=average&gtime=0&format=json&options=seconds&options=jsonwrap').then((response) => {
        response.data.result.data[0].forEach((val, idx, arr) => {
          if (idx !== 0) {
            that.cpu_usage += val
          }
        })
        that.cpu_usage = Math.round(that.cpu_usage + 0.5)
        let raw = response.data.result.data.reverse()
        let needIdx = []
        let data = []
        response.data.result.labels.forEach((val, idx, arr) => {
          if (val === 'softirq' ||
            val === 'irq' ||
            val === 'user' ||
            val === 'system' ||
            val === 'iowait') {
            needIdx.push(idx)
          }
        })
        needIdx.forEach((val, idx, arr) => {
          data.push({
            label: response.data.result.labels[val],
            data: []
          })
        })
        let rawTimeline = []
        raw.forEach((val, idx, arr) => {
          rawTimeline.push(val[0])
          needIdx.forEach((target, targetIdx, arr2) => {
            data[targetIdx].data.push(val[target])
          })
        })
        that.chartdata_cpu = {
          type: '%',
          axisX: that.calculateX(rawTimeline),
          data: data
        }
      })
    },
    generateData: function () {
      this.chartdata_ipv4 = {
        type: 'Mbps',
        axisX: this.generateTestX(),
        data: [
          {
            label: 'http-ipv4',
            data: this.generateTestY(800)
          },
          {
            label: 'https-ipv4',
            data: this.generateTestY(800)
          },
          {
            label: 'rsync-ipv4',
            data: this.generateTestY(800)
          }
        ]
      }
      this.chartdata_ipv6 = {
        type: 'Mbps',
        axisX: this.generateTestX(),
        data: [
          {
            label: 'http-ipv6',
            data: this.generateTestY(800)
          },
          {
            label: 'https-ipv6',
            data: this.generateTestY(800)
          },
          {
            label: 'rsync-ipv6',
            data: this.generateTestY(800)
          }
        ]
      }
      this.chartdata_memory = {
        type: '%',
        axisX: this.generateTestX(),
        data: [
          {
            label: 'used',
            data: this.generateTestY(100)
          },
          {
            label: 'buffered',
            data: this.generateTestY(100)
          },
          {
            label: 'cached',
            data: this.generateTestY(100)
          },
          {
            label: 'free',
            data: this.generateTestY(100)
          }
        ]
      }
      this.generateCpuData()
    }
  },
  components: {
    'my-line': MyLine
  },
  mounted () {
    this.generateData()
    this.$message('测试数据，非真实数据')
  }
}
</script>

<style scoped>
h1, h2 {
  font-weight: normal;
}
</style>
<style>
.el-card__header {
  padding: 12px 20px;
}
</style>

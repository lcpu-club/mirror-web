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
          <el-progress type="circle" :percentage="95"></el-progress>
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
      chartdata_memory: null
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
      this.chartdata_cpu = {
        type: '%',
        axisX: this.generateTestX(),
        data: [
          {
            label: 'User',
            data: this.generateTestY(100)
          },
          {
            label: 'System',
            data: this.generateTestY(100)
          },
          {
            label: 'Idle',
            data: this.generateTestY(100)
          },
          {
            label: 'I/O wait',
            data: this.generateTestY(100)
          },
          {
            label: 'Soft IRQ',
            data: this.generateTestY(100)
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

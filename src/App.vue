

<template>

  <el-row :gutter="20">
    <el-col :span="16">

      <el-table :data="tableData" stripe style="width: 100%">
        <el-table-column prop="policyNo" label="保单号"   />
        <el-table-column prop="name" label="姓名" />
        <el-table-column prop="idNo" label="身份证号" />
        <el-table-column prop="plateNo" label="车牌" />
        <el-table-column prop="carBand" label="品牌" />
        <el-table-column prop="amount" label="总金额" />
        <el-table-column prop="startDate" label="生效日期" />
        <el-table-column prop="endDate" label="截止日期" />
        <el-table-column prop="policyStatus" label="状态" >
          <template #default="scope">
            {{ scope.row.policyStatus === "CONFIRMED" ? "✅" : "❌"}}
          </template>
        </el-table-column>
      </el-table>
    </el-col>

    <el-col :span="8" style="background-color: aliceblue;">
      <div style="height: 500px;overflow: scroll">
        <el-timeline style="max-width: 100%">
          <el-timeline-item
              v-for="(activity, index) in activities"
              :key="index"
              :icon="activity.icon"
              :type="activity.type"
              :color="activity.color"
              :size="activity.size"
              :hollow="activity.hollow"
              :timestamp="activity.timestamp"
          >
            {{ activity.content }}
          </el-timeline-item>
        </el-timeline>
      </div>
      <div id="container">
        <div id="chat">
          <el-input
              v-model="msg"
              input-style="width: 100%;height:50px"
              :rows="2"
              type="text"
              placeholder="Please input"
              @keydown.enter="sendMsg();"
          />
          <el-button  @click="sendMsg()">发送</el-button>
        </div>
      </div>
    </el-col>
  </el-row>

</template>
<script lang="ts">
import { MoreFilled } from '@element-plus/icons-vue'
import {ref, onMounted} from "vue";
import axios from 'axios'//引入axios

export default {
  setup() {
    const activities = ref([
      {
        content: '⭐欢迎信任智能保险！请问有什么可以帮您的?',
        timestamp: new Date().toLocaleDateString() + " " + new Date().toLocaleTimeString(),
        color: '#0bbd87',
      },
    ]);
    const msg = ref('');
    const tableData = ref([]);
    let count = 2;
    let eventSource;

    const sendMsg = () => {
      if (eventSource) {
        eventSource.close();
      }

      activities.value.push(
          {
            content: `你:${msg.value}`,
            timestamp: new Date().toLocaleDateString() + " " + new Date().toLocaleTimeString(),
            size: 'large',
            type: 'primary',
            icon: MoreFilled,
          },
      );

      activities.value.push(
          {
            content: 'waiting...',
            timestamp: new Date().toLocaleDateString() + " " + new Date().toLocaleTimeString(),
            color: '#0bbd87',
          },
      );

      // sse: 服务端推送 Server-Sent Events
      eventSource = new EventSource(`http://localhost:8080/ai/streamAsString?message=${msg.value}`);
      msg.value='';
      eventSource.onmessage = (event) => {
        if (event.data === '[complete]') {
          count = count + 2;
          eventSource.close();
          getBookings();  // 每次对话完后刷新列表
          return;
        }
        activities.value[count].content += event.data;
      };
      eventSource.onopen = (event) => {
        activities.value[count].content = '';
      };
    };

    const getBookings = () => {
      axios.get('http://localhost:8080/policy/list')
          .then((response) => {
            debugger;
            tableData.value = response.data;
          })
          .catch((error) => {
            console.error(error);
          });
    };
    // Use onMounted to call getBookings when the component is mounted
    onMounted(() => {
      getBookings();
    });

    return {
      activities,
      msg,
      tableData,
      sendMsg,
      getBookings,
    };
  },
};
</script>
<style scoped>
  * {
    margin: 0;
    padding: 0;
  }
  #chat button{
  position: absolute;
  margin-left: -60px;
  margin-top: 19px;
  }
</style>

<template>
  <!-- FIXME: add by axiang [20190702] 功能错误。讨论界面里没有楼主的贴子信息只有置顶的管理员消息，学长为了答辩演示挖的坑，记得填上 -->
  <div class="docker">
    <el-card
      :body-style="{ padding: '0px' }"
      class="box-card"
    >
      <div class="chat-head">
        <div style="float:left">{{ this.$route.query.title }}</div>
      </div>
      <div class="chat-main">
        <div class="chat-left">
          <div style="height:30px;margin-top:90px;font-size:30px">管理员</div>
        </div>
        <div class="chat-right">
          <div style="min-height:140px;margin:20px 20px 20px 20px;float:left;font-size:20px">
            请同学们遵纪守法，不允许发送辱骂语言，否则封号处理！
            <br>请同学们发表积极性语言！
          </div>
        </div>
      </div>
    </el-card>
    <div
      v-if="messageList==''"
      style="width:80%;height:200px;margin:5% 10% 2% 10%;font-size:40px"
    >没有同学回复</div>
    <el-card
      v-for="(o,index) in messageList"
      :key="o"
      :body-style="{ padding: '0px' }"
      class="box-card"
    >
      <div class="chat-head">
        <div style="float:left">{{ index+1+(currentPage-1)*50 }}楼：{{ o.time }}</div>
      </div>
      <div class="chat-main">
        <div class="chat-left">
          <div style="height:30px;margin-top:90px;font-size:25px;">{{ o.author }}</div>
        </div>
        <div class="chat-right">
          <div
            style="min-height:140px;margin:20px 20px 20px 20px;float:left;font-size:20px"
            v-html="o.text"
          >
            <pre>{{ o.text }}</pre>
          </div>
        </div>
      </div>
    </el-card>
    <el-pagination
      style="margin-bottom:2%;"
      layout="prev, pager, next"
      :current-page="currentPage"
      :total="currentTotal*10"
      @current-change="getChat"
    />
    <el-card
      v-if="isLogin"
      :body-style="{ padding: '0px' }"
      class="call"
    >
      <div class="chat-head">
        <div style="float:left">回复</div>
      </div>
      <div style="height:380px;width:98%;padding:20px;">
        <el-input
          v-model="message"
          type="textarea"
          rows="10"
        />
        <el-button
          style="margin-top:20px;float:left"
          @click="sendMessage"
        >submit</el-button>
      </div>
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      text: '',
      data: '',
      title: '',
      websocket: null,
      messageList: '',
      currentPage: 1,
      currentTotal: 0,
      message: '',
      isLogin: false
    }
  },
  mounted() {
    this.isLoginStatu()
    this.getChat(this.currentPage)
  },
  methods: {
    async getChat(val) {
      this.tableData = []
      this.currentPage = val
      const params = new URLSearchParams()
      params.append('discussid', this.$route.query.id)
      params.append('pagenum', val)
      const dataDisuccssReplyById = await this.$http
        .post('/newdiscussreply/GDiscussReplyById', params)
        .catch(() => {
          this.$message({ message: '服务器繁忙，请稍后再试！', type: 'error' })
        })

      if (dataDisuccssReplyById.data[0] !== null) {
        this.currentTotal = dataDisuccssReplyById.data[0]
        this.messageList = dataDisuccssReplyById.data[1]
      }
    },
    isLoginStatu() {
      if (this.$store.getters.getUsername) {
        this.isLogin = true
      } else {
        this.isLogin = false
      }
    },
    async sendMessage() {
      const params = new URLSearchParams()
      params.append('discussid', this.$route.query.id)
      params.append('author', this.$store.getters.getUsername)
      params.append('text', this.message)
      const datainsertDiscussReply = await this.$http
        .post('/newdiscussreply/insertDiscussReply', params)
        .catch(() => {
          this.$message({ message: '服务器繁忙，请稍后再试！', type: 'error' })
        })
      this.logger.i(datainsertDiscussReply.data[0])
      this.$router.go({
        path: '/Chat',
        query: { title: this.$route.query.title, id: this.$route.query.id }
      })
    }
  }
}
</script>
<style scoped>
.docker {
  width: 100%;
  min-height: 700px;
  margin: 0;
  padding: 0;
}

.box-card {
  min-height: 200px;
  width: 80%;
  margin: 2% 10% 2% 10%;
  padding: 0;
}

.call {
  height: 400px;
  width: 80%;
  margin: 2% 10% 2% 10%;
  padding: 0;
}

.chat-head {
  background-color: #f5f5f5;
  height: 20px;
  width: 100%;
  margin: 0;
  border-bottom: 1px solid #eeeeee;
  padding: 10px 15px;
  display: block;
}

.chat-main {
  width: 100%;
  margin: 0;
  padding: 0;
  min-height: 180px;
  display: inline-flex;
}

.chat-left {
  width: 20%;
  margin: 0;
  min-height: 100%;
  background-color: #e6e6fa;
  line-height: 100%;
}

.chat-right {
  width: 80%;
  margin: 0;
  padding: 0;
  min-height: 100%;
  background-color: #ffffff;
}
</style>

<template>
  <div>
    <div class="messenger-header">
      <label class="back-icon" @click="changeConversationStatus('start')"><i class="fa fa-chevron-left"></i></label>
      <label><b>Your conversations</b></label>
    </div>
    <div class="conversation-content" v-if="data !== null">
      <div class="item-row" v-for="item, index  in data" @click="changeConversationStatus('conversation', item)">
        <div class="profile" v-if="item.last_message.title !== null">
          <img :src="config.BACKEND_URL + item.last_message.title.profile.url" v-if="item.last_message.title.profile !== null">
          <i class="fa fa-user-circle-o text-green" v-else></i>
        </div>
        <div class="details" v-if="item.last_message !== null">
          <span class="top" v-if="item.last_message.title !== null">
            <label>{{item.last_message.title.username}}</label>
            <label class="pull-right">{{item.last_message.created_at_human}}</label>
          </span>
          <span class="middle" >
            <label>{{item.last_message.description}}</label>
          </span>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped lang="scss">
@import "~assets/style/colors.scss";
.messenger-header{
  width: 100%;
  float: left;
  height: 60px;
  border-top-right-radius: 10px;
  border-top-left-radius: 10px;
  background-image: linear-gradient(to right, #fffff0, $primary);
}
.messenger-header{
  line-height: 60px;
}
.back-icon{
  width: 40px;
  height: 40px;
  float: left;
  margin-top: 10px;
  text-align: center;
  margin-left: 2%;
  margin-right: 2%;
  line-height: 40px;
}
.back-icon:hover{
  border-radius: 5px;
  background: $primary;
  cursor: pointer;
  color: #fff;
}
.conversation-content{
  width: 100%;
  float: left;
  height: 290px;
  margin-bottom: 5px;
  margin-top: 5px;
  overflow-y: auto;
}
.item-row{
  height: 60px;
  width: 100%;
  float: left;
  border-bottom: solid 1px #eaeaea;
}
.item-row:hover, .item-row .details label:hover{
  cursor: pointer;
}
.item-row .profile{
  width: 20%;
  float: left;
  height: 60px;
  text-align: center;
}
.profile img{
  height: 40px;
  width: 40px;
  border-radius: 50%;
  margin-top: 10px;
}
.profile i{
  line-height: 40px;
  font-size: 40px;
  margin-top: 10px;
}
.item-row .details{
  width: 75%;
  float: left;
  margin-right: 5%;
  height: 60px;
}
.details .top{
  width: 100%;
  float: left;
  height: 20px;
  margin-top: 10px;
  color: #555;
  line-height: 20px;
}
.details .middle{
  width: 100%;
  float: left;
  height: 20px;
  font-size: 12px;
  line-height: 20px;
  font-style: italic;
  font-weight: 500;
  text-overflow: ellipsis;
}
</style>
<script>
import ROUTER from 'src/router'
import AUTH from 'src/services/auth'
import CONFIG from 'src/config.js'
import axios from 'axios'
export default {
  mounted(){
    this.retrieve()
  },
  data(){
    return {
      user: AUTH.user,
      config: CONFIG,
      data: null
    }
  },
  methods: {
    redirect(parameter){
      ROUTER.push(parameter)
    },
    changeConversationStatus(status, item){
      this.$parent.conversationStatus = status
      this.$parent.group = item
      if(status === 'conversation'){
        AUTH.support.messengerGroupId = parseInt(item.last_message.messenger_group_id)
        AUTH.support.message = null
      }
    },
    retrieve(){
      let parameter = null
      if(this.user.type === 'PARTNER' || this.user.type === 'USER'){
        parameter = {
          condition: [{
            column: 'account_id',
            value: this.user.userID,
            clause: '='
          },
          {
            column: 'payload',
            value: 'support',
            clause: '='
          }],
          sort: {
            updated_at: 'desc'
          },
          account_id: this.user.userID
        }
      }else{
        parameter = {
          condition: [{
            column: 'payload',
            value: 'support',
            clause: '='
          }],
          sort: {
            updated_at: 'desc'
          },
          account_id: this.user.userID
        }
      }
      this.APIRequest('messenger_groups/retrieve_my_issue', parameter).then(response => {
        if(response.data.length > 0){
          this.data = response.data
        }else{
          this.data = null
        }
      })
    }
  }
}
</script>

<template>
  <div id="table">
    <div class="add">
      <input type="text" v-model="addDetail.user_name" name="user_name" value="" placeholder="用户名" />
      <input type="text" v-model="addDetail.license_plate" name="license_plate" value="" placeholder="车牌号码" />
      <input type="text" v-model="addDetail.parking_space_id" name="parking_space_id" value="" placeholder="停车位ID" />
      <input type="text" v-model="addDetail.in_black_list" name="in_black_list" value="" placeholder="是否为黑名单用户" />
      <input type="text" v-model="addDetail.parking_space_rent" name="parking_space_rent" value="" placeholder="停车位租赁状况" />
      <input type="text" v-model="addDetail.parkingspace_owner_state" name="parkingspace_owner_state" value="" placeholder="停车位拥有人状态" />
<!--      <input type="date" v-model="addDetail.dates" name="date" value="" placeholder="发布时间" />-->
      <button @click="adddetail">新增</button>
    </div>
    <table cellpadding="0" cellspacing="0">
      <thead>
      <tr>
        <th>ID</th>
        <th>用户名</th>
        <th>车牌号码</th>
        <th>停车位ID</th>
        <th>是否为黑名单用户</th>
        <th>停车位租赁状况</th>
        <th>停车位拥有人状态</th>
        <th>操作</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(item,index) in newsList" :key='index'>
        <td >{{item.id}}</td>
        <td>{{item.user_name}}</td>
        <td>{{item.license_plate}}</td>
        <td >{{item.parking_space_id}}</td>
        <td>{{item.in_black_list == 0 ? "否" : "是"}}</td>
        <td>{{item.parking_space_rent == 0 ? "未租赁" : "已租赁"}}</td>
        <td v-if="item.parkingspace_owner_state == 0">用户不在停车场内</td><td v-else-if="item.parkingspace_owner_state == 1">用户在停车场入口道闸处</td><td v-else-if="item.parkingspace_owner_state == 2">用户已进入停车场</td><td v-else>用户已停车</td>
        <td><span @click="deletelist(item.id,index)" class="delete">删除</span><span class="edit" @click="edit(item)">编辑</span></td>
      </tr>
      </tbody>
    </table>
    <div id="mask" v-if="editlist">
      <div class="mask">
        <div class="title">
          编辑
          <span @click="editlist=false">
					X
				</span>
        </div>
        <div class="content">
          <input type="text" v-model="editDetail.user_name" name="user_name" value="" placeholder="用户名" />
          <input type="text" v-model="editDetail.license_plate" name="license_plate" value="" placeholder="车牌号码" />
          <input type="text" v-model="editDetail.parking_space_id" name="parking_space_id" value="" placeholder="停车位ID" />
          <input type="text" v-model="editDetail.in_black_list" name="in_black_list" value="" placeholder="是否为黑名单用户" />
          <input type="text" v-model="editDetail.parking_space_rent" name="parking_space_rent" value="" placeholder="停车位租赁状况" />
          <input type="text" v-model="editDetail.parkingspace_owner_state" name="parkingspace_owner_state" value="" placeholder="停车位拥有人状态" />
          <button @click="update">更新</button>
          <button @click="editlist=false">取消</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import Vue from 'vue'
  import axios from 'axios'
  import qs from 'qs'
  import VueRouter from "vue-router"
  import global from './Global'

  export default {
    name: "ParkingSpaceOwner",
    data(){
      return{
        addDetail: {},
        editlist: false,
        editDetail: {},
        newsList: [],
        editid:'',
      }
    },
    mounted() {
      let count = 0;
      global.MessageWatchFlag = false;
      global.IoTMessageWatchFlag = false;
      global.ParkingLotUserWatchFlag = false;
      global.ParkingOrderWatchFlag = false;
      global.ParkingSpaceWatchFlag = false;
      global.ParkingSpaceOwnerWatchFlag = true;
      while((count < 100)) {
        setTimeout(() =>{
          if(global.ParkingSpaceOwnerWatchFlag){
          axios
            .get('http://118.31.77.203:8080/Entity/U21a840a21ebf11/PeterPark2/Parkingspaceowner/')
            .then(response => {
              this.newsList = response.data.Parkingspaceowner;
            })
        }}, count * 5000);
        count++;
      }
    },

    methods: {
      adddetail() {
        //这里的思路应该是把this.addDetail传给服务端，然后加载列表this.newsList
        //this.newsList.push(this.addDetail)
        this.newsList.push({
          user_name: this.addDetail.user_name,
          license_plate: this.addDetail.license_plate,
          parking_space_id: this.addDetail.parking_space_id,
          in_black_list: this.addDetail.in_black_list,
          parking_space_rent: this.addDetail.parking_space_rent,
          parkingspace_owner_state: this.addDetail.parkingspace_owner_state
        });
        let json = JSON.stringify({
          "user_name": this.addDetail.user_name,
          "license_plate": this.addDetail.license_plate,
          "parking_space_id": parseInt(this.addDetail.parking_space_id),
          "in_black_list": parseInt(this.addDetail.in_black_list),
          "parking_space_rent": parseInt(this.addDetail.parking_space_rent),
          "parkingspace_owner_state": parseInt(this.addDetail.parkingspace_owner_state),
        });

        axios
          .post('http://118.31.77.203:8080/Entity/U21a840a21ebf11/PeterPark2/Parkingspaceowner/', json, {headers: {'Content-Type': 'application/json'}}
          ).then((res) => {
          console.log(res);
          axios
            .get('http://118.31.77.203:8080/Entity/U21a840a21ebf11/PeterPark2/Parkingspaceowner/')
            .then(response => {
              this.newsList = response.data.Parkingspaceowner
            })
        });

      },
      //删除
      deletelist(id, i) {
        this.newsList.splice(i, 1);
        axios.delete('http://118.31.77.203:8080/Entity/U21a840a21ebf11/PeterPark2/Parkingspaceowner/' + id)
          .then((res) => {
            console.log(res);
            axios
              .get('http://118.31.77.203:8080/Entity/U21a840a21ebf11/PeterPark2/Parkingspaceowner/')
              .then(response => {
                this.newsList = response.data.Parkingspaceowner
              })
          });
        //这边可以传id给服务端进行删除  ID = id
        //axios.get('url',{ID:id}).then((res) =>{
        //			加载列表
        //})
      },
      //编辑
      edit(item) {
        this.editDetail = {
          user_name: item.user_name,
          license_plate: item.license_plate,
          in_black_list: item.in_black_list,
          parking_space_id: item.parking_space_id,
          parking_space_rent: item.parking_space_rent,
          parkingspace_owner_state: item.parkingspace_owner_state,
        };
        this.editlist = true;
        this.editid = item.id

      },

      //确认更新
      update() {
        //编辑的话，也是传id去服务端
        //axios.get('url',{ID:id}).then((res) =>{
        //			加载列表
        //})
        for (let i = 0; i < this.newsList.length; i++) {
          if (this.newsList[i].id == this.editid) {
            this.newsList[i] = {
              user_name: this.editDetail.user_name,
              license_plate: this.editDetail.license_plate,
              parking_space_id: this.editDetail.parking_space_id,
              in_black_list: this.editDetail.in_black_list,
              parking_space_rent: this.editDetail.parking_space_rent,
              parkingspace_owner_state: this.editDetail.parkingspace_owner_state,
              id: this.editid
            };
            this.editlist = false
          }
        }

        let json = JSON.stringify({
          "user_name": this.editDetail.user_name,
          "license_plate": this.editDetail.license_plate,
          "parking_space_id": parseInt(this.editDetail.parking_space_id),
          "in_black_list": parseInt(this.editDetail.in_black_list),
          "parking_space_rent": parseInt(this.editDetail.parking_space_rent),
          "parkingspace_owner_state": parseInt(this.editDetail.parkingspace_owner_state),
        });
        axios
          .put('http://118.31.77.203:8080/Entity/U21a840a21ebf11/PeterPark2/Parkingspaceowner/' + this.editid, json, {headers: {'Content-Type': 'application/json'}}
          ).then((res) => {
          console.log(res);
        })
      }
    }
  }
</script>
<style scoped>
  #table table {
    width: 100%;
    font-size: 14px;
    border: 1px solid #eee
  }

  #table {
    padding: 0 10px;
    margin-top: 80px;
    width: 100%;
  }

  table thead th {
    background: #f5f5f5;
    padding: 10px;
    text-align: left;
  }

  table tbody td {
    padding: 10px;
    text-align: left;
    border-bottom: 1px solid #eee;
    border-right: 1px solid #eee;
  }

  table tbody td span {
    margin: 0 10px;
    cursor: pointer;
  }

  .delete {
    color: red;
  }

  .edit {
    color: #008cd5;
  }

  .add {
    border: 1px solid #eee;
    margin: 10px 0;
    padding: 15px;
  }

  input {
    border: 1px solid #ccc;
    padding: 5px;
    border-radius: 3px;
    margin-right: 15px;
  }

  button {
    background: #008cd5;
    border: 0;
    padding: 4px 15px;
    border-radius: 3px;
    color: #fff;
  }

  #mask {
    background: rgba(0, 0, 0, .5);
    width: 100%;
    height: 100%;
    position: fixed;
    z-index: 4;
    top: 0;
    left: 0;
  }

  .mask {
    width: 300px;
    height: 350px;
    background: rgba(255, 255, 255, 1);
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    margin: auto;
    z-index: 47;
    border-radius: 5px;
  }

  .title {
    padding: 10px;
    border-bottom: 1px solid #eee;
  }

  .title span {
    float: right;
    cursor: pointer;
  }

  .content {
    padding: 10px;
  }

  .content input {
    width: 270px;
    margin-bottom: 15px;
  }
</style>

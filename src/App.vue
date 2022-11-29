<template>
<div style="position:relative;margin:auto;">
  <div>
    <input
        type="text"
        class="inputHash"
        v-model="hash" />
      <button type="button" class="search" @click="searchClick">搜索</button>
  </div>
  <div class="transition">
    <div v-for="(key,val,index) in hashData" :key="index" class="hashData">
      <div class="name"><span>{{val}}</span></div>
      <div class="value"><span>{{key}}</span></div>
    </div>
  </div>
  <div class="tranTit">Block Transactions</div>
  <table>
    <tr>
      <td>Free</td>
      <td>Hash</td>
      <td>Time</td>
    </tr>
    <tr v-for="(item,index) in hashTx" :key="'a'+index">
      <td>{{item.fee}}</td>
      <td>{{item.hash}}</td>
      <td>{{item.times}}</td>
    </tr>
  </table>
  <div v-show="statu" class='tost'>获取交易失败</div>
  <pagination
    :page="pageData.page"
    :page-size="pageData.pageSize"
    :total="pageData.pageTotal"
    :on-page-change="onPage"
    :showSizes="true"
    :pageSizeList="pageData.pageSizeList"
    :on-page-size-change="onSize"
    class="pagi page-content"
/>
</div>
</template>

<script>
import axios from 'axios';
import moment from 'moment'

import Pagination from './components/pagination.vue';
export default {
  name: "App",
  components: {
        Pagination,
  },
  data() {
    return {
      statu:false,
      hash:'00000000000000000007878ec04bb2b2e12317804810f4c26033585b3f81ffaa',
      hashData:{
        Hash:'',
        Bits:'',
        Fees:'',
        Height:'',
        MrklRoot:'',
        Nonce:'',
        Size:'',
        Weight:'',
      },
      hashTx:[],
      resTxTo:[],
      pageData: {
                pageTotal: 0,
                page: 1,
                pageSize: 10,
                pageSizeList: [10, 20, 30],
            }
    };
  },
  created(){
    this.getHashData(this.hash);
  },
  methods:{
    searchClick(){
      this.getHashData(this.hash)
    },
    getHashData(hashUrl){
      axios.get(`https://blockchain.info/rawblock/${hashUrl}`).then(res=>{
        console.log(res.data)
        let resData = res.data;
        this.hashData = {
          Hash:resData.hash,
          Time:moment(resData.time * 1000).format('YYYY-MM-DD hh:mm:ss'),
          Bits:resData.bits,
          Fees:resData.fee,
          Height:resData.height,
          MrklRoot:resData.mrkl_root,
          Nonce:resData.nonce,
          Size:resData.size,
          Weight:resData.weight,
        }
        resData.tx.map(item=>{
          item.times = moment(item.time * 1000).format('YYYY-MM-DD hh:mm:ss')
          item.fee = item.fee/100000000 +'BTC'
        })
        this.resTxTo = resData.tx;
        this.hashTx = resData.tx.slice((this.pageData.page-1)*10,this.pageData.page*10);
        this.pageData.pageTotal = resData.tx.length;
      }).catch(err=>{
        console.log(7777)
        this.statu = true;
        setTimeout(()=>{
          this.statu = false;
        },1500)
      })
    },
    // 分页
        onPage(pageNow) {
            console.log('pageNow', pageNow);
            this.pageData.page = pageNow;
            this.hashTx = this.resTxTo.slice((this.pageData.page-1)*10,this.pageData.page*10);
        },
        onSize(e) {
            this.pageData.pageSize = e;
        }
  }
}
</script>
<style>
.inputHash{
  outline: none;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  width: 40%;
  height: 30px;
  font-size: 14px;
  padding: 0 5px;
  margin-right: 10px;
}
.search{
  height: 30px;
  width: 60px;
  font-size: 14px;
  border: none;
  outline: none;
  cursor: pointer;
  text-decoration: none;
  color: #fff;
  background-color: #409eff;
  border-color: #409eff;
  border-radius: 4px;
}
.search:hover{
    background-color: #66b1ff;
    border-color: #66b1ff;
}
.search:active{
  background: #3a8ee6;
    border-color: #3a8ee6;
}
.transition{
  margin-top: 30px;
}
.hashData{
  width: 100%;
  display: flex;
  border-bottom: 1px solid #dcdfe6;
  padding: 10px 0;
}
.name{
  width:28%;
}
.name{
  display: flex;
  align-items: center;
  color: #606266;
}
.value{
  flex: 1;
  word-break: break-all;
  word-wrap: break-word;
  color: #606266;
}
.tranTit{
  font-weight: bold;
  margin: 20px 0;
  font-size: 18px;
}
table{
border-collapse:collapse;
width:100%;
}
tr td{
  word-break: break-all;
  word-wrap: break-word;
  border:1px solid black;
  padding: 5px 0 5px 5px;
}
.tost{
  background-color: #faecd8;
  color: #e6a23c;
  padding: 20px;
  display: inline-block;
  border-radius: 5px;
  position: absolute;
  left:50%;
  top:50%;
  transform: translate3d(-50%,-50%,0);
}
</style>

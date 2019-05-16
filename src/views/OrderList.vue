<template>
    <div>
      <nav-header></nav-header>
      <div class="container">
        <div class="page-title-normal">
          <h2 class="page-title-h2"><span>订单管理</span></h2>
        </div>
         <el-table
            :data="data"
            stripe
            style="width: 100%">
            <el-table-column
              prop="orderId"
              label="订单编号"
              width="180">
            </el-table-column>

            <el-table-column
              prop="createDate"
              label="创建日期"
              width="180">
            </el-table-column>
            <el-table-column
              prop='goodsList'
              label="订单商品"
              width="180">
            </el-table-column>
            <el-table-column
              prop="orderTotal"
              label="订单总价"
              width="180">
            </el-table-column>
             <el-table-column
              prop="orderStatus"
              label="订单状态"
              width="180">
            </el-table-column>
             <el-table-column
               label="操作">
                <template slot-scope="scope">
                  <el-button
                    size="mini"
                    @click="handleDetail(scope.$index, scope.row)">详情</el-button>
                    <el-button
                    size="mini"
                    type="success"
                    v-if="scope.row.orderStatus === '卖家已发货'"
                    @click="handleConfirm(scope.$index, scope.row)">确认收货</el-button>
                    <el-button
                    size="mini"
                    type="danger"
                    v-if="scope.row.orderStatus === '卖家已发货'"
                    @click="handleCancel(scope.$index, scope.row)">取消订单</el-button>
                  <el-button
                    size="mini"
                    type="danger"
                    @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                    <el-popover
                      placement="right"
                      width="400"
                      trigger="click">
                       <ul>
                         <li v-for="(item , index) in orderList[cIndex].goodsList">
                           <div>
                             <div style="display:inline-block;padding:16px;width:30%"> 
                              <span>{{item.productName}}</span><br>
                              <img
                                style="width: 60px; height: 60px;"
                                :src="'static/'+item.productImage"
                                >
                              </div>
                             <div style="display:inline-block;height:160px;text-align:right;width:60%"">
                                  <el-input type="textarea" v-model="comment[index]" style="padding:0"></el-input> 
                                  <el-button
                                      size="mini"
                                      type="warning"
                                      @click="handleComment(item.productId,comment[index])"
                                      style="margin-top:3px"
                                  >发布评论</el-button>
                             </div>
                          </div>
                          </li>
                       </ul>
                      <el-button
                        size="mini"
                        type="warning"
                        v-if="scope.row.orderStatus === '订单已完成'"
                        slot="reference"
                        @click="hoverComment(scope.$index, scope.row)"
                        >评价</el-button>
                      </el-popover>
                </template>
             </el-table-column>
          </el-table>
          <el-dialog title="订单详细" :visible.sync="detailVisible">
            <p><span class="label">订单编号:</span>{{detailData.orderId}}</p>
            <p><span class="label">订单商品:</span></p>
            <ul>
              <li v-for="item in detailData.goodsList">{{item.productName}}</li>
            </ul>
          </el-dialog>
      </div>
      <nav-footer></nav-footer>
    </div>
</template>
<script>
    import NavHeader from './../components/NavHeader'
    import NavFooter from './../components/NavFooter'
    import NavBread from './../components/NavBread'
    import {currency} from './../util/currency'
    import axios from 'axios'
    export default{
        data(){
            return{
              orderList:{},
              data:[],
              detailData:{},
              detailVisible:false,
              cIndex:0,
              comment:[],
            }
        },
        components:{
          NavHeader,
          NavFooter,
          NavBread
        },
        filters:{
          currency:currency
        },
        mounted(){
           this.getOrederList()
        },
        methods:{
           getOrederList(){
              this.orderList={},
              this.data=[],
                axios.get("/users/orderList").then((response)=>{
                  let res = response.data;
                  this.orderList=res.msg.orderList;
                  for (let item of this.orderList){
                    const i = {};
                    i.orderId = item.orderId;
                    i.createDate=item.createDate;
                    i.goodsList=item.goodsList[0].productName + '...';
                    if(item.orderStatus==0){
                      i.orderStatus='等待接单'
                    }else if (item.orderStatus==1){
                      i.orderStatus='卖家已发货'
                    }else if (item.orderStatus==2){ 
                      i.orderStatus='订单已完成'
                    }else if (item.orderStatus==3){ 
                      i.orderStatus='订单已取消'
                    };
                    i.orderTotal=item.orderTotal;
                    this.data.push(i)

                  }
                  console.log(this.orderList)
              });
            },
            handleDelete(index, row) {
               axios.post("/users/orderDel",{
                orderId:row.orderId
              }).then((response)=>{
                  let res = response.data;
                  if(res.status == '0'){
                    this.getOrederList();
                  }
              });
            },
            handleDetail(index,row){
              this.detailData=this.orderList[index]
              this.detailVisible=true
            },
            handleConfirm(index ,row){
              axios.post("/users/orderStatus",{
                orderId:row.orderId,
                orderStatus:2
              }).then((response)=>{
                  let res = response.data;
                  if(res.status == '0'){
                    this.getOrederList();
                  }
              });
            },
            handleCancel(index ,row){
              axios.post("/users/orderStatus",{
                orderId:row.orderId,
                orderStatus:3
              }).then((response)=>{
                  let res = response.data;
                  if(res.status == '0'){
                    this.getOrederList();
                  }
              });
            },
            hoverComment(index){
              this.cIndex = index ,
              this.comment=[],
              console.log(this.comment)
            },
            handleComment(productId,comment){
              console.log(comment)
              axios.post("/goods/addComment",{
                productId,
                comment
                })

            }
        }
    }
</script>

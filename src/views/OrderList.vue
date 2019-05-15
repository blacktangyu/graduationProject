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
                    @click="handleEdit(scope.$index, scope.row)">详情</el-button>
                  <el-button
                    size="mini"
                    type="danger"
                    @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                </template>
             </el-table-column>
          </el-table>
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
              data:[]
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
                    if(item.orderStatus==1){
                      i.orderStatus='已完成'
                    }else{
                      i.orderStatus='未完成'
                    };
                    i.orderTotal=item.orderTotal;
                    this.data.push(i)

                  }
                  console.log(this.orderList)
              });
            },
            handleDelete(index, row) {
              console.log(row.orderId);
               axios.post("/users/orderDel",{
                orderId:row.orderId
              }).then((response)=>{
                  let res = response.data;
                  if(res.status == '0'){
                    this.getOrederList();
                  }
              });
            }
        }
    }
</script>

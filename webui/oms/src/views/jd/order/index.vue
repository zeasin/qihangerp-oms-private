<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="88px">
      <el-form-item label="订单号" prop="orderId">
        <el-input
          v-model="queryParams.orderId"
          placeholder="请输入订单号"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="店铺" prop="shopId">
        <el-select v-model="queryParams.shopId" placeholder="请选择店铺" clearable @change="handleQuery">
          <el-option
            v-for="item in shopList"
            :key="item.id"
            :label="item.name"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="下单时间" prop="orderTime">
        <el-date-picker clearable
                        v-model="orderTime" value-format="yyyy-MM-dd"
                        type="daterange"
                        range-separator="至"
                        start-placeholder="开始日期"
                        end-placeholder="结束日期">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="订单状态" prop="orderState">
        <el-select v-model="queryParams.orderState" placeholder="请选择状态" clearable @change="handleQuery">
          <el-option label="等待出库" value="WAIT_SELLER_STOCK_OUT" ></el-option>
          <el-option label="等待确认收货" value="WAIT_GOODS_RECEIVE_CONFIRM"></el-option>
          <el-option label="等待发货" value="WAIT_SELLER_DELIVERY"> </el-option>
          <el-option label="POP暂停" value="POP_ORDER_PAUSE"></el-option>
          <el-option label="完成" value="FINISHED_L"></el-option>
          <el-option label="取消" value="TRADE_CANCELED"></el-option>
          <el-option label="父单取消" value="PARENT_TRADE_CANCELED"></el-option>
          <el-option label="已锁定" value="LOCKED"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">

      <el-col :span="1.5">
        <el-button
          :loading="pullLoading"
          type="success"
          plain
          icon="el-icon-edit"
          size="mini"
          @click="handlePull"
        >API拉取订单</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="el-icon-refresh"
          size="mini"
          :disabled="multiple"
          @click="handlePushOms"
        >重新推送选中订单到订单库</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="lists" @selection-change="handleSelectionChange">
       <el-table-column type="selection" width="55" align="center" />
<!--      <el-table-column label="订单ID" align="center" prop="orderId" />-->
<!--      <el-table-column label="店铺" align="center" prop="shopId" >-->
<!--        <template slot-scope="scope">-->
<!--          <el-tag size="small">{{shopList.find(x=>x.id === scope.row.shopId)?shopList.find(x=>x.id === scope.row.shopId).name:''}}</el-tag>-->
<!--        </template>-->
<!--      </el-table-column>-->
<!--      <el-table-column label="商品" width="350">-->
<!--        <template slot-scope="scope">-->
<!--          <el-row v-for="item in scope.row.items" :key="item.id" :gutter="20">-->

<!--            <div style="float: left;display: flex;align-items: center;" >-->
<!--&lt;!&ndash;              <el-image  style="width: 70px; height: 70px;" :src="item.picPath"></el-image>&ndash;&gt;-->
<!--              <div style="margin-left:10px">-->
<!--                <p>{{item.skuName}}</p>-->
<!--                <p>SKU编码：{{item.outerSkuId}}&nbsp;-->
<!--                </p>-->
<!--               <p>-->
<!--                 数量： <el-tag size="small">x {{item.itemTotal}}</el-tag>-->
<!--               </p>-->
<!--              </div>-->
<!--            </div>-->
<!--          </el-row>-->
<!--        </template>-->
<!--      </el-table-column>-->
      <el-table-column label="订单号" align="left" prop="orderId" width="220px">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-view"
            @click="handleDetail(scope.row)"
          > {{scope.row.orderId}}</el-button>
          <i class="el-icon-copy-document tag-copy" :data-clipboard-text="scope.row.orderId" @click="copyActiveCode($event,scope.row.orderId)" ></i>
        <br/>
          <el-tag type="info">{{ shopList.find(x=>x.id === scope.row.shopId) ? shopList.find(x=>x.id === scope.row.shopId).name : '' }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="商品明细" align="center" width="700px" >
        <template slot="header">
          <table>
            <th>
<!--              <td width="50px">图片</td>-->
              <td width="250px" align="left">商品名</td>
<!--              <td width="150" align="left">SKU名</td>-->
              <td width="200" align="left">Sku编码</td>
              <td width="150" align="left">平台SkuId</td>
              <td width="50" align="left">数量</td>
            </th>
          </table>
        </template>
        <template slot-scope="scope" >
          <el-table :data="scope.row.items" :show-header="false" :cell-style="{border:0 + 'px' }"  :row-style="{border:0 + 'px' }" >
<!--            <el-table-column label="商品图片" width="50px">-->
<!--              <template slot-scope="scope">-->
<!--                <image-preview :src="scope.row.picPath" :width="40" :height="40"/>-->
<!--              </template>-->
<!--            </el-table-column>-->
            <el-table-column label="商品名" align="left" width="250px" prop="skuName" />
<!--            <el-table-column label="SKU名" align="left" prop="goodsSpec" width="150"  :show-overflow-tooltip="true"/>-->
            <el-table-column label="Sku编码" align="left" prop="outerSkuId" width="200"/>
            <el-table-column label="平台SkuId" align="left" prop="skuId" width="150"/>
            <el-table-column label="商品数量" align="center" prop="itemTotal" width="50px">
              <template slot-scope="scope">
                <el-tag size="small" type="danger">{{scope.row.itemTotal}}</el-tag>
              </template>
            </el-table-column>
          </el-table>
        </template>
      </el-table-column>
      <el-table-column label="订单金额" align="left" prop="orderSellerPrice"  :formatter="amountFormatter" width="150px">
        <template slot-scope="scope">
        <div>
          <span style="font-size: 10px">货款金额：</span>
          <span>{{amountFormatter(null,null,scope.row.orderSellerPrice)}}</span>
        </div>
          <div>
            <span style="font-size: 10px">应付金额：</span>
            <span>{{amountFormatter(null,null,scope.row.orderPayment)}}</span>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="收件人" prop="receiverName" width="200px">
        <template slot-scope="scope">
          {{scope.row.fullname}}&nbsp;
          {{scope.row.mobile}} <br />
          {{scope.row.province}} {{scope.row.city}} {{scope.row.town}} <br />

        </template>
      </el-table-column>
<!--      <el-table-column label="应付金额" align="center" prop="orderPayment"  :formatter="amountFormatter"/>-->
<!--      <el-table-column label="收件人" align="center" prop="fullname" />-->
<!--      <el-table-column label="手机号" align="center" prop="mobile" />-->
      <el-table-column label="商家备注" align="center" prop="venderRemark" />
      <el-table-column label="状态" align="center" prop="orderStateRemark" >
        <template slot-scope="scope">
          <el-tag size="small" >{{scope.row.orderStateRemark}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="下单时间" align="center" prop="orderStartTime" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            :loading="pullLoading"
            icon="el-icon-refresh"
            @click="handlePullUpdate(scope.row)"
          >更新订单</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />
  </div>
</template>

<script>
import {listOrder, pullOrder, getOrder, pushOms, pullOrderDetail} from "@/api/jd/order";
import { listShop } from "@/api/shop/shop";
import {MessageBox} from "element-ui";
import {isRelogin} from "@/utils/request";
import Clipboard from "clipboard";
export default {
  name: "OrderJd",
  data() {
    return {
      // 遮罩层
      loading: true,
      pullLoading: false,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 商品管理表格数据
      lists: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      orderTime:null,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        name: null,
        startTime: null,
        endTime: null,
        image: null,
        orderState:'WAIT_SELLER_STOCK_OUT'
      },
      // 表单参数
      form: {},
      shopList: [],
      // 表单校验
      rules: {}
    };
  },
  created() {
    listShop({type:200}).then(response => {
      this.shopList = response.rows;
      if(this.shopList && this.shopList.length>0){
        this.queryParams.shopId = this.shopList[0].id
      }
      this.getList();
    });

  },
  methods: {
    copyActiveCode(event,queryParams) {
      console.log(queryParams)
      const clipboard = new Clipboard(".tag-copy")
      clipboard.on('success', e => {
        this.$message({ type: 'success', message: '复制成功' })
        // 释放内存
        clipboard.destroy()
      })
      clipboard.on('error', e => {
        // 不支持复制
        this.$message({ type: 'waning', message: '该浏览器不支持自动复制' })
        // 释放内存
        clipboard.destroy()
      })
    },
    amountFormatter(row, column, cellValue, index) {
      return '￥' + parseFloat(cellValue).toFixed(2).replace(/\d(?=(\d{3})+\.)/g, '$&,');
    },
    /** 查询商品管理列表 */
    getList() {
      if(this.orderTime){
        this.queryParams.startTime = this.orderTime[0]
        this.queryParams.endTime = this.orderTime[1]
      }else {
        this.queryParams.startTime = null
        this.queryParams.endTime = null
      }
      this.loading = true;
      listOrder(this.queryParams).then(response => {
        this.lists = response.rows;
        this.total = response.total;
        this.loading = false;
      });
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单重置
    reset() {
      this.form = {
        id: null,
        name: null,
        image: null,
        number: null
      };
      this.resetForm("form");
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm("queryForm");
      this.orderTime=null
      this.handleQuery();
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.orderId)
      this.single = selection.length!==1
      this.multiple = !selection.length
    },
    handlePushOms(row) {
      const ids = row.orderId || this.ids;
      this.$modal.confirm('是否手动推送到OMS？').then(function() {
        return pushOms({ids:ids});
      }).then(() => {
        // this.getList();
        this.$modal.msgSuccess("推送成功");
      }).catch(() => {});
    },
    handlePull(){
      if(this.queryParams.shopId){
        this.pullLoading = true
        pullOrder({shopId:this.queryParams.shopId,updType:0}).then(response => {
          console.log('拉取JD订单接口返回=====',response)
          if(response.code === 1401) {
            MessageBox.confirm('Token已过期，需要重新授权！请前往店铺列表重新获取授权！', '系统提示', { confirmButtonText: '前往授权', cancelButtonText: '取消', type: 'warning' }).then(() => {
              this.$router.push({path:"/shop/shop_list",query:{type:2}})
              // isRelogin.show = false;
              // store.dispatch('LogOut').then(() => {
              // location.href = response.data.tokenRequestUrl+'?shopId='+this.queryParams.shopId
              // })
            }).catch(() => {
              isRelogin.show = false;
            });

            // return Promise.reject('无效的会话，或者会话已过期，请重新登录。')
          }else if(response.code === 1402){
            this.$modal.msgError(JSON.stringify(response));
            this.pullLoading = false
          }else {
            this.$modal.msgSuccess(JSON.stringify(response));
            this.pullLoading = false
            this.getList()
          }

        })
      }else{
        this.$modal.msgSuccess("请先选择店铺");
      }
    },
    handlePullUpdate(row) {
      // 接口拉取订单并更新
      this.pullLoading = true
      pullOrderDetail({shopId:row.shopId,orderId:row.orderId}).then(response => {
        console.log('拉取JD订单接口返回=====',response)
        this.$modal.msgSuccess(JSON.stringify(response));
        this.pullLoading = false
      })
    }
  }
};
</script>

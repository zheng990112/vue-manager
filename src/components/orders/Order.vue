<template>
    <div>
        <el-card>
            <!--button按钮-->
            <el-row>
                <el-col>
                    <el-input placeholder="请输入内容" class="input-with-select" style="width:30%">
                        <el-button slot="append" icon="el-icon-search"></el-button>
                    </el-input>
                </el-col>
            </el-row>
            <!--订单列表表格区域-->
            <el-table
                    :data="ordersList.goods"
                    border
                    stripe
                    style="width: 100%">
                <el-table-column
                        type="index"
                        label="#">
                </el-table-column>
                <el-table-column
                        prop="order_number"
                        label="订单编号">
                </el-table-column>
                <el-table-column
                        prop="order_price"
                        label="订单价格">
                </el-table-column>
                <el-table-column
                        prop="pay_status"
                        label="是否付款">
                    <template #default="{ row }">
                        <el-tag type="danger" v-if="row.pay_status == 0">未付款</el-tag>
                        <el-tag type="success" v-else>已付款</el-tag>
                    </template>
                </el-table-column>
                <el-table-column
                        prop="is_send"
                        label="是否发货">
                </el-table-column>
                <el-table-column
                        prop="create_time"
                        label="下单时间">
                    <template #default="{ row }">
                        {{ new Date(row.create_time).toLocaleString() }}
                    </template>
                </el-table-column>
                <el-table-column
                        prop="address"
                        label="操作">
                </el-table-column>
            </el-table>
            <!--分页组件-->
            <el-pagination
                    @size-change="handleSizeChangeOrders"
                    @current-change="handleCurrentChangeOrders"
                    :current-page="queryInfo.pagenum"
                    :page-sizes="[5, 10, 15]"
                    :page-size="queryInfo.pagesize"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="totalOrders">
            </el-pagination>
        </el-card>
    </div>
</template>

<script>
    export default {
        name: 'Order',
        data () {
            return {
                ordersList: [],
                totalOrders: 0,
                // 分类的参数对象
                queryInfo: {
                    type: '',
                    pagenum: 1,
                    pagesize: 10
                }
            }
        },
        created () {
            this.getOrdersList()
        },
        methods: {
            async getOrdersList() {
                // 获取订单列表
                const { data: res } = await this.$http.get('orders', {
                    params: this.queryInfo
                })
                if (res.meta.status !== 200) return this.$message.error('获取失败')
                // 成功
                this.ordersList = res.data
                this.totalOrders = res.data.total
            },
            // 监听pagesize改变事件
            handleSizeChangeOrders (newSize) {
                this.queryInfo.pagesize = newSize
                // 重新获取数据
                this.getOrdersList()
            },
            // 监听page改变事件
            handleCurrentChangeOrders (newPage) {
                this.queryInfo.pagenum = newPage
                // 重新获取数据
                this.getOrdersList()
            }
        }
    }
</script>

<style scoped>

</style>

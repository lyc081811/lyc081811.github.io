 <template>
	<div>
		<el-dialog title="订单详情" :visible.sync="orderDetailsModal" width="1000px" :close-on-click-modal="false" @open="getDetailData" @close="clearParams">
			<div class="peopleData">
				<div class="searchParams">
					<el-input
						placeholder="请搜索"
						v-model="params.filter"
						@keyup.enter.native="filterData"
						style="width: 192px;">
					</el-input>
					<el-select v-model="params.sex" clearable placeholder="性别" style="margin: 0 8px;width: 75px;">
						<el-option label="男"	:value="1"></el-option>
						<el-option label="女"	:value="2"></el-option>
					</el-select>
					<el-select v-model="params.maritalStatus" clearable placeholder="婚姻状况" style="width: 100px;">
						<el-option label="未婚"	:value="1"></el-option>
						<el-option label="已婚"	:value="2"></el-option>
					</el-select>
					<el-select v-if="deptNames.length" v-model="params.deptName" clearable placeholder="部门" style="margin: 0 8px;width: 100px;">
						<el-option
							v-for="item in deptNames"
							:key="item.value"
							:label="item.label"
							:value="item.value"
						></el-option>
					</el-select>
					<el-select v-if="teamNames.length" v-model="params.teamName" clearable placeholder="组别" style="width: 100px;">
						<el-option
							v-for="item in teamNames"
							:key="item.value"
							:label="item.label"
							:value="item.value"
						></el-option>
					</el-select>
					<el-select v-model="params.isCheckBegin" clearable placeholder="是否到检" style="margin: 0 8px;width: 100px;">
						<el-option label="是"	:value="true"></el-option>
						<el-option label="否"	:value="false"></el-option>
					</el-select>
					<el-select v-model="params.isCheckEnd" clearable placeholder="是否完成" style="width: 100px;margin-right: 8px;">
						<el-option label="是"	:value="true"></el-option>
						<el-option label="否"	:value="false"></el-option>
					</el-select>
					<el-select
						v-model="params.batch"
						v-if="batchs.length"
						clearable
						style="margin-right: 8px;width: 193px;"
						placeholder="请选择批次">
						<el-option
							v-for="item in batchs"
							:key="item.value"
							:label="item.label"
							:value="item.value">
						</el-option>
					</el-select>
					<el-button @click="filterData">查询</el-button>
					<el-button @click="resetSearch">重置</el-button>
				</div>
				<div class="searchParams">
					<el-button-group>
						<el-button icon="el-icon-plus" @click="addPackage">添加项目</el-button>
						<el-button icon="el-icon-delete" @click="delPackage">删除项目</el-button>
						<el-button icon="el-icon-delete" @click="delCustomer">删除人员</el-button>
					</el-button-group>
				</div>
			</div>
			<el-table
				ref="table"
				:data="OrderDataShow"
				height="380"
				style="width: 100%"
				v-loading="loading"
				@row-click="clickRow2"
				@selection-change="handleSelectionChange">
				<el-table-column type="selection" fixed="left" width="55"></el-table-column>
				<el-table-column prop="customerName" fixed="left" label="姓名"></el-table-column>
				<el-table-column prop="sex" label="性别">
					<template slot-scope="scope">
						{{scope.row.sex === 1 ? '男' : '女'}}
					</template>
				</el-table-column>
				<el-table-column prop="tele" label="联系电话"></el-table-column>
				<el-table-column prop="cardNum" label="体检卡号"></el-table-column>
				<el-table-column prop="idcardNum" label="身份证号"></el-table-column>
				<el-table-column prop="maritalStatus" label="婚姻">
					<template slot-scope="scope">
						{{scope.row.maritalStatus === 1 ? '未婚' : '已婚' }}
					</template>
				</el-table-column>
				<el-table-column prop="deptName" label="部门"></el-table-column>
				<el-table-column prop="teamName" label="组别"></el-table-column>
				<el-table-column prop="diyFlag" label="定制类型">
					<template slot-scope="scope">
						{{scope.row.diyFlag === 1 ? '个性化（有限）': scope.row.diyFlag === 2 ? '个性化（无限）': scope.row.diyFlag === 0 ? '固定套餐' : ''}}
					</template>
				</el-table-column>
				<el-table-column  label="套餐名称">
					<template slot-scope="scope">
						{{scope.row.packages.length ? scope.row.packages[0].packageName : ''}}
					</template>
				</el-table-column>
				<el-table-column prop="remark" label="备注"></el-table-column>
				<el-table-column prop="orderMoney" label="金额">
					<template slot-scope="scope">
						￥{{scope.row.orderMoney | numFilter}}
					</template>
				</el-table-column>
				<el-table-column prop="isCheckBegin" label="是否到检">
					<template slot-scope="scope">
						{{scope.row.isCheckBegin ? '是' : '否'}}
					</template>
				</el-table-column>
				<el-table-column prop="isCheckEnd" label="是否完成">
					<template slot-scope="scope">
						{{scope.row.isCheckEnd ? '是' : '否'}}
					</template>
				</el-table-column>
				<el-table-column fixed="right" label="操作">
					<template slot-scope="scope">
						<el-button type="text" @click="getCustomerDetail(scope.row)">详情</el-button>
					</template>
				</el-table-column>
			</el-table>
			<div slot="footer" class="dialog-footer">
				<div style="float: left">
					<el-button type="text" @click="toggleSelection('OrderDataShow', 'table')">反选</el-button>
					<span class="subitem">
						合计：
						<span class="labelColor ftArial">{{OrderDataShow.length}}</span>
					</span>
					<span class="subitem">
						选中：
						<span class="labelColor ftArial">{{multipleSelection.length}}</span>
					</span>
				</div>

				<el-button @click="orderDetailsModal = false">取消</el-button>
				<el-button type="primary" @click="saveOrder">保存</el-button>
			</div>
		</el-dialog>
		<el-dialog title="添加项目" :visible.sync="addProjectModal" :close-on-click-modal="false" width="800px" @open="getDicItemList" @close="closeModal('projectDiscount')">
			<div class="addProject">
				<div class="project-left">
					<el-tabs v-model="activeTabName" type="card" @tab-click="tabsClick" class="addTabs" style="user-select: none;">
						<el-tab-pane label="按项目添加" name="third">
							<div class="modal-tree" style="border-top: none;">
								<div class="modal-top"  style="text-indent: 8px;">
									<el-input placeholder="请搜索" v-model="filterProject" style="width: 150px;" @keyup.enter.native="filterTree"></el-input>
									<div class="right" style="margin-right: 8px">
										<el-button @click="addBtn" v-no-more-click>添加</el-button>
									</div>
								</div>
								<div class="modal-con">
									<el-tree
										:data="GetItemListView"
										empty-text="正在获取中，请稍后..."
										show-checkbox
										node-key="id"
										@check="getNum"
										ref="third"
										@node-click="dbClick"
										:filter-node-method="filtertree">
									</el-tree>
								</div>
								<div class="fixBottom">
									<el-button type="text" @click="allTreeSelection('GetItemListView', 'third', GetItemListNum)">全选</el-button>
									<span class="subitem">合计： <span class="labelColor ftArial">{{GetItemListView.length}}</span></span>
									<span class="subitem">选中： <span class="labelColor ftArial">{{selectedThirdTotal || 0}}</span></span>
								</div>
							</div>
						</el-tab-pane>
						<el-tab-pane label="按科室添加" name="second">
							<div class="modal-tree" style="border-top: none;">
								<div class="modal-top"  style="text-indent: 8px;">
									<div class="right" style="margin-right: 8px">
										<el-button @click="addBtn" v-no-more-click>添加</el-button>
									</div>
								</div>
								<div class="modal-con">
									<el-tree
										:data="GetDeptList"
										empty-text="正在获取中，请稍后..."
										show-checkbox
										@check="getNum"
										ref="second"
										node-key="id"
										@node-click="dbClick"
										:filter-node-method="filtertree"
										>
									</el-tree>
								</div>
								<div class="fixBottom">
									<el-button type="text" @click="allTreeSelection('GetDeptList', 'second', GetDeptListNum)">全选</el-button>
									<span class="subitem">合计： <span class="labelColor ftArial">{{GetDeptListItems}}</span></span>
									<span class="subitem">选中： <span class="labelColor ftArial">{{selectedSecondTotal || 0}}</span></span>
								</div>
							</div>
						</el-tab-pane>
					</el-tabs>
				</div>
				<div class="project-right">
					<div class="modal-tree" style="margin-top: 40px;">
						<div class="modal-top">
							<span>已选项目</span>
						</div>
						<div class="modal-con">
							<el-tree
								:data="selectedListAll"
								node-key="id"
								ref="treeRight"
								:render-content="renderContent"
								>
							</el-tree>
						</div>
						<div class="fixBottom">
							<span class="subitem" style="margin-left: 8px;">合计： <span class="labelColor ftArial">{{selectedListAll.length}}</span></span>
						</div>
					</div>
				</div>
			</div>
			<div class="packageDis">
				<span class="item">原价：￥ <span>{{projectDiscount.totalPrice2 | numFilter}}</span></span>
				<el-checkbox v-model="isFree" v-if="USERINFO.canExcItemLimit">优惠自由</el-checkbox>
				折扣：<div class="item-list iptNum" style="display: inline-block">
								<el-input-number v-model="projectDiscount.discount" @blur="discountHandle('projectDiscount')" @change="discountHandle('projectDiscount')" :min="0" :max="1" :step="0.1"></el-input-number>
							</div>&nbsp;&nbsp;&nbsp;&nbsp;
				实收：<el-input v-model.number="projectDiscount.exePrice2" type="number" style="width: 100px;" @blur="realPriceHandle('projectDiscount')"></el-input>&nbsp;元
			</div>
			<div slot="footer" class="dialog-footer">
				<el-button @click="cancelBtn">取 消</el-button>
				<el-button type="primary" @click="confirmAddProjectBtn">确 定</el-button>
			</div>
		</el-dialog>
		<el-dialog title="删除项目" :visible.sync="delProjectModal" :close-on-click-modal="false" width="600px">
			<div class="modal-tree">
				<div class="modal-top">
					<span>项目名称</span>
				</div>
				<div class="modal-con" style="height:358px;">
					<el-tree
						:data="delCustomerPackage"
						show-checkbox
						node-key="id"
						ref="treeDel"
						@check="delTreeChange"
						>
					</el-tree>
				</div>
				<div class="fixBottom">
					<el-button type="text" @click="toogleTreeSelection('delCustomerPackage', 'treeDel')">反选</el-button>
					<span class="subitem">合计： <span class="labelColor ftArial">{{delCustomerPackage.length}}</span></span>
					<span class="subitem">选中： <span class="labelColor ftArial">{{delSelectNum || 0}}</span></span>
				</div>
			</div>
			<div slot="footer" class="dialog-footer">
				<el-button @click="cannelDelPackage">取 消</el-button>
				<el-button type="primary" @click="delPackageBtn">确 定</el-button>
			</div>
		</el-dialog>
		<el-dialog title="当前客户订单" :visible.sync="currentOrderModal" width="1000px" :close-on-click-modal="false" @close="cancelType">
			<div class="peopleData" v-if="isCanEdit">
				<div class="searchParams">
					<span style="margin-left: 8px;">定制类型：</span>
					<el-select v-model="currentType.diyFlag" clearable placeholder="定制类型" style="width: 130px;">
						<el-option label="固定套餐" :value="0"></el-option>
						<el-option label="个性化（有限）" :value="1"></el-option>
						<el-option label="个性化（无限）" :value="2"></el-option>
					</el-select>
					<el-input
						type="number"
						v-show="currentType.diyFlag === 1"
						min="0"
						style="width: 150px;margin: 0 8px;"
						placeholder="请输入限定价格"
						v-model.number="currentType.diyPrice"
						clearable>
					</el-input>
					<!-- <span>报告类型：</span>
					<el-select v-model="params.feetype" multiple collapse-tags placeholder="报告类型">
						<el-option v-for="item in batchs" :key="item.value" :label="item.label" :value="item.value"></el-option>
					</el-select> -->
				</div>
			</div>
			<el-table ref="table" :data="orderDetails.items" tooltip-effect="dark" max-height="400" style="width: 100%" @row-click="clickRow($event, 'table')"
				@selection-change="handleSelectionCur">
				<el-table-column prop="itemName" label="项目名称" :show-overflow-tooltip="true">
				</el-table-column>
				<el-table-column prop="isUnitItem" label="付费方式" width="100px" align="center">
					<template slot-scope="scope">
						{{scope.row.feeType ? scope.row.feeType :'个人'}}
					</template>
				</el-table-column>
				<el-table-column prop="fullPrice" label="原价" sortable width="150px" align="center">
					<template slot-scope="scope">
						￥{{scope.row.fullPrice | numFilter}}
					</template>
				</el-table-column>
				<el-table-column prop="exePrice" label="执行价格" sortable width="150px" align="center">
					<template slot-scope="scope">
						￥{{scope.row.exePrice | numFilter}}
					</template>
				</el-table-column>
				<el-table-column prop="checkStatus" label="项目状态" width="200px">
					<template slot-scope="scope">
						{{scope.row.isGiveUp ? '弃检' : scope.row.checkStatus === 1 ? '已检' : scope.row.checkStatus === 2 ? '已获取结果' : '未检'}}
					</template>
				</el-table-column>
			</el-table>
			<div slot="footer" class="dialog-footer">
				<el-button @click="currentOrderModal = false">取消</el-button>
				<el-button type="primary" @click="saveType">保存</el-button>
			</div>
		</el-dialog>
	</div>
</template>
<script>
import _ from 'lodash'
import moment from 'moment'
import PinyinMatch from 'pinyin-match'
import {mapState} from 'vuex'
export default {
	name: 'OrderDetails',
	data () {
		return {
			isFree: false,
			dbClickFlag: 0,
			isCanEdit: true,//订单详情操作框中是否允许操作
			loading: false,
			orderDetailsModal: false,
			addProjectModal: false,//添加项目
			delProjectModal: false,//删除项目

			currentOrderModal: false,//当前客户订单
			orderDetails: [],//当前客户项目详情

			// 订单详情中options
			teamNames: [],
			deptNames: [],
			batchs: [],//批次，需请求接口获取

			selectedThirdTotal: 0,
			selectedSecondTotal: 0,

			activeTabName: 'third',//默认
			params: {//筛选条件
				filter: '',
				sex: '',
				maritalStatus: '',
				deptName: '',
				teamName: '',
				isCheckBegin: '',
				isCheckEnd: '',
				batch: ''
			},
			delCustomerPackage: [],//要删除选中客户的项目的并集
			OrderParams: {},//父级传递的参数，用于获取订单列表
			OrderData: [],//订单详情数据集合
			OrderDataShow: [],//订单详情搜索后的数据
			multipleSelection: [],//订单详情选中的数据
			multipleSelectionCur: [],//订单项目选中的数据
			subOrderParams: { // 提交的数据对象
				GroupOrderCode: "",
				UnitCode: '',
				Operator: "001",
				Orders: []
			},

			GetItemList: [], // 检查项目字典源-添加项目
			GetItemListNum: 0, // 检查项目字典项目数-添加项目
			GetDeptList: [], // 检查科室字典源-添加项目
			GetDeptListNum: 0, // 检查科室字典项目数-添加项目
			GetItemListView: [], // 检查项目字典源-添加项目-弹窗中显示的数据

			selectedRight: [],// 右侧所选择的
			selectedListAll: [],// 右侧展示的树

			// tree - 筛选字段
			filterProject: '',

			selectedRightTotal: 0,
			delSelectNum: 0,

			currentType: {
				orderType: 0,
				diyFlag: 0,
				diyPrice: ''
			},

			projectDiscount: {//添加项目打折数据
				totalPrice: 0,
				totalPrice2: 0,
				discount: 0,
				discount2: 0,
				exePrice: 0,
				exePrice2: 0
			},
		}
	},
	created: function(){
	},
	methods: {
		//保存
		saveOrder(){
			let orders = []
			this.OrderData.forEach(x => {
				orders.push(this.toUpperCase(x));
			})
			this.subOrderParams.Orders = orders;
			this.$axios.post(this.$api.SubmitGroupOrder, this.subOrderParams).then(res => {
				if (res.data.status === 1) {
					this.$message.success('订单修改成功！');
					this.orderDetailsModal = false;
					this.$parent.$parent.getData();
				} else {
					this.$message.error(res.data.message);
				}
			}).catch(err => {
				this.$message.error(err.data.message);
			})
		},
		toUpperCase(obj) {
			let result = {};
				for (let key in obj) {
					if (obj.hasOwnProperty(key)) {
						let key2 = '';
						if (key.length > 0) {
							key2 = key.substr(0, 1).toUpperCase() + key.substr(1);
						}
						result[key2] = obj[key];
						if (Array.isArray(obj[key])) {
							obj[key].forEach((x,index) => {
								obj[key][index] = this.toUpperCase(x)
							})

						}
					}
				}
				return result;
		},
		//清空搜索参数
		clearParams() {
			this.params = {
				filter: '',
				sex: '',
				maritalStatus: '',
				deptName: '',
				teamName: '',
				isCheckBegin: '',
				isCheckEnd: '',
				batch: ''
			}
		},
		//订单详情-查询
		filterData(){
			let filter = this.params.filter;
			let sex = this.params.sex;
			let maritalStatus = this.params.maritalStatus;
			let deptName = '';
			let teamName = '';
			let batch = '';
			if (filter) {
				this.OrderDataShow = this.OrderData.filter(x => PinyinMatch.match(x.customerName, filter)
				)
			} else {
				this.OrderDataShow = this.OrderData;
			}
			if (sex) {
				this.OrderDataShow = this.OrderDataShow.filter(x => x.sex === sex);
			}
			if (maritalStatus) {
				this.OrderDataShow = this.OrderDataShow.filter(x => {
					if (x.maritalStatus === maritalStatus) {
						return x;
					}
				})
			}
			if (this.params.deptName || this.params.deptName === 0) {
				for(let i = 0; i < this.deptNames.length; i++) {
					if(this.deptNames[i].value === this.params.deptName) {
						deptName = this.deptNames[i].label;
						break;
					}
				}
				this.OrderDataShow = this.OrderDataShow.filter(x => {
					return	x.deptName === deptName;
				});
			}
			if (this.params.teamName || this.params.teamName === 0) {
				for(let i = 0; i < this.teamNames.length; i++) {
					if(this.teamNames[i].value === this.params.teamName) {
						teamName = this.teamNames[i].label;
						break;
					}
				}
				this.OrderDataShow = this.OrderDataShow.filter(x => {
					return x.teamName === teamName;
				})
			}
			if (this.params.isCheckBegin) {
				this.OrderDataShow = this.OrderDataShow.filter(x => {
					return x.isCheckBegin === this.params.isCheckBegin;
				})
			}
			if (this.params.isCheckEnd) {
				this.OrderDataShow = this.OrderDataShow.filter(x => {
					return x.isCheckEnd === this.params.isCheckEnd;
				})
			}
			if (this.params.batch) {
				this.OrderDataShow = this.OrderDataShow.filter(x => {
					return x.batchCode === this.params.batch;
				})
			}
		},
		//订单详情-重置
		resetSearch(){
			this.params.filter = '';
			this.params.maritalStatus = '';
			this.params.sex = '';
			this.params.deptName = '';
			this.params.teamName = '';
			this.params.isCheckBegin = '';
			this.params.isCheckEnd = '';
			this.params.batch = '';
			this.OrderDataShow = this.OrderData;
		},
		// 添加项目切换tab时fn
		tabsClick(tab){
			this.selectedLeft = [];
			this.$refs.second.setCheckedKeys([]);
			this.$refs.third.setCheckedKeys([]);
			if (tab.name === 'second' && this.GetDeptList.length === 0) {
				this.getDeptList();
			}
			if (tab.name === 'third' && this.GetItemList.length === 0) {
				this.getDicItemList();
			}
		},
		// 获取订单详情
		getDetailData(){
			this.loading = true;
			this.$axios.get(this.$api.GetGroupOrder, {params: {OrderCode: this.orderParams.orderCode}}).then(res => {
				if (res.data.status === 1) {
					this.deptNames = [];
					this.teamNames = [];
					this.batchs = [];
					this.OrderData = this.OrderDataShow = res.data.entity.orders;
					this.subOrderParams.UnitCode = res.data.entity.unitCode;
					res.data.entity.importBatches.forEach(x => {
						this.batchs.push({
							value: x.batchCode,
							label: moment(x.importTime).format('YYYY-MM-DD HH:mm:ss')
						})
					})
					this.OrderData.forEach((x, index) => {
						if(x.deptName) {
							this.deptNames.push({
								value: index,
								label: x.deptName
							});
						}
						if(x.teamName) {
							this.teamNames.push({
								value: index,
								label: x.teamName
							});
						}
					})
					this.deptNames = _.uniqBy(this.deptNames, 'label');
					this.teamNames = _.uniqBy(this.teamNames, 'label');
				} else {
					this.$message.error(res.data.message);
				}
					this.loading = false;
			}).catch(err => {
					this.$message.error(err.data.message);
					this.loading = false;
			})
		},
		//删除人员
		delCustomer(){
			if (this.multipleSelection.length === 0) {
				this.$message.error('请先选择后点击删除！');
				return;
			}
			let flag = '';
			this.multipleSelection.forEach(x => {
				if (x.unitPaidStatus !== 0) {
					flag += x.customerName + '，';
				}
			})
			if(flag.length) {
				this.$message.error(`${flag}不可删除`);
				return;
			}
			if (this.multipleSelection.every(x => x.isCheckBegin)) {
				this.$message.error('已到检查的不能删除');
				return;
			}
			 this.$confirm('是否确认删除选中人员?', '提示：', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
					this.OrderDataShow = this.OrderDataShow.filter(x => {
						return this.multipleSelection.every(y => y.idcardNum != x.idcardNum)
					})
					this.OrderData = this.OrderData.filter(x => {
						return this.multipleSelection.every(y => y.idcardNum != x.idcardNum)
					})
					this.$message.success('删除成功');
				}).catch(() => {})
		},
		//点击当前行选中
		clickRow2(row, event){
			if(event.label === '操作') {
				return;
			}
			this.$refs.table.toggleRowSelection(row);
		},
		handleSelectionChange(val) {
			this.multipleSelection = val;
		},
		handleSelectionCur(val) {
			this.multipleSelectionCur = val;
		},
		// 添加项目
		addPackage(){
			if (this.multipleSelection.length === 0 ) {
				this.$message.error('请先选择要添加项目的人员');
				return;
			}
			if(this.multipleSelection.every(x => x.isCheckEnd)){
				this.$message.error('已结束体检的不能追加项目');
				return;
			}
			this.addProjectModal = true;
		},
		// 添加项目-确定
		confirmAddProjectBtn(){
			//this.USERINFO.discountLowLimit = 0;
			let lowestPriceAll = 0;//项目最低价之和
			let exePriceAll = 0;//项目原执行价之和
			let fullPriceAll = 0;//项目原价之和
			this.selectedListAll.forEach(x => {
				lowestPriceAll += x.lowestPrice || 0;
				exePriceAll += x.exePrice || 0;
				fullPriceAll += x.fullPrice || 0;
			})
			let lowestDiscount = exePriceAll/fullPriceAll;
			if(this.isFree) {
				if(this.USERINFO.discountLowLimit > this.projectDiscount.discount2) {
					this.$message.error(`您最低的折扣为${this.USERINFO.discountLowLimit.toFixed(2)}`);
					return;
				}
			} else {
				lowestDiscount = this.USERINFO.discountLowLimit < lowestDiscount ? this.USERINFO.discountLowLimit : lowestDiscount;
				if(lowestDiscount > this.projectDiscount.discount2) {
					this.$message.error(`您最低的折扣为${lowestDiscount.toFixed(2)}`);
					return;
				}
				if (lowestPriceAll > this.projectDiscount.exePrice) {
					this.$message.error(`您最低的折扣金额为${lowestPriceAll.toFixed(2)}元`);
					return;
				}
			}
			if(this.projectDiscount.discount2 !== exePriceAll/fullPriceAll && exePriceAll !== this.projectDiscount.exePrice) {
				if(this.isFree) {
					this.selectedListAll.forEach(x => {
						x.exePrice = x.fullPrice * this.projectDiscount.discount2;
					})
				} else {
					this.distributePrice(this.selectedListAll, this.projectDiscount.exePrice, this.projectDiscount.discount2);
				}
			}
			this.multipleSelection = this.multipleSelection.map(x => {
				let arr = x.items.concat(_.cloneDeep(this.selectedListAll));
				x.items = _.uniqBy(arr, 'itemCode');
				x.orderMoney = 0;
				x.items.forEach(y => {
					x.orderMoney += y.exePrice;
				})
				return x;
			})
			this.cancelBtn();
			this.addProjectModal = false;
		},
		// 对项目执行价的处理
		distributePrice(data, exePriceAll, discount){//要处理的一维数组，执行价之和，折扣率
			let difPrice = exePriceAll;
			data = data.map((x, index) => {
				//x.exePrice2 = x.exePrice;
				x.exePrice = x.fullPrice * discount;
				if(x.exePrice <= x.lowestPrice) {
					difPrice = difPrice - x.lowestPrice;
					x.exePrice = x.lowestPrice;
				}
				return x;
			})
			if (difPrice !== exePriceAll ) {
				data = data.filter(x => {
					return x.exePrice > x.lowestPrice;
				})
				let lastFullPrice = 0;
				data.forEach(x => {
					lastFullPrice += x.fullPrice;
				})
				this.distributePrice(data, difPrice, difPrice / lastFullPrice);
			}
		},
		//添加项目添加按钮
		addBtn() {
			if (this.$refs[this.activeTabName].getCheckedNodes().length === 0) {
				this.$message.error('请先选择后点击添加！');
				return;
			}
			let flag = '';
			this.multipleSelection.forEach(x => {
				if (x.unitPaidStatus !== 0) {
					flag += x.customerName + '，';
				}
			})
			if(flag.length) {
				this.$message.error(`${flag}不可添加项目`);
				return;
			}
			let arr = this.$refs[this.activeTabName].getCheckedNodes().filter(x => {
				return !(x.children);
			})
			arr = this.selectedListAll.concat(arr);
			this.selectedListAll = _.uniqBy(arr, 'id');
			this.getDiscount('projectDiscount', this.selectedListAll);
			this.$refs[this.activeTabName].setCheckedKeys([]);
		},
		// 计算添加项目中打折折扣
		getDiscount(key, data){//key - 修改字符   data - 数据源
			this[key].totalPrice = 0;
			this[key].exePrice = 0;
			if(data.length === 0) {
				this[key].exePrice2 = 0;
				this[key].totalPrice2 = 0;
				this[key].discount2 = 0;
				this[key].discount = 0;
				return;
			}
			data = data.filter(x => {
				return x.itemCode;
			});
			data.forEach(x => {
				this[key].exePrice += x.exePrice || 0;
				this[key].totalPrice += x.fullPrice || 0;
			})
			this[key].discount = (parseInt(this[key].exePrice / this[key].totalPrice *100) / 100) || 0;
			this[key].discount2 = (this[key].exePrice / this[key].totalPrice) || 0;
			this.handleTwo(key);
		},
		//关于打折保留2位小数的处理
		handleTwo(key) {
			let num1 = this[key].totalPrice.toFixed(3);
			let num2 = this[key].exePrice.toFixed(3);
			this[key].totalPrice2 = Number(num1.substring(0, (num1 + '').length - 1));
			this[key].exePrice2 = Number(num2.substring(0, (num2 + '').length - 1));
			if(this[key].difPrice || this[key].difPrice == 0) {
				let num3 = this[key].difPrice.toFixed(3);
				this[key].difPrice2 = Number(num3.substring(0, (num3 + '').length - 1));
			}
		},
		//添加项目删除按钮
		removeBtn() {
			if (this.$refs.treeRight.getCheckedKeys().length === 0) {
				this.$message.error('请先选择后点击删除！');
				return;
			}
			let arr = this.$refs.treeRight.getCheckedKeys();
			this.selectedListAll = this.selectedListAll.filter(x => {
				return arr.every(y => y!== x.id);
			})
			this.getDiscount('projectDiscount', this.selectedListAll);
			this.selectedRight = [];
		},
		// 添加项目-取消
		cancelBtn(){
			this.addProjectModal = false;
			this.selectedRight = [];
			this.selectedListAll = [];
			this.$refs.second.setCheckedKeys([]);
			this.$refs.third.setCheckedKeys([]);
		},
		// 删除项目
		delPackage(){
			if (this.multipleSelection.length === 0 ) {
				this.$message.error('请先选择要删除项目的人员');
				return;
			}
			let flag = '';
			this.multipleSelection.forEach(x => {
				if (x.unitPaidStatus !== 0) {
					flag += x.customerName + '，';
				}
			})
			if(flag.length) {
				this.$message.error(`${flag}不可删除项目`);
				return;
			}

			this.delCustomerPackage = [];
			this.multipleSelection.forEach( x => {
				this.delCustomerPackage = this.delCustomerPackage.concat(x.items);
			})
			this.delCustomerPackage = _.uniqBy(this.delCustomerPackage, 'itemCode');
			this.handleTreeChildren(this.delCustomerPackage, 0);
			this.delProjectModal = true;
		},
		// 删除项目-取消
		cannelDelPackage(){
			this.delCustomerPackage = [];
			this.delProjectModal = false;
		},
		// 删除项目-确定
		delPackageBtn(){
			let msg = '';
			this.multipleSelection.forEach((x, index) => {
				x.items.forEach((y, i) => {
					this.$refs.treeDel.getCheckedKeys().forEach(z => {
						if (y.itemCode === z) {
							if (z.checkStatus) {
								msg += `${this.multipleSelection[index].customerName}的${y.customerName}项目已检，不可删除`;
							} else if(z.isGiveUp){
								msg += `${this.multipleSelection[index].customerName}的${y.customerName}项目弃检，不可删除`;
							} else {
								this.multipleSelection[index].items[i] = 0;
							}
						}
					})
				})
			})
			this.multipleSelection.forEach((x, index) => {
				let items = x.items.filter(y => {
					return y;
				})
				this.multipleSelection[index].items = _.cloneDeep(items);
				this.multipleSelection[index].orderMoney = 0;
				this.multipleSelection[index].items.forEach(y => {
					this.multipleSelection[index].orderMoney += y.exePrice;
				})
			})
			this.delCustomerPackage = [];
			this.delProjectModal = false;
		},
		// 获取单人项目详情(当前客户订单open)
		getCustomerDetail(data){
			//this.currentType.orderType = data.orderType;
			this.isCanEdit = data.unitPaidStatus === 0;
			this.currentType.diyFlag = data.diyFlag;
			this.orderDetails = data;
			this.currentOrderModal = true;
		},
			//当前客户订单-取消
		cancelType(){
			this.currentType.diyPrice = '';
			//this.currentType.orderType = '';
			this.currentType.diyFlag = '';
			this.currentOrderModal = false;
		},
		//当前客户订单-保存
		saveType(){
			if(this.currentType.diyFlag === 1) {
				if (!this.currentType.diyPrice && this.currentType.diyPrice != 0) {
					this.$message.error('请输入个性化套餐的限定金额');
					return;
				} else {
					this.orderDetails.UnitPayMoney = this.currentType.diyPrice;
				}
			}
			//this.orderDetails.orderType = this.currentType.orderType;
			this.orderDetails.diyFlag = this.currentType.diyFlag;
			this.currentOrderModal = false;
		},
		//关闭添加项目框后操作
		closeModal(key){
			this[key].totalPrice = 0;
			this[key].totalPrice2 = 0;
			this[key].discount = 0;
			this[key].discount2 = 0;
			this[key].exePrice = 0;
			this[key].exePrice2 = 0;
			this.isFree = false;
		},
		//获取检查项目字典
		getDicItemList(){
			this.$axios.get(this.$api.GetItemList).then(res => {
				if (res.status === 200 && res.data.status === 1) {
					this.GetItemList = this.GetItemListView =  res.data.entity;
					this.handleTreeChildren(this.GetItemListView, 3);
				} else {
					this.$message({
						type: 'error',
						message: err.data.message
					});
				}
			}).catch(err => {
				this.$message({
					type: 'error',
					message: err.data.message
				});
			})
		},
		//获取科室字典
		getDeptList(){
			this.$axios.get(this.$api.GetDeptListDD).then(res => {
				if (res.status === 200 && res.data.status === 1) {
					this.GetDeptList =  res.data.entity;
					this.handleTreeChildren(this.GetDeptList, 2);
				} else {
					this.$message.error(res.data.message);
				}
			}).catch(err => {
				this.$message.error(err.data.message);
			})
		},
		// 添加项目切换tab时fn
		tabsClick(tab){
			this.$refs.second.setCheckedKeys([]);
			this.$refs.third.setCheckedKeys([]);
			if (tab.name === 'second' && this.GetDeptList.length === 0) {
				this.getDeptList();
			}
			if (tab.name === 'third' && this.GetItemList.length === 0) {
				this.getDicItemList();
			}
		},
		//树的筛选
		filtertree(value, data) {
			if (!value) return true;
			return data.label.indexOf(value) !== -1;
		},

		// 处理字典tree结构
		handleTreeChildren(data, type){
			if (!data ) return;
			data.map((i,index) => {
				i.id = (i.packageCode || i.itemCode || i.deptCode || ('#' + index))
				i.label = (i.packageName || i.itemName || i.deptName || i.name)
				i.type = type;//1- 套餐 2- 科室  3-项目
				if (type === 1) {
					this.GetPackageFilterNum++;
				} else if(type === 2) {
					this.GetDeptListNum++;
				} else if(type === 3) {
					this.GetItemListNum++;
				}
				if (i.items && i.items.length > 0) {
					i.children = i.items;
					this.handleTreeChildren(i.children, type);
				}
			})
		},
		//添加项目右侧删除按钮
		remove(node, data) {
			const parent = node.parent;
			const children = parent.data.children || parent.data;
			const index = children.findIndex(d => d.id === data.id);
			children.splice(index, 1);
			this.getDiscount('projectDiscount', children);
		},
		renderContent(h, { node, data, store }) {
			return (
				<span style="flex: 1; display: flex; align-items: center; justify-content: space-between; font-size: 14px; padding-right: 8px;">
					<span>
						<span>{node.label}</span>
					</span>
					<span>
						<el-button style="font-size: 12px;" type="text" on-click={ () => this.remove(node, data) }>移除项目</el-button>
					</span>
				</span>);
		},
		// tree - 全选
		allTreeSelection(source, target, num){
			let selectedNum = this.$refs[target].getCheckedNodes().length;
			if (selectedNum === num) {
				this.$refs[target].setCheckedKeys([]);
				if(this.activeTabName === 'third') {
					this.selectedThirdTotal = 0;
				} else {
					this.selectedSecondTotal = 0;
				}
			} else {
				this.$refs[target].setCheckedNodes(this[source]);
				if(this.activeTabName === 'third') {
					this.selectedThirdTotal = this.GetItemListView.length;
				} else {
					this.selectedSecondTotal = this.GetItemListView.length;
				}
			}
		},
		// tree - 反选
		toogleTreeSelection(source, target){
			let toogle = this[source].filter(x => {
				return this.$refs[target].getCheckedNodes().every(y => y.id != x.id)
			})
			this.$refs[target].setCheckedKeys([]);
			this.$refs[target].setCheckedNodes(toogle);
		},
		// 点击tree节点计算选中数
		getSelectedRightTotal(data, checked, indeterminate){
			let num = 0;
			if (checked) ++num;
			this.selectedRightTotal = num;
		},
		// tree - 反选
		toogleTreeSelection(source, target){
			let toogle = this[source].filter(x => {
				return this.$refs[target].getCheckedNodes().every(y => y.id != x.id)
			})
			this.$refs[target].setCheckedKeys([]);
			this.$refs[target].setCheckedNodes(toogle);
			if(this.$refs.treeDel) {
				this.delSelectNum = this.$refs.treeDel.getCheckedNodes().length;
			}
		},
		//判断是否为项目
		arrPush(data){
			if (data.items && data.items.length > 0) {
				data.children.forEach(x => {
					this.arrPush(x);
				})
			} else if (!data.items) {
				this.selectedListAll.push(data);
			}
		},
		//双击事件
		dbClick(data){
			this.dbClickFlag++;
			if (this.dbClickFlag >= 2) {
				this.arrPush(data);
				this.selectedListAll = _.uniqBy(this.selectedListAll, 'id');
				this.getDiscount('projectDiscount', this.selectedListAll);
			}
			setTimeout(() =>{
				this.dbClickFlag = 0
			}, 500)
		},
		//按回车后对折扣的进行处理
		discountHandle(key) {
			if (this[key].discount >= 0 || this[key].discount <= 1) {
				this.isDisCount = true;
			} else {
				this.$message.error('请输入正确的折扣');
				return;
			}
			this[key].exePrice = (this[key].totalPrice * this[key].discount).toFixed(4);
			this[key].exePrice = Number(this[key].exePrice.substring(0, this[key].exePrice.length - 1));
			this[key].difPrice = this[key].totalPrice - this[key].exePrice;
			this[key].difPrice2 = parseInt(this[key].difPrice * 100) / 100;
			this[key].discount2 = this[key].discount;
			this.handleTwo(key);
		},
		//按回车后对折扣金额的进行处理
		difPriceHandle(key) {
			if(this[key].difPrice2 < 0 || this[key].difPrice2 > this[key].totalPrice) {
				this.$message.error('请输入正确的折扣金额');
				return;
			}
			this[key].difPrice = this[key].difPrice2;
			this[key].exePrice = this[key].totalPrice - this[key].difPrice;
			this[key].discount = parseInt(this[key].exePrice / this[key].totalPrice * 100) / 100;
			this[key].discount2 = this[key].exePrice / this[key].totalPrice;
			let num = this[key].exePrice.toFixed(3);
			this[key].exePrice2 = Number(num.substring(0, (num + '').length - 1));
		},
		//按回车后对实收金额的进行处理
		realPriceHandle(key) {
			if(this[key].exePrice2 > this[key].totalPrice || this[key].exePrice2 < 0) {
				this.$message.error('请输入正确的折扣金额');
				return;
			}
			this[key].exePrice = this[key].exePrice2;
			this[key].discount = parseInt(this[key].exePrice / this[key].totalPrice * 100) / 100;
			this[key].discount2 =this[key].exePrice / this[key].totalPrice;
			this[key].difPrice = this[key].totalPrice - this[key].exePrice;
			this[key].difPrice2 = parseInt(this[key].difPrice * 100) / 100;
			let num = this[key].difPrice.toFixed(3);
			this[key].difPrice2 = Number(num.substring(0, (num + '').length - 1));
		},
		delTreeChange(a,b) {
			this.delSelectNum = b.checkedKeys.length;
		},
		getNum(a,b){
			if(this.activeTabName === 'third') {
				this.selectedThirdTotal = b.checkedKeys.length;
			} else {
				this.selectedSecondTotal = b.checkedKeys.filter(x => x.indexOf('K') != 0).length;
			}
		}
	},
	computed: {
		...mapState([
			'USERINFO'
		]),
		GetDeptListItems: function() {
			let num = 0;
			this.GetDeptList.forEach(x => {
				if(x.children) {
					x.children.forEach(y => {
						num ++;
					})
				}
			})
			return num;
		}
	},
	watch: {
		filterProject: function(val, oldVal) {
			if(val != oldVal) {
				if(val) {
					this.GetItemListView = this.GetItemList.filter(x => {
						return PinyinMatch.match(x.itemName, val);
					})
				} else {
					this.GetItemListView = this.GetItemList;
				}
			}
		}
	}
}
</script>
<style scoped>
.addProject{
	overflow: hidden;
}
.addProject .project-left{
	width: 49%;
	float: left;
}
.addProject .project-right {
	width: 49%;
	float: right;
}
.addProject >>> .el-tabs__nav-scroll {
	padding-left: 0;
}
.searchParams >>> .el-tag--small {
	height: 21px;
}
</style>

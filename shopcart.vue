<template>
	<div>
		<header class="mui-bar mui-bar-nav">
			<!--<a class="mui-action-back mui-icon mui-icon-left-nav mui-pull-left"></a>-->
			<h1 class="mui-title" style="font-size:16px;">购物车</h1>
		</header>
		<div v-if="shDelete">
			<div class="bian" @click="Ondelete" v-if="UnDelete">编辑</div>
			<div class="bian" @click="Updelete" v-else>完成</div>
		</div>
		<div class="shopcart">
			<!--<div id="mescroll40" class="mescroll">-->
			<div class="mui-card">
				<ul class="shopbus mui-input-group" id="result" v-if="getGou">
					<li v-for="(list,index) in shopcart">
						<div class="upper clear">
							<div class="mui-input-row mui-checkbox">
								<input name="checkbox1" type="checkbox" @change="onChange(index,list)">
							</div>
							<div @click="" class="clear">
								<div style="float:left;">{{list.shopname}}</div>
								<!--<span style="float:right">&gt;</span>-->
							</div>

						</div>
						<div class="mui-input-group">
							<ul class="mid ">
								<li class="clear" v-for="(data,index1) in list.goods">
									<div class="mui-input-row mui-checkbox">
										<input name="checkbox2" type="checkbox" :tar="data.id" :id="data.id" :value="data.id" :g="data.sid" :p="getPri(data.totmoney,data.num)" @change="onChange1(index,list,data)">
									</div>
									<div @click="Detail(data.sid,data.id,data.gid,data.type)">
										<div class="simg">
											<img v-lazy="data.gimg" alt="" />
										</div>
										<div class="sdetail">
											<div class="stitle">{{data.gname}}</div>
											<div class="guige">规格：{{data.products}}</div>
											<div class="smoney">
												<span>￥{{data.totmoney}}</span>
												<!--<span style="float:right">x{{data.num}}</span>-->
											</div>
										</div>
									</div>
									<div class="numb">
										<input class="add" type="button" value="-" @click="addtion(index,index1,data.id,data.gid,data.num,1,list)" /><input class="add num" style="width:40px" type="number" :id="data.id" :value="data.num" /><input class="add" type="button" value="+" @click="addtion(index,index1,data.id,data.gid,data.num,2,list)" />
										<!--@click="shopcartadd(data.id,data.gid,data.num,2)"-->
									</div>
								</li>

							</ul>
						</div>

						<div class="lower clear">
							<div class="stotal">总计：<span :id="list.sid">0</span></div>
							<div class="jiesuan" @click="jiesuan(list.sid)">结算</div>
						</div>
					</li>

				</ul>
				<div class="wu" v-if="!getGou">
					<img src="static/img/pingji.png" />
					<span>购物车里还没有商品，快去添加吧！</span>
				</div>
			</div>

			<!--</div>-->

		</div>
		<div id="Select" v-if="item1">
			<div class="mui-input-row mui-checkbox mui-left">
				<label>全选</label>
				<input name="checkbox" value="" type="checkbox" id="btnd" @change="getAll" v-model="btndVal">
			</div>
			<div class="shan">
				<button id="quxiao" @click="Updelete">取消</button><button id="delete" @click="Hide">删除</button>
			</div>
		</div>
		<div class="isLoadingBox">
			<v-loading v-if="isLoading" type="bars" color="#fc0000" style="width:50px;height:50px;margin-top:5px;"></v-loading>
		</div>
	</div>
</template>

<script>
	import vLoading from 'vue-loading-template'
	export default {
		data() {
			return {
				isLoading: false,
				showNum: 15,
				toPage: 1,
				item1: false,
				uid: '',
				shopcart: [],
				shDelete: false,
				UnDelete: true,
				btndVal: false,
				totalmoney: 0,
				getGou: true,
				num: '1',
			}
		},
		methods: {
			//购物车加减
			addtion(index, index1, id, gid, num, type, list) {
				var vm = this;
				if(type == 2) {
					$("#" + id).val(++num);
					var num1 = $("#" + id).val();
					vm.shopcart[index].goods[index1].num = num1;
				} else {
					if(num < 2) {
						mui.toast("数量不能再少了哦");
						return;
					}
					$("#" + id).val(--num);
					var num1 = $("#" + id).val();
					vm.shopcart[index].goods[index1].num = num1;
				}
				var schecked = $("#result>li:eq(" + index + ") input[name='checkbox1']").prop("checked");
				var chec = $("#result>li:eq(" + index + ") input[tar='" + id + "']").prop("checked");
				var price = 0;
				var parames = {
					uid: vm.uid,
					id: id,
					gid: gid,
					num: num1
				}
				$.post(sendHttp + "home/shoptrolley/shoptrolley_money", JSON.stringify(parames)).then(function(res) {
					try {
						var obj = eval("(" + res + ")");
						if(obj.code == 1) {
							vm.getMyshou();
							setTimeout(function() {
								if(schecked) {
									$("#result>li:eq(" + index + ") input[name='checkbox2']").prop("checked", true);
									for(var i = 0; i < list.goods.length; i++) {
										price += Number(Number(list.goods[i].totmoney) * Number(list.goods[i].num));
										price = Math.floor(price * 100) / 100;
										$("#" + list.sid).html(price);
										store.set("tp",price);
									}
								} else {
									if(chec) {
										var pr = 0;
										$("[g='" + list.sid + "'][name='checkbox2']:checked").each(function() {
											var p = $(this).attr("p");
											pr += Number(p)
											pr = Math.floor(pr * 100) / 100;
											$("#" + list.sid).html(pr);
										})
									}

								}
							}, 100)

						}
					} catch(e) {
						vm.isLoading = false;
					}
					vm.isLoading = false;
				}, function(err) {
					mui.toast("亲！您的网络环境不佳！");
				})
			},

			//点击进店铺
			Shop(sid) {
				var vm = this;
				store.set("psid", sid);
				vm.$router.push('/Dianpu');
			},
			//结算商品
			jiesuan(sid) {
				var vm = this;
				var slength = $("[g='" + sid + "'][name='checkbox2']:checked").length;

				if(slength > 0) {
					var ids = "";
					$("[g='" + sid + "'][name='checkbox2']:checked").each(function() {
						var id = $(this).val();
						ids = ids == '' ? id : ids + "," + id
					})
					store.set("ids", ids);

					vm.$router.push('/cartque')
				} else {
					mui.toast("您还没有选择商品")
				}

			},
			//获取购物车总价
			getPri(pri, num) {
				var pris = 0;
				pris = eval(pri) * eval(num);
				pris = Math.floor(pris * 100) / 100;
				return pris;
			},
			//展示编辑购物车
			Ondelete() {
				var vm = this;
				vm.item1 = true;
				vm.UnDelete = false;
				vm.btndVal = false;
			},
			//展示完成购物车
			Updelete() {
				var vm = this;
				vm.item1 = false;
				vm.UnDelete = true;
				vm.btndVal = false;
			},
			//店铺的选择事件
			onChange(index, list) {
				store.remove("selid");
				var vm = this;
				var schecked = $("#result>li:eq(" + index + ") input[name='checkbox1']").prop("checked");
				var chec = $("#result>li input[name='checkbox1']:checked").length;
				if(chec == vm.shopcart.length) {
					vm.btndVal = true;
					$("#Select .mui-input-row label").text("全选");
				} else {
					vm.btndVal = false;
					$("#Select .mui-input-row label").text("取消全选");
				}
				if(schecked) {
					store.set("quan", 1);
					var price = 0;
					$("#result>li:eq(" + index + ") input[name='checkbox2']").prop("checked", true);
					for(var i = 0; i < list.goods.length; i++) {
						price += Number(Number(list.goods[i].totmoney) * Number(list.goods[i].num));
						price = Math.floor(price * 100) / 100;
						$("#" + list.sid).html(price);
						store.set("tp",price);
					}
				} else {
					store.set("quan", 2)
					$("#" + list.sid).html(0);
					store.remove("ids");
					
					$("#result>li:eq(" + index + ") input[name='checkbox2']").prop("checked", false);
				}
			},
			//商品的选择事件
			onChange1(index, list) {
				var vm = this;
				var schecked = $("#result>li:eq(" + index + ") input[name='checkbox2']:checked").length;
				var shop = vm.shopcart[index].goods.length;
				var sprice = 0;
				if(schecked == shop) {
					store.set("quan", 1)
					var price = 0;
					$("#result>li:eq(" + index + ") input[name='checkbox1']").prop("checked", true);
					var chec = $("#result>li input[name='checkbox1']:checked").length;
					for(var i = 0; i < list.goods.length; i++) {
						price += Number(Number(list.goods[i].totmoney) * Number(list.goods[i].num));
						list.summoney = price;
						price = Math.floor(price * 100) / 100;
						$("#" + list.sid).html(price);
						store.set("tp", price);
					}
					if(chec == vm.shopcart.length) {
						vm.btndVal = true;
						$("#Select .mui-input-row label").text("全选");
					} else {
						vm.btndVal = false;
						$("#Select .mui-input-row label").text("取消全选");
					}
				} else {
					store.set("quan", 2)
					$("#result>li:eq(" + index + ") input[name='checkbox1']").prop("checked", false);
					var ids = '';
						$("[g='"+list.sid+"'][name='checkbox2']:checked").each(function(){
							var id=$(this).val();
							ids=ids==''?id:id+","+ids;
							store.set("ids",ids);
							store.set("tpid",list.sid);
						})
					vm.btndVal = false;
					$("#Select .mui-input-row label").text("取消全选");
				}
			},
			//全选删除
			getAll() {
				var vm = this;
				if(vm.btndVal) {
					$("#result input[name='checkbox1']").each(function() {
						$("#result input[name='checkbox1']").prop("checked", "true");
						$("#result input[name='checkbox2']").prop("checked", "true");
						$("#Select .mui-input-row label").text("全选");
					})
				} else {
					$("#result input[name='checkbox1']").each(function() {
						$("#result input[name='checkbox1']").prop("checked", function() {
							return false;
						});
						$("#result input[name='checkbox2']").prop("checked", function() {
							return false;
						});
						$("#Select .mui-input-row label").text("取消全选");
					})
				}
			},
			//选择商品的id
			getDeleteAll() {
				var ids = "";
				$("#result input[name='checkbox2']:checked").each(function() {
					var id = $(this).val();
					ids = ids == "" ? id : ids + "," + id;
				});
				return ids;
			},
			sweetAlertFun(type, text, time) {
				swal({
					title: "",
					text: text,
					type: type,
					timer: time,
					showConfirmButton: false
				});
			},
			//是否删除购物车商品
			Hide() {
				var vm = this;
				var id = vm.getDeleteAll();
				if(id != "") {
					setTimeout(function() {
						swal({
								title: "确定删除商品吗",
								type: "warning",
								showCancelButton: true,
								confirmButtonText: "确定",
								cancelButtonText: "取消",
								closeOnConfirm: false,
								allowOutsideClick: true
							},
							function() {
								vm.getDelete();
							});
					}, 100);
					return;
				} else {
					vm.sweetAlertFun("error", "请选择要删除的商品", 1500);
				}

			},
			//购物车商品删除
			getDelete() {
				var vm = this;
				var id = vm.getDeleteAll();
				if(id != "") {
					var parames = {
						id: id,
					}
					vm.isLoading = true;
					$.post(sendHttp + "home/shoptrolley/delshoptrolley", JSON.stringify(parames)).then(function(res) {
						try {
							var obj = eval("(" + res + ")");
							if(obj.code == 1) {
								vm.sweetAlertFun("success", "删除成功", 2000);
								vm.getMyshou();
							} else {
								vm.sweetAlertFun("error", "删除失败", 2000);
							}
						} catch(e) {
							vm.isLoading = false;
						}
						vm.isLoading = false;
					}, function(err) {
						mui.toast("亲！您的网络环境不佳！");
					})
				}

			},
			//上拉
			upCallback() {
				var vm = this;
				vm.getMyshou();
			},
			mergeJSON(json1, json2) {
				var str1 = JSON.stringify(json1);
				var str2 = JSON.stringify(json2);
				var str3 = "";
				if(str1 == "{}" || str1 == null || str1 == "" || str1 == "[]") {
					str3 = str2;
				} else {
					str1 = str1.replace("[", "").replace("]", "");
					str2 = str2.replace("[", "").replace("]", "");
					str3 = "[" + str1 + "," + str2 + "]";
				}
				str3 = str3.replace("]]", "]");
				if(str3 != '"[]"') {
					this.huod = eval("(" + str3 + ")");
					store.set("huod", str3);
				} else {
					this.huod = {};
				}
			},
			// 调用购物车数据
			getMyshou() {
				var vm = this;
				var parames = {
					uid: vm.uid,
					/*psize: vm.showNum,
					page: vm.toPage*/
				}
				vm.isLoading = true;
				$.post(sendHttp + "home/shoptrolley/shoptrolleylist", JSON.stringify(parames)).then(function(res) {
					try {
						var obj = eval("(" + res + ")");
						if(obj.code == 1) {
							vm.shDelete = true;
							vm.shopcart = obj.data;
							setTimeout(function() {
								$("[name='checkbox2']").click(function() {
									var g = $(this).attr("g");
									var tp = 0;
									$("[g='" + g + "'][name='checkbox2']:checked").each(function() {
										var p = $(this).attr("p");
										tp += eval(p);
										tp = Math.floor(tp * 100) / 100;
									})
									$("#" + g).html(tp);
									store.set("tpid", g);
									store.set("tp", tp);
								})
							}, 200)

						} else {
							vm.shDelete = false;
							vm.item1 = false;
							vm.shopcart = [];
							vm.getGou = false;
						}
						vm.isLoading = false;
					} catch(e) {
						vm.isLoading = false;
						mui.toast("服务器开小差")
					}
				}, function(err) {
					vm.isLoading = false;
					mui.toast("亲！您的网络环境不佳！");
				})
			},
			// 跳转商品详情
			Detail(sid, id, gid, type) {
				var vm = this;
				store.set("gid", gid);
				store.set("ty", type);
				//				var tp=$("[id='"+id+"'][name='checkbox2']").attr("p");
				//				$("#"+sid).html(tp);
				//				store.set("tpid",sid);
				//				store.set("tp",tp);
				//store.set("selid",id);//哪个是选中的商品
				this.$router.push({
					path: '/Sear_detail'
				});
			},
		},
		mounted() {
			var vm = this;
			vm.uid = store.get("uid");
			$("#nav a:eq(1)").addClass("router-link-active").siblings().removeClass("router-link-active");
			$("#nav a:eq(1) span").addClass("tu_2").parent().parent().siblings().children().children().removeClass();
			//			vm.mescroll = new MeScroll("mescroll40", {
			//				down: {
			//					use: false
			//				},
			//				up: {
			//					callback: vm.upCallback,
			//					htmlNodata: '<p class="upwarp-nodata">没有更多记录啦~~</p>'
			//				}
			//			});
			//			vm.upCallback();
			vm.getMyshou();
			var quan = store.get("quan");
			var tpid = store.get("tpid");
			var ids=store.get("ids");
			var tp=store.get("tp");
			var pr = 0;
			setTimeout(function() {
					if(quan==1){
						$("#result>li:eq("+0+") input[name='checkbox1']").prop("checked",true);
						$("#result>li:eq("+0+") input[name='checkbox2']").prop("checked",true);
						$("#"+tpid).html(tp);
					}else{
						var id=	ids.split(",");
						for(var i=0;i<id.length;i++){
							$("[id='"+id[i]+"'][name='checkbox2']").prop("checked","true");
						}
						$("[g='" + tpid + "'][name='checkbox2']:checked").each(function() {
							var p = $(this).attr("p");
							pr += Number(p)
							pr = Math.floor(pr * 100) / 100;
							$("#" + tpid).html(pr);
						})
						
						
					}

			}, 100)

		},
		components: {
			vLoading,
		},
	}
</script>

<style scoped>
	* {
		box-sizing: border-box;
	}
	
	input[type=button]:enabled:active {
		background: #fff;
		color: #9C9C9C;
	}
	
	input[type=number] {
		float: none;
	}
	/*购物车数量加减*/
	
	.numb {
		position: absolute;
		bottom: 5px;
		right: 3px;
		border: 1px solid #eee;
		border-radius: 3px;
		width: 100px;
	}
	
	.numb .add {
		padding: 0;
		margin: 0;
		display: inline-block;
		width: 28px;
		height: 26px;
		line-height: 26px;
		color: #9C9C9C;
		font-size: 16px;
	}
	
	.numb .num {
		text-align: center;
		border-right: 1px solid #eee;
		border-left: 1px solid #eee;
	}
	
	.wu {
		width: 100%;
		margin: 40px auto;
		text-align: center;
	}
	
	.wu img {
		width: 140px;
		height: 140px;
	}
	
	.wu span {
		margin-top: 20px;
		display: block;
		font-size: 14px;
		color: #bfbfbf;
	}
	
	.shopcart {
		background: #eee;
		padding: 0 10px;
		position: fixed;
		top: 44px;
		bottom: 45px;
		left: 0;
		right: 0;
		overflow: scroll;
	}
	
	#Select {
		position: fixed;
		bottom: 0px;
		z-index: 10000;
		height: 45px;
		width: 100%;
		background: #fff;
	}
	
	.sel {
		float: left;
	}
	
	#quxiao,
	#delete {
		width: 80px;
		height: 45px;
		background: #EAAA50;
		border-radius: 0;
		/*opacity: 0.9;*/
		color: #fff;
		outline: none;
		border: none;
	}
	
	#delete {
		background: #DB3737;
	}
	
	.mui-checkbox.mui-left label,
	.mui-radio.mui-left label {
		padding-left: 55px;
		font-size: 14px;
		padding-top: 15px;
	}
	
	.mui-checkbox {
		width: 130px;
		float: left;
	}
	
	.shan {
		float: right;
	}
	
	.bian {
		position: fixed;
		top: 8px;
		right: 10px;
		z-index: 1000;
		border-radius: 5px;
		height: 30px;
		line-height: 30px;
		width: 70px;
		color: #fff;
		font-size: 14px;
		text-align: center;
	}
	
	.mui-input-group:before {
		height: 0;
	}
	
	.mui-input-group:after {
		height: 0
	}
	
	.mui-input-group .mui-input-row {
		position: absolute;
		width: 40px;
		height: 40px;
		top: 19px;
		left: 15px;
	}
	
	.mui-checkbox input[type=checkbox]:before {
		font-size: 24px;
	}
	
	.mui-checkbox input[type=checkbox]:checked:before,
	.mui-radio input[type=radio]:checked:before {
		color: #DB3737;
	}
	
	.mui-input-group .mui-input-row:after {
		height: 0 !important;
	}
	
	.mui-checkbox input[type=checkbox] {
		top: 0;
		left: 0;
	}
	
	.mui-card {
		font-size: 14px;
		overflow: hidden;
		background-color: #eee;
		margin: 0 !important;
		box-shadow: none !important;
	}
	
	#Select .mui-checkbox input[type=checkbox] {
		top: 9px;
		left: 22px;
	}
	
	#result {
		background: #EEEEEE;
	}
	
	#result>li {
		color: #9C9C9C;
		background: #fff;
		border-radius: 10px;
		padding: 10px;
		margin: 10px 0;
		position: relative;
	}
	
	.upper {
		height: 40px;
		line-height: 40px;
		border-bottom: 1px solid #F6F6F6;
		padding-left: 35px;
	}
	
	.mid>li {
		padding: 10px 0;
		border-bottom: 1px solid #F6F6F6;
		padding-left: 35px;
		position: relative;
	}
	
	.simg {
		float: left;
		margin-right: 10px;
		width: 27%;
	}
	
	.simg img {
		width: 80px;
		height: 80px;
	}
	
	.sdetail {
		float: left;
		width: 67%;
	}
	
	.guige {
		font-size: 12px;
	}
	
	.mid .mui-input-row {
		position: absolute;
		width: 40px;
		height: 40px;
		top: 35px;
		left: 4px;
	}
	
	.stitle {
		color: #333;
		font-size: 13px;
		display: -webkit-box;
		-webkit-box-orient: vertical;
		-webkit-line-clamp: 2;
		overflow: hidden;
	}
	
	.smoney {
		color: #DB3736;
		font-size: 16px;
	}
	
	.lower {
		height: 40px;
		line-height: 40px;
		font-size: 16px;
		;
	}
	
	.lower span {
		color: #DB3736;
	}
	
	.stotal {
		float: left;
	}
	
	.jiesuan {
		float: right;
		height: 30px;
		line-height: 30px;
		width: 60px;
		background: #DB3736;
		color: #fff;
		text-align: center;
		border-radius: 5px;
		margin-top: 10px;
	}
</style>
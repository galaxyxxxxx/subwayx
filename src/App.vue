<template>
	<div id="app">
		<my-head />
		
		<div id="container">
		<!-- 上传文件组件 -->
		<Upload msg="上传文件" @txt="handleTxt" />
		<!-- 输入框 -->
		<input-form v-if="dist.length > 0" @input="handleInput" class="inputForm"/>
		<!-- 价格表 -->
		<price-table v-if="dist.length > 0" :lineSite="lineSite" :price="cost" class="priceTable"/>

		</div>
		
		<my-foot />
	</div>
</template>

<script>
	// 组件引入
	import MyHead from './components/MyHead.vue'
	import Upload from './components/Upload.vue'
	import InputForm from './components/Input.vue'
	import PriceTable from './components/PriceTable.vue'
	import MyFoot from './components/MyFoot.vue'
	// 主组件 信息及逻辑函数
	export default ({
		name: 'App',
		data() {
			return {
				txt: '',
				start: '',
				lineSite: null, //根据线路找站点
				siteMap: null, //根据站点找路线
				indexSite: [], //根据矩阵索引找站点
				siteIndex: null, //根据站点找矩阵索引
				dist: [], //最短路径矩阵
				cost: null, //根据站点找票价
			}
		},
		components: {
			Upload,
			InputForm,
			PriceTable,
			MyHead,
			MyFoot
		},
		methods: {

			/**
			 * 根据路程计算票价
			 * @param {Number} distance
			 */
			countCost(distance) {
				let d = distance
				if (d >= 0 && d < 6) return 3;
				else if (d < 12) return 4;
				else if (d < 22) return 5;
				else if (d < 32) return 6;
				else if (d < 52) return 7;
				else if (d < 72) return 8;
				else if (d < 92) return 9;
			},

			/**
			 * 将txt录入邻接矩阵subway[]
			 * @param {Object} data
			 */
			handleTxt(data) {

				const subwayRaw = data.split('\n') // subwayRaw存储以换行分割的txt数组	
				const linesNum = Number.parseInt(subwayRaw[0]) // 记录总线路数量

				console.log("subwayRaw", subwayRaw, linesNum)

				var subway = [] //txt
				var transferMap = new Map() //换乘站点
				var transferNum = 0 //换乘站数目
				var siteMap = new Map() //站点和线路的对应关系
				var lineSite = new Map() //线路与站点的对应关系
				var lineIndex = new Map() //线路名称与index的对应关系 (对应subway的index)
				var indexSite = []
				var siteIndex = new Map() // 站点名称与index的对应关系 (对应matrix的index)
				var matrix = [] //邻接矩阵
				var dist = [] //最短路径长度 二维矩阵

				var MAX_PATH_LENGTH = 10001
				readTxt();

				/**
				 * 读取所有地铁线路信息、站点信息、换乘站信息
				 */
				let that = this
				async function readTxt() {

					console.log("1.0 开始读txt")

					const awaitLines = await getLines();
					console.log("1.1 线路读取完成", awaitLines);

					const awaitTransfer = await getTransfer();
					console.log("1.2 换乘站读取完成", awaitTransfer);

					const awaitSiteMap = await siteToLine();
					console.log("2.0 站点和路线对应关系 map构建完成", awaitSiteMap);

					const awaitLineSite = await lineToSite();
					console.log("2.1 路线和站点对应关系 map构建完成", awaitLineSite);

					const awaitSiteIndex = await siteToIndex();
					console.log("2.2 站点与索引值对应关系 array构建完成", awaitSiteIndex);

					const awaitMatrix = await createMatrix();
					console.log("3.0 站点矩阵初始化完成", awaitMatrix);

					const awaitWeight = await setWeight();
					console.log("3.1 第一次赋权完成", awaitWeight);

					const awaitDijkstra = await allDijkstra();
					console.log("4.0 Dijkstra查询", awaitDijkstra);

					const awaitSetData = await setData();
					console.log("5.0 给全局变量赋值", awaitSetData);

				}

				async function setData() {
					// that.$data.txt = data 	
					that.$data.siteMap = siteMap
					that.$data.indexSite = indexSite
					that.$data.siteIndex = siteIndex
					that.$data.dist = dist
					that.$data.lineSite = lineSite
				}

				async function getLines() {

					// 将每条线路信息记录再在subway中
					for (let i = 1; i <= linesNum; i++) {
						let item = subwayRaw[i].split(',')
						let line = [];
						line.push(item[0]); //线路id
						line.push(item[1]); //线路名称
						line.push(item[2]); //线路总站数
						lineIndex.set(item[1], i - 1); //线路名称index对应表

						var site = [];
						let siteNumOfALine = line[2] * 2 - 1
						for (let j = 0; j < siteNumOfALine; j++) {
							site.push(item[j + 3])
						}
						line.push(site);
						subway.push(line);
					}
					return subway;
				}

				async function getTransfer() {

					// 记录换乘站
					transferNum = subwayRaw.length > linesNum + 1 ? Number.parseInt(subwayRaw[linesNum + 1]) : 0; //检测是否有换乘站

					for (let i = 0; i < transferNum; i++) {
						let item = subwayRaw[linesNum + 2 + i].split(',')
						let tmpSites = item.slice(2)
						transferMap.set(item[0], tmpSites); //记录换乘站名称,对应的线路名称
					}
					return transferMap;

				}


				async function siteToLine() {

					// 利用map记录站点和线路的关系
					for (let i = 0; i < subway.length; i++) {
						let line = subway[i][1]; //线路名称
						let item = subway[i][3];
						for (let j = 0; j < item.length; j = j + 2) {
							siteMap.set(item[j], line)	//出现两次的环形线路也会被只记录一次
						}
					}

					// 有换乘站的情况
					for (let i = 0; i < transferNum; i++) {
						let index = i + linesNum + 2

						let item = subwayRaw[index]
						item = item.split(',')

						let transferLines = new Array()
						// Object.assign(transferLines,item)	//ES6 Object.assign浅复制
						transferLines = [...item] //ES6 另一种浅复制的另一种方法
						transferLines.shift()
						transferLines.shift() //两次shift去掉数组开头的换乘站名称和可换乘数

						siteMap.set(item[0], transferLines)
					}

					return siteMap;
				}

				async function lineToSite() {

					for (let i = 0; i < subway.length; i++) {
						let item = subway[i]
						let lineName = item[1]
						let sites = item[3]

						let sitesNew = new Array()
						for (let i = 0; i < sites.length; i = i + 2) {
							sitesNew.push(sites[i])
						}

						lineSite.set(lineName, sitesNew)
					}

					return lineSite
				}

				async function siteToIndex() {

					// 建立站点与索引值的对应关系
					let i = 0
					siteMap.forEach((value, key) => {
						siteIndex.set(key, i);
						indexSite[i] = key;
						i++;
					})

					return siteIndex;

				}

				async function createMatrix() {

					// 邻接矩阵初始化 创建n*n的二维数组
					for (let i = 0; i < siteMap.size; i++) {
						matrix[i] = new Array(siteMap.size)
						matrix[i].fill(MAX_PATH_LENGTH)
						matrix[i][i] = 0 //到达自身为0
					}
					return matrix;
				}

				async function setWeight() {

					// 矩阵赋权			
					for (let i = 0; i < subway.length; i++) {
						let item = subway[i][3]
						for (let j = 0; j < item.length - 2; j = j + 2) { //j<siteNum-1 是因为最后一个节点没有后驱 不用记录 本次循环就从第一个到倒数第二个即可						
							let siteCur = item[j] //当前站点名字
							let siteNext = item[j + 2] // 下一站点名字
							let idxCur = siteIndex.get(siteCur) //根据站名找邻接矩阵里对应index
							let idxNext = siteIndex.get(siteNext)
							matrix[idxCur][idxNext] = parseInt(item[j + 1])
							matrix[idxNext][idxCur] = parseInt(item[j + 1])
						}
					}
					return matrix;
				}


				/**
				 * Dijkstra算法求最短路径
				 * @param {Object} v	当前传入的顶点的index
				 * @param {Object} G	邻接矩阵
				 */
				function Dijkstra(v) {

					var s = new Array(); // 判断是否已存入该点到S集合中
					var prev = new Array(); //记录当前结点的前一个结点
					var distcur = new Array(); //记录到各点的最短路径

					let G = matrix
					let n = siteMap.size

					for (let i = 0; i < n; i++) {
						distcur[i] = G[v][i];
						s[i] = false;
						if (distcur[i] == MAX_PATH_LENGTH) {
							prev[i] = 0;
						} else {
							prev[i] = v;
						}
					}
					distcur[v] = 0;
					s[v] = true;

					// 依次将未放入S集合的结点中，取distcur[]最小值的结点，放入结合S中
					// 一旦S包含了所有V中顶点，distcur就记录了从源点到所有其他顶点之间的最短路径长度
					for (let i = 1; i < n; i++) {
						var tmp = MAX_PATH_LENGTH;
						var u = v;
						// 找出当前未使用的点j的distcur[j]最小值
						for (let j = 0; j < n; j++) {
							if ((!s[j]) && (parseInt(distcur[j]) < tmp)) {
								u = j; // u保存当前邻接点中距离最小的点的号码
								tmp = distcur[j];
							}
						}

						s[u] = true; // 表示u点已存入S集合中
						// 更新distcur
						for (let j = 0; j < n; j++) {
							if ((!s[j]) && (parseInt(G[u][j]) < MAX_PATH_LENGTH)) {
								var newdistcur = parseInt(distcur[u]) + parseInt(G[u][j]);
								if (newdistcur < distcur[j]) {
									distcur[j] = newdistcur;
									prev[j] = u;
								}
							}
						}
					}
					return distcur;
				}

				/**
				 * 列出所有结点抵达其他站点的最短路径
				 */
				async function allDijkstra() {
					for (let i = 0; i < siteMap.size; i++) {
						dist[i] = Dijkstra(i)
					}
					return dist;
				}

			},

			/**
			 * 根据输入框获取出发站点和目标站点
			 */
			handleInput(data) {

				let dist = this.$data.dist
				let siteIndex = this.$data.siteIndex

				let start = data.start
				let startIndex = siteIndex.get(start)

				let that = this

				check()

				/**
				 * 校验：先检测文件是否已上传；再检查起始点是否输入 以及是否正确；最后检查目标站点是否输入 若输入是否正确
				 * 		最后一个检查中，判断是查所有站点 还是两个站点的位置
				 */
				async function check() {
					const awaitCheckTxt = await checkTxt()
					if (awaitCheckTxt == false) return

					const awaitcheckStart = await checkStart()
					if (awaitcheckStart == false) return

					that.$data.start = start
				}

				async function checkTxt() {
					if (dist.length == 0) {
						alert("嘿 先传下文件！")
						return false
					}
					return true
				}

				async function checkStart() {
					if (start == '') {
						alert("嘿 您猜怎么着 这起始站您咋没输入呢！")
						return false
					}
					if (startIndex == undefined) {
						alert("起始站不存在 请重新输入")
						return false
					}

					await that.getAllPath(start)
					return true
				}
			},

			/**
			 * 查询所有路径
			 */
			getAllPath(start) {

				let cost = new Map()
				let indexSite = this.$data.indexSite

				let that = this

				beginToGet()

				async function beginToGet() {
					const awaitAllPath = await allPath();
					console.log("读取所有站点费用已完成", awaitAllPath)
				}

				async function allPath() {
					for (let i = 0; i < indexSite.length; i++) {
						cost.set(indexSite[i], that.getAPath(start, indexSite[i]))
					}
					return cost
				}

				this.$data.cost = cost
				return cost
			},

			/**
			 * 查询特定路径
			 */
			getAPath(start, end) {

				let dist = this.$data.dist
				let siteIndex = this.$data.siteIndex
				let startIndex = siteIndex.get(start)
				let endIndex = siteIndex.get(end)

				let distance = dist[startIndex][endIndex]
				let cost = this.countCost(distance)
				
				return cost
			}
		}
	})
</script>

<style>
	#app {
		font-family: Avenir, Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #45617e;
	}
	
	#container {
		margin: 5%;
		padding: 60px;
		position: relative;
		height: 100%;
		border: transparent 1px solid;
		border-radius: 3rem;
	}
	
	.inputForm {
		display: block !important;
		padding-top: 5rem !important;
	}
	
	.priceTable {
		text-align: center;
		display: block !important;
		padding-top: 5rem !important;
	}
</style>

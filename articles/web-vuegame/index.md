---
title : "生命游戏-Vue版"
date: "2020-04-15"
categories:
  - 前端

---

普林斯顿大学数学系教授 约翰•康威 近日因新冠病毒去世。其因发明生命游戏被孰知。

生命游戏有趣的是，有的会周期性震荡，有的会像宇宙飞船一样到处飞，有的像机关枪一样发射。有的呈现出某种循环，有的演化了很久都没有发现什么规律，有的还能在演化了几千万步后，消灭掉母版，复制出一个子版来。

生命游戏规则：每个格子看成一个生命体，有死（白色）和生（黑色）两种状态。每个生命体周围有8个生命体，若其中三个为生，当前则生；若其中两个为生，当前状态不变；其余情况则死。

<!--more-->

以下代码使用vue实现，预定义了三个有趣的初始形状，也可以点击格子自定义初始生命体。

```html
<html lang="en">
<head>
	<meta charset="utf-8" >
	<title>生命游戏</title>
	<style type="text/css">
		#app {
			display: flex;
			margin: 100px auto;
		}
		#sideBar {
			flex: 3;
			text-align: center;
		}
		#sideBar div{
			padding: 10px;
		}
		#container {
			flex: 7;
		}
		.row:last-child .column {
			border-bottom: 1px solid #bbb;
		}
		.column {
			display: inline-block;
			width: 15px;
			height: 15px;
			border-left: 1px solid #bbb;
			border-top: 1px solid #bbb;
			background: #fff;
		}
		.column:last-child {
			border-right: 1px solid #bbb;
		}
		.live {
			background: #000;
		}
	</style>
</head>
<body>

	<div id="app">

		<div id="sideBar">
			<div>
				<button @click="start()">开始游戏</button>
			</div>
			<div v-for="graphic in initGraphics">
				<button @click="initLife(graphic.coordinate)" v-text="graphic.name"></button>
			</div>
		</div>

		<div id="container">
			<div v-for="r in size" class="row">
				<span v-for="c in size" @click="giveLife(r - 1, c - 1)" :class="{column : true, live: cells[r - 1][c - 1]}" ></div>
			</div>
		</div>

	</div>

</body>

<!-- 引入vue -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

<script type="text/javascript">
	// 变化速度
	var speed = 100;
	// 规格
	var size = 35;

	// 初始化细胞
	function initCells() {
		var cells = {};
		for (var i = 0; i < size; i++) {
			cells[i] = {};
			for (var j = 0; j < size; j++) {
				cells[i][j] = false;
			}
		}
		return cells;
	}

	var vue = new Vue({
		el: '#app',
		data: {
			size : size,
			cells : initCells(),
			changes : [],
			initGraphics: [],
			timer : null,
			scope: {
				x : {min: size, max: 0},
				y : {min: size, max: 0},
			},
		},
		methods: {
			// 点击格子赋予生命
			giveLife(x, y) {
				this.cells[x][y] = !this.cells[x][y];
				this.updateScope(x, y);
			},
			// 更新观察范围
			updateScope(x, y) {
				if (this.scope.x.min > x - 1 && x - 1 >= 0) {
					this.scope.x.min = x - 1;
				}
				if (this.scope.x.max < x + 1 && x + 1 < size) {
					this.scope.x.max = x + 1;
				}
				if (this.scope.y.min > y - 1 && y - 1 >= 0) {
					this.scope.y.min = y - 1;
				}
				if (this.scope.y.max < y + 1 && y + 1 < size) {
					this.scope.y.max = y + 1;
				}
			},
			// 开始游戏
			start() {
				this.timer = setInterval( _ => {
					this.compute();
					this.transform();
				}, speed);
			},
			// 查找须变幻生命
			compute() {
				// let f = 0;
				for (let i = this.scope.x.min; i <= this.scope.x.max; i++) {
					for (let j = this.scope.y.min; j <= this.scope.y.max; j++) {
						// 存活邻居数
						let count = 0;
						// 判断邻居存活状态
						this.getNeighbors(i, j).forEach(cell => {
							let [x, y] = cell;
							if(this.cells[x] !== undefined && this.cells[x][y] !== undefined && this.cells[x][y] === true) {
								count += 1;
							}
							// f += 1;
						});
						// 死变生
						if(count === 3 && this.cells[i][j] === false) {
							this.changes.push([i, j]);
						}
						// 生变死
						if (count !== 3 && count !== 2 && this.cells[i][j] === true) {
							this.changes.push([i, j]);
						}
					}
				}
				// console.log(f);
				if (this.changes.length === 0) {
					clearInterval(this.timer)
				}
			},
			// 变幻
			transform() {
				this.scope = {
					x : {min: size, max: 0},
					y : {min: size, max: 0},
				};
				this.changes.forEach(cell => {
					let [x, y] = cell;
					this.cells[x][y] = !this.cells[x][y];
					this.updateScope(x, y);
				});
				this.changes = [];
			},
			// 获取8个邻居坐标
			getNeighbors(x, y) {
				return [
					[x - 1, y - 1],
					[x - 1, y],
					[x - 1, y + 1],
					[x, y - 1],
					[x, y + 1],
					[x + 1, y - 1],
					[x + 1, y],
					[x + 1, y + 1]
				];
			},
			// 预定义图形
			initLife(graphic) {
				this.cells = initCells();
				this.changes = graphic;
				this.transform();
			},
		},
	});

	vue.initGraphics = [
		{
			name : '滑翔者',
			coordinate : [[2,1],[3,2],[1,3],[2,3],[3,3]],
		},
		{
			name : '振荡子 Queen bee shuttle',
			coordinate: [[4,1],[5,1],[4,2],[5,2],[4,6],[3,7],[5,7],[2,8],[6,8],[3,9],[4,9],[5,9],[1,10],[2,10],[6,10],[7,10],[4,21],[5,21],[4,22],[5,22]]
		},
		{
			name : '振荡子 Twin bees shuttle',
			coordinate: [[3, 1], [3, 2], [4, 1], [4, 2], [10, 1], [10, 2],[11, 1], [11,2], [2, 18], [2, 19], [3, 18], [3, 20], [4, 20], [5, 20],
				[5, 18], [5, 19], [9, 18], [9,19], [9,20],[10, 20], [11, 20], [11, 18], [12, 18],[12, 19], [3, 28], [3, 29], [4, 28], [4, 29] ]
		},
	];

</script>

</html>
```

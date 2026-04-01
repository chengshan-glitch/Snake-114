Snake Game 规格书 - 学习指南
1. 资料结构
1.1 蛇（Snake）

类型：对象数组
结构：每个蛇身体段是 { x: 数值, y: 数值 } 对象
初始状态：[{ x: 2, y: 0 }, { x: 1, y: 0 }, { x: 0, y: 0 }] (3段身体)
蛇头：数组第一个元素 snake[0]
1.2 方向（Direction）

类型：对象 { x: 数值, y: 数值 }
用途：表示蛇移动的方向和速度
四个方向：
右：{ x: 1, y: 0 }
左：{ x: -1, y: 0 }
上：{ x: 0, y: -1 }
下：{ x: 0, y: 1 }
两个变量：
direction：当前实际方向
nextDirection：下一个方向（预输入）
1.3 食物（Food）

类型：对象 { x: 数值, y: 数值 }
生成：随机坐标在 0 到 29 之间
1.4 其他游戏状态

score：得分（数值，初始 0）
speed：游戏速度（毫秒，初始 200）
isRunning：游戏是否运行中（布尔值）
gameInterval：游戏循环的定时器 ID
2. 地图绘制
2.1 网格初始化

配置项	值	说明
GRID_SIZE	30	地图是 30×30 的网格
总格子数	900	30 × 30
显示尺寸	600px × 600px	CSS 中定义
每格尺寸	20px × 20px	600 ÷ 30 = 20
2.2 绘制过程

initBoard() - 创建 DOM 网格

遍历所有 900 个格子
每个格子是 <div class="cell"> 元素
设置 data-x 和 data-y 属性用于定位
clearBoard() - 清空所有样式

移除所有格子的 snake 和 food 类名
draw() - 渲染蛇和食物

调用 drawSnake() - 给蛇身体的格子加上 snake 类
调用 drawFood() - 给食物格子加上 food 类
2.3 样式定义

元素	样式	说明
.snake	背景白色	background: var(--primary)
.food	背景红色	background: var(--accent)
3. 方向键移动
3.1 输入处理（handleKey）

3.2 防御机制

防止蛇立即反向移动（例如向右移动时按左键无效）：

判断条件：newDir.x + direction.x !== 0 || newDir.y + direction.y !== 0
作用：两个相反方向的 x 或 y 相加为 0 时被拒绝
3.3 执行移动（moveSnake）

4. 产生食物
4.1 randomFood() 函数

4.2 生成时机

时机	位置	说明
游戏开始	startGame()	第一个食物位置
蛇吃到食物	handleFood()	生成下一个食物
4.3 特殊情况

食物可以生成在蛇身体上（不做额外检查）
只有蛇头碰到食物才算吃到
5. 贪食蛇吃到食物会变长
5.1 变长机制

步骤	操作	代码位置
1	检测碰撞	if (head.x === food.x && head.y === food.y)
2	增加得分	score++
3	更新显示	更新 DOM 中的 score 文本
4	生成新食物	food = randomFood()
5	加快速度	speed = Math.max(10, speed - 2)
6	头部插入	snake.unshift(newHead)
7	不移除尾部	跳过 snake.pop()，导致身体加长
5.2 对比：未吃到食物的情况

5.3 难度递增

每吃到一个食物，游戏速度加快：

初始速度：200 ms
每次递减：2 ms
最小速度：10 ms（下限保护）
6. 贪食蛇吃到自己会死
6.1 碰撞检测（isCollision）

检查新蛇头位置是否与蛇身体任何一段重叠
返回 true 表示发生碰撞
6.2 死亡触发

步骤	操作	位置
1	计算新蛇头	moveSnake()
2	检查碰撞	isCollision(newHead)
3	如果碰撞	clearInterval(gameInterval)
4	停止游戏	isRunning = false
5	弹出提示	alert('Game Over')
6.3 触发条件

自己咬自己：新蛇头坐标与身体任何部分重合
注意：蛇头与蛇头的前一段不会立即碰撞（因为 update 时先 unshift 新头）
7. 重新开始按钮
7.1 startGame() 函数职责

初始化项	初始值	说明
snake	3段身体	[{x:2,y:0}, {x:1,y:0}, {x:0,y:0}]
direction	向右	{ x: 1, y: 0 }
nextDirection	向右	复制 direction
food	随机位置	randomFood()
score	0	重置得分
speed	200 ms	重置速度
isRunning	true	游戏开始
游戏板	重新绘制	initBoard()
7.2 执行流程

8. 暂停游戏
8.1 togglePause() 函数

8.2 状态切换

当前状态	按钮点击	新状态	说明
运行中	Pause	暂停	clearInterval，gameInterval = null
暂停中	Pause	运行	重启定时器
游戏结束	Pause	无效	isRunning = false 时不响应
8.3 关键点

暂停时 不改变 任何游戏状态（蛇位置、食物、得分等）
通过检查 gameInterval 是否为 null 来判断暂停/运行状态
恢复时使用相同的 speed 重启循环
9. 主游戏循环
gameLoop() → update() → draw()
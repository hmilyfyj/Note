title: 算法复习-背包问题
date: 2016-03-04 08:45
tags: [算法,基础不丢,在成为最厉害最厉害最厉害的路上]
categories: 算法
---

经典的动态规划模型。

<!-- more -->

---

# 描述

> 给定一组物品，每种物品都有自己的重量和价格，在**限定的总重量**内，我们如何选择，才能使得物品的总价格最高。

# 解决思想

设有n个物品，背包的重量为`w`，`C[i][w]`为最优解。`v[i]`为第i个商品的价值，`w[i]` 为其重量。

首先我们考虑一个问题：我们从第一件商品开始向背包里放置商品，通过不停的拿取 、组合从而获得了最大价值`C[i][w]`时，最后一个商品`(i=n`)只有两种状态，要么放、要么不放（废话）：

| 商品i的状态	 | C[i][w]     |
|:--------:| -------------:|
| 放时：| w[i] +  C[i - 1][w - w[i]] |
| 不放：| C[i - 1][w] |

这时，目标就变成了求  前i-1 个商品 的最优解。这样的话我们就需要把不同容量w(w<最大容量)和不同商品数数量i(i<最大商品数数量)全部推导出来。

![](http://7xnocp.com1.z0.glb.clouddn.com/16-3-4/78190806.jpg)


# PHP 实现

```php
/**
	 * 测试
	 *
	 * @author Feng <fengit@shanjing-inc.com>
	 */
	function test() {
		//导入类库
		
		//获取参数
		
		//校验参数
		
		$v = array(10, 40, 30, 50); //商品价值
		$w = array(5, 4, 6, 3); //商品重量
		$W = 10; //背包容量
	
		Knaspack($v, $w, $W);
	}
	
	/**
	 * 背包问题
	 *
	 * @author Feng <fengit@shanjing-inc.com>
	 */
	function Knaspack($v, $w, $W) {
		//导入类库
		
		//获取参数
		
		//校验参数
		
		//初始化
		$l	= count($v); //商品数量
		$V	= array();	//二维数组
		
		//第0列、0行均为0
		for($i = 0; $i <= $l; $i++)  $V[$i][0]	= 0;
		for($i = 0; $i <= $W; $i++)  $V[0][$i]	= 0;
		

		for($i = 1; $i <= $l; $i++) {
			for($weight = 1; $weight <= $W; $weight++) {
				//元素小于背包容量
				if ($w[$i - 1] < $weight) {
					//尝试放入背包
					$V[$i][$weight]		= max($w[$i - 1] + $V[$i - 1][$weight - $w[$i - 1]], $V[$i -1][$weight]);
				} else {
					//抛弃，保持 $i - 1的状态
					$V[$i][$weight]		= $V[$i - 1][$weight];
				}
			}
		}
		
		//输出选择方案（商品编号）
		$j			= $W;
		$selected	= array();
		for($i = $l; $i > 0; $i--) {
			if ($V[$i][$j] > $V[$i - 1][$j]) {
				$selected[]	= $i;
				$j			= $j - $w[$i - 1];
			}
		}
		
		//输出最大价值
		print_r($V[$l][$W]);
	}
```

# 参考地址

http://www.cnblogs.com/Anker/archive/2013/05/04/3059070.html

http://www.hawstein.com/posts/dp-knapsack.html

http://blog.csdn.net/zs234/article/details/7487960

https://www.javacodegeeks.com/2014/07/the-knapsack-problem.html

http://www.importnew.com/13072.html


# 八皇后
每一行、列、对角线的元素个数必须小于2。

# PHP 实现

```php
$solution_count = 0;
$arr			= array();

search(0);
echo $solution_count;

function search($index) {
	global $solution_count;
	global $arr;
	for ($i = 0; $i < 8; $i++) {
		$arr[$index] = $i; 
		
		//检查方案是否可行
		if (is_safe($arr, $index)) {
			//
			if ($index == 7 ) {
				//进行到最后一步，输出结果
				$solution_count++;
				echo implode(',', $arr)."\n";
			} else {
				//继续进行搜索
				search($index+1);
			}
		} else {
			//该方案不可行尝试其他方案
		}
	}
}


function is_safe(&$arr, $index) {
	//
	for ($i = 0; $i < $index; $i++) {
		if (abs($index - $i) == abs($arr[$index] - $arr[$i]) || $arr[$i] == $arr[$index]) {
			return FALSE;
		}
	}
	
	return TRUE;
}
```


http://blog.csdn.net/zssureqh/article/details/21116883

https://zh.wikipedia.org/wiki/%E5%85%AB%E7%9A%87%E5%90%8E%E9%97%AE%E9%A2%98


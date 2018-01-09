# Product Recongnization
####目的：

```
	秋千 儿童 室内 婴儿幼儿童室内荡秋千户外 宝宝玩具
	儿童六一礼物儿童室内秋千户外吊椅室外荡秋千家用运动健身器材
	幼儿园儿童组合滑梯淘气堡室内滑梯宝宝游乐园滑滑梯秋千玩具包邮
	加厚滑滑梯儿童室内幼儿园婴儿玩具家用滑梯秋千组合海洋球球池
	滑梯批发 幼儿园儿童滑梯 室外最畅销塑料滑梯 秋千组合滑滑梯
	游乐设施幼儿园玩具户外大型小区儿童组合工程塑料室外幼儿园滑梯
```
以上字段是從淘寶網上爬取的標題，可以看出商家習慣性的把所有關鍵字塞進去，且沒明顯的分界。
目標是要將上面的句子分詞，並找出哪些是產品名。

####困難處：
* 特定領域的分詞任務，general 分詞器（Jieba hanglp) 可能效用不佳
* 產品名歧異大，人都很難說準什麼是產品
	* ex: 儿童组合滑梯淘气堡？ 淘气堡？ 组合滑梯？ 组合滑梯淘气堡？


####流程分成三大步驟：

1. Pattern Discovery： 分詞和新詞發現 
2. Human Correction: 發現的新詞和分詞，要經由人工校正並找出**詞意**是產品的詞彙
3. 神經網路學習 Product name 的組成

####Furtue work:

1. 第三步驟中，神經網路訓練可以疊代
2. 第一步驟中，加入字典的幫助，減少人工量




## Pattern Discovery
####目標：

* 冷啟動-無監督新詞發現
* 發現的詞用於之後深度學習
* 盡量減少第二步驟時的人工量

### 計算詞彙的 Entropy
把所有的詞彙組合找出來

####Run code
	python3 find_pattern/find_frequency_pattern_by_entropy.py
	
* **find\_frequency\_pattern\_by\_entropy.py** 的 input 是一個未分詞的文件
	* 預設在 **usr/find\_frequency\_pattern** 下的 **source.txt**

* output 有三, 預設放在 **usr/find\_frequency\_pattern/** 下。
	1. **word\_mutual\_entropy.txt**：記錄了 word 之間的 mutual entropy 格式如下 
		* 格式`詞彙	mutual entropy`,中間以 \t 隔開
		* EX: `卡通	0.09291187304856806`
	2. **word\_entropy.txt** 記錄了 word 左右兩側的自由度。
		* 格式：`詞彙	左側自由度	右側自由度` 中間以 \t 隔開
		* ex：`遥控特技车	0.6365141682948128	0.6365141682948128`

	3. **detail\_information.txt** 包含了詞彙的mutual entropy 以及跟左右兩側的自由度，還加入了詞彙出現在語料中的機率。
		*	格式：同時完整記錄每個詞彙的內部組成，中間以 \t 分開。

```
詞彙	出現機率	互消息	左entropy	右entropy
	左側組成 右側組成	左機率	右機率 左右機率相乘	組成間的互消息
	
	
毛绒玩具礼品	0.000109	0.001067	1.831020	1.831020
	毛	绒玩具礼品	0.028833	0.000109	0.000003	0.000387
	毛绒	玩具礼品	0.025530	0.000864	0.000022	0.000174
	毛绒玩	具礼品	0.014916	0.000942	0.000014	0.000223
	毛绒玩具	礼品	0.014650	0.012643	0.000185	-0.000058
	毛绒玩具礼	品	0.000122	0.039404	0.000005	0.000340
```
最主要會用到 **detail\_information.txt** 其他兩個皆是參考, 決定hyper parameter用

### 分析 **detail\_information.txt**

此步驟分析 **detail\_information.txt** 輸出*可能有意義*的詞彙給接下來的人工審核。
有意義的詞包含了 *產品名* 和 *一般詞彙*。

	以下組合皆是有意義 ex: 毛绒玩具, 毛绒, 玩具

這是個比單純分詞還稍微困難點的問題，所有粒度都得概括，且每個粒度都必須有意義。

	室内儿童淘气堡: 室内儿童淘气堡, 儿童淘气堡, 淘气堡
以下是沒意義：
	
	室内儿童淘气堡: 内儿童淘气堡, 童淘气堡, 气堡

執行檔 **alg.py** 實現了一套框架以及演算法針對 **detail\_information.txt** 挖掘

####Run code

	python3 find_pattern/alg.py

* Input 即是上述 **detail\_information.txt**
* Ouput 有2，預設寫入 **usr/find\_frequency\_pattern** 之下
	1. **all_alg.txt**: 記錄了對每個字段**'可能形成有意義詞彙'**的評分
		* 格式： `字段	'accept score'	'reject score'`, 中間以 \t 分開
		* ex: 
	
	```
	音乐钓鱼	1	0
	音乐闪光	16	0
	音乐陀螺	1	0
	音响	10	0
	音箱	22	0
	项目	60	0
	```

	2. **filtered_alg.txt**:

####Alogrithmn


	
# 棒球数据可视化

## 概要

>  该数据集是一个包含1,157名棒球手的数据集，包括他们的用手习惯（左手还是右手）、身高（英寸）、体重（磅）、击球率和全垒得分。通过该可视化过程，展示这些球手的表现差异。



## 设计

- 本次可视化的数据样本来源于 https://s3.amazonaws.com/udacity-hosted-downloads/ud507/baseball_data.csv，共包含了 891 条数据

- 数据具有5个属性，分别为：
	- name: 棒球队员名称
	- handedness： 惯用手
	- height： 身高
	- weight： 体重
	- avg： 击球率
	- HR： 全垒得分

- 本次的可视化分析图表采用 D3 + Dimple 绘制，鼠标移动到图表上时可以自动显示相关信息。所以并没有额外添加标签来显示相关信息。

### 棒球选手的惯用手情况情况

首先，我先对样本数据进行聚合处理：

```

		var nests = d3.nest()
            .key(function(d) { return d.handedness; })
            .rollup(function(v) { return v.length; })
            .entries(data);

```

根据棒球选手的习惯用手，分成三组数据，然后使用 Dimple 画柱状图：

```

	  	var myChart = new dimple.chart(svg, nests);
	    var x = myChart.addCategoryAxis("x", "key")
	    var y = myChart.addMeasureAxis("y", "values");
	    var s = myChart.addSeries(null, dimple.plot.bar);
	    x.title = "handedness";
	    y.title = "number of people";
	    myChart.draw();

```


柱状图可以很直观的看出来选手惯用手的情况和比例



### 棒球选手惯用手与平均全垒得分的关系

我们将棒球选手根据惯用手来进行分类得到的三组数据的同时进行统计其全垒得分的总数，将这个总数除以当前情况下该惯用手的人数，记得到该惯用手的全部选手的平均全垒得分

```

  	    var nests_HR = d3.nest()
            .key(function(d) { return d.handedness; })
            .rollup(function(v) { 
                var total_HR = d3.sum(v, function(t){
                  return t['HR']
                })
                var total_num = v.length
                
                return total_HR / total_num

              })
            .entries(data);

```

再将其用柱状图展示出来得到可视化图像


```

	  	var myChart = new dimple.chart(svg, nests_HR);
	    var x = myChart.addCategoryAxis("x", "key")
	    var y = myChart.addMeasureAxis("y", "values");
	    var s = myChart.addSeries(null, dimple.plot.bar);
	    x.title = "handedness";
	    y.title = "number of people";
	    myChart.draw();

```

### 棒球选手惯用手与平均击球率的关系

同理我们也将平均击球率进行了跟平均全垒得分一样的可视化。


### 小结
通过上面得到的三个可视化图像，可以看到右手为惯用手的球员在人数、击球率和全垒得分上具有更大的优势


### 更多可视化
此外，我们还对身高和体重对击球率和全垒得分的影响通过散点图进行表示，并将每一个球员作为一个数据点，绘制在可视化图中。 此过程采用了更加多的交互过程实现。包括通过添加按钮来使读者可以自行选择变量进行可视化，并且读者可以通过将鼠标悬停在数据点上以获取更多的数据信息。

## 反馈

我向三位好友分享了我的可视化，并收到了一些反馈：

- 看到图之后不太清楚是什么意思。

  - 改进：补充标题和注释。

- 交互过程较少

  - 添加按钮来进行让读者选择，而不是全部的静态图片	


- 第一个柱状图ok，不过第二三图使用柱状图只能表达出击球率和全垒得分的平均值，感觉信息有点少，可以考虑使用箱线图来体现数据内部的分布。

	- 把柱状图改成箱线图来表达

- 更多可视化中颜色有什么含义吗？如果没有具体的含义的话最好不使用颜色，这样会给观众带来疑问

	- 颜色本来的意思是代表每一个不同的棒球选手，现在统一改成了黑色

- 互动设计中存在bug，按一次按键就会多产生一个图形

	- 在修改成箱线图后忘记查看之前按钮的功能导致选择器选择元素产生错误


### 箱线图的绘制

首先我们建立一个箱线图函数：该函数的传入参数为数据源， 需要绘制成箱线图的属性（击球率或全垒得分）， 绘制图像的div元素id 以及绘制的图像的标题
"""

function boxplot(data, attr, div_id, title_name){
              var y0=[],y1=[],y2=[]
              for (var i = 0; i < data.length; i ++) 
              {
                  if(data[i].handedness == "L"){
                    if (attr == 'HR')
                      y0[i] = data[i].HR;

                    else
                      y0[i] = data[i].avg;
                  }
                  else if (data[i].handedness == "R"){
                    if (attr == 'HR')
                      y1[i] = data[i].HR;

                    else
                      y1[i] = data[i].avg;
                  }
                  else{
                    if (attr == 'HR')
                      y2[i] = data[i].HR;

                    else
                      y2[i] = data[i].avg;
                  }
              }

              var Left_hand = {
                y: y0,
                marker: {color: '#8FBBDA'},
                type: 'box',
                name: 'Left_hand'
              };

              var right_hand = {
                y: y1,
                marker: {color: '#8FBBDA'},
                type: 'box',
                name: 'right_hand'
              };

              var both_hand = {
                y: y2,
                marker: {color: '#8FBBDA'},
                type: 'box',
                name: 'both_hand'
              };

              var layout = {
                title: title_name
              };

              var tmp = [Left_hand, right_hand, both_hand];

              Plotly.newPlot(div_id, tmp, layout);
          }

"""

然后通过不同的调用来使得输出不同的图像

"""

	boxplot(data, 'HR', 'boxplot1', "惯用手与全垒得分的箱线图") 
	boxplot(data, 'avg', 'boxplot2', "惯用手与击球率的箱线图")

"""

## 参考资源

- Dimple：[dimplejs.org](http://dimplejs.org)
- d3js.org：[d3js.org](https://d3js.org)
- poly.py：[poly.py](https://plot.ly/javascript)
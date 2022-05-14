# vue自定义类似flomo、github活跃图(格子图、贡献图)



最终效果如下图所示：

![02.最终效果](/pic/02.最终效果.gif)

## 一、布局样式

该组件采用了`grid`网格布局，容器时自定义的卡片，具体内容上由`<ul>`和`<li>`实现，格子部分和下边月份部分是独立的`<ul>`，内容均为`<li>`。

![03.基本样式](/pic/03.基本样式.png)

html框架如下:

```html
<div class="container-card">
    <ul class="graph">
        <el-tooltip class="item" effect="dark" :content="item.year + '-' + item.month + '-' + item.date"
            placement="top-start" v-for="(item, index) in infos" :key="index" :open-delay="500">
            <li :data-level="item.level" class="li-day" :isToday="item.isToday" @click="handleClick(item)"></li>
        </el-tooltip>
    </ul>

    <ul class="months">
        <li class="li-month" v-for="(item,index) in monthBar" :key="index">{{item}}</li>
    </ul>
</div>
```



具体样式设定见如下css代码：

```css
.graph {
  display: grid;
  grid-template-columns: repeat(12, 21px);  /*竖向12列，21px宽*/
  grid-template-rows: repeat(7, 21px);     /*横向7列，21px宽*/
  padding-inline-start: 0px;
  grid-auto-flow: column;               /*生成7*12的格子后，设置为竖向排布*/
  margin: 20px 20px 5px 20px;
}

.months {
  display: grid;
  grid-template-columns: repeat(12, 21px);
  grid-template-rows: 21px;
  font-size: 8px;
  color: #aaa;
  padding-inline-start: 0px;
  margin: 5px 20px 5px 20px;
}

.li-month {
  display: inline-block;
}

.li-day {
  background-color: #eaeaea;
  list-style: none;             /*记得把list的圆点效果去掉*/
  margin: 1.5px;
  border-radius: 3px;
}

.li-day:hover {               /*添加hover强调效果*/
  box-shadow: 0px 0px 5px rgb(57, 120, 255);
}
```

## 二、功能实现

该组件的需求就是展示近12周（84天）的活跃度，每个方格代表一天，本星期永远都在右边最后一列，而且点击某个方格会有其对应那一天的点击事件（如点击某天格子请求某一天的数据页面）。具体实现如下：

首先创建的data数据有：

```javascript
data: {
    infos: [],  //存放84天每一天的数据（year，month，date，状态数量，isToday标记）
    current: {  //存放今天的年月日
        year: "",  
        month: "",
        date: "",
    },
    monthBar: ["", "", "", "", "", "", "", "", "", "", "", ""],//12列对应的月份，比如第三列开始是五月份，则令monthBar[2]="5月"，算法实现见下面method
}
```

`created()`初始化数据：

```javascript
created() {
    let d = new Date();
    let day = d.getDay();           //获取今天所在星期，以进行后续计算
    let today = d.getDate();        //获取今天的日数

    this.current.year = d.getFullYear();   //初始化今日的年月日
    this.current.month = d.getMonth();
    this.current.date = d.getDate();

    let info = {};         //用来存放某一天的数据对象（年月日、isToday、level）      
    let month = "";        //后续计算某月第一天在哪一列用，表示第几月
    let weekOfMonth = ""   //后续计算某月第一天在哪一列用，表示第几列

    /**
    *前提：Date对象通过setDate()设置到某一天的年月日，例如setDate(1)就是设置日期月本月1号
    *而setDate(0)则是上个月最后一天，setDate(-1)则是设置为上个月倒数第二天
    *以此类推，假如今天是星期五，如果周日算本周第一天，则今天是本周第6天，也就是本周在今日前面还有5天，本周在第12列，则今天前的日期有11*7+5=82天，所以第1列第1格为82天前。
    *假如今天是13日，82-13=69，那么第1列第1格为本月第1天的前69天，则那天的日期设置为setDate(-69+1)（+1是为了算上0），最后就能获得那一天的年月日对象，再获得年月日数值。
    *知道前提后下面的代码可以自己体会了
    */
    for (let i = 0; i < 84; i++) {
        d.setFullYear(this.current.year);  //每次循环要重置年月日为今天否则会以上次循环结尾的年月日计算而计算错误
        d.setMonth(this.current.month);
        d.setDate(this.current.date);

        d.setDate(today - 77 - day + i);   //设置到本次循环的date   
        //(today-(84-(7-day)+i)

        let level = Math.floor(Math.random() * 3); //这里是随机设置每天的频率等级，后续开发要替换为自己计算的真实等级（不同等级对应不同颜色方格）

        if (     //判断是否为今天，是则做些标记，后续渲染时可以突出强调今天的格子
            d.getMonth() == this.current.month &&
            d.getDate() == this.current.date
        ) {
            info = {                      //每个格子（天）的info对象
                year: d.getFullYear(),      //年月日
                month: d.getMonth() + 1,
                date: d.getDate(),
                number: 10,    //今日的数据量
                level: level,  //今日数据量对应的等级
                isToday: true, //是否是今天
            };
            this.infos.push(info);
        } else {
            info = {
                year: d.getFullYear(),
                month: d.getMonth() + 1,
                date: d.getDate(),
                number: 10,
                level: level,
                isToday: false,
            };
            this.infos.push(info);
        }
        //判断每月第一天在12列种的哪一列
        if (d.getDate() == 1) { //date为1的肯定是某月第一天
            month = d.getMonth() + 1  //获取这一天对应的月份（0-11，所以还要+1）
            weekOfMonth = parseInt((i + 1) / 7)  //这个月的第一天的index（84天的第几天）除以7获得所在列的index（12列的第几列），作为下面monthBar的index，并把原来空的内容用替换为xx月
            this.monthBar[weekOfMonth] = month + "月"
        }
    }
}
```

不同等级显示不同颜色样式，

```css
.graph li[data-level="1"] {
  background-color: #d9ecff;
}

.graph li[data-level="2"] {
  background-color: #8cc5ff;
}

.graph li[data-level="3"] {
  background-color: #409eff;
}
```

每次v-for循环中，给每个格子打上不同等级颜色：

```html
<li :data-level="item.level" class="li-day" :isToday="item.isToday" @click="handleClick(item)"></li>
```



## 三、后续功能

后续添加每个格子的点击处理，点击格子弹出对应天的消息，实际开发中替换成自己的功能

```javascript
methods: {
    handleClick: function (item) {
        console.log(item.year + "-" + item.month + "-" + item.date)
        alert(item.year + "-" + item.month + "-" + item.date)
    }
}
```

至于鼠标悬停显示信息则是采用了element-ui的tooltip组件，并作为v-for所在的组件包裹住`<li>`标签，代替`<li>`标签的for循环

![04.tooltip](/pic/04.tooltip.png)

```html
<el-tooltip class="item" effect="dark" :content="item.year + '-' + item.month + '-' + item.date" placement="top-start" v-for="(item, index) in infos" :key="index" :open-delay="500">
    <li :data-level="item.level" class="li-day" :isToday="item.isToday" @click="handleClick(item)"></li>
</el-tooltip>
```


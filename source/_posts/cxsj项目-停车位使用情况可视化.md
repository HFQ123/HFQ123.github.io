---
title: cxsj项目-停车位使用情况可视化
date: 2020-04-10 08:53:18
tags: [项目,JavaWeb]
---

## 需求分析

在智慧停车管理平台(WEB端)项目中，为了清晰地展示当前停车场的车位使用情况，想到把车位的使用状态可视化地展示到室内停车场地图上。效果如下：

![image-20200410113840534](/images/image-20200410113840534.png)

根据室内地图的车位图标颜色不同，用户可以看到停车场各个车位的使用状态。此外，当用户点击地图内的车位图标时，还能获取当前指定车位的状态（是否占用，如果占用，还会显示车牌号）。



## 实现技术

前端：Jquery (ajax请求数据)

后端：SpringBoot Mybatis (数据库查询数据)

室内地图支持：Esmap （地图展示)



## 实现方案

### 1. 引入Esmap的室内地图,熟悉其sdk

首先，从下载室内地图的[示例](https://www.esmap.cn/escopemap/content/cn/member/mapdownload.html)，以获得示例地图的数据包。我选择的是“基本地图显示”示例。

然后，在html中引入室内地图，这一步Esmap官网的[开发文档](https://www.esmap.cn/escopemap/content/cn/develope/develope.html)有详细的流程我就不多介绍了。按照指示引入地图到html后，实现的初始效果如下：

![image-20200409174547260](/images/image-20200409174547260.png)

示例默认的主题颜色比较单调，可以通过[配置参数](https://www.esmap.cn/escopemap/content/cn/develope/map-styleedit.html)来修改主题样式（如地图背景颜色等）。

接下来，熟悉Esmap的sdk，本案例中，我们主要关注的就是：如何跟踪点击事件？(用户点击后停车位能够触发操作)，修改图标的颜色（加载车位可视化页面时根据车位的使用状态为车位图标设置不同的颜色），发现这分别对应开发文档中[地图事件](https://www.esmap.cn/escopemap/content/cn/develope/map-event.html)中的“地图点击返回事件”和[常用方法](https://www.esmap.cn/escopemap/content/cn/develope/map-fun.html)中的“改变房子颜色”的changeModelColor()方法。

接下来我们就研究一下这两个方法。

#### 方法1：地图点击返回事件

``` javascript
//地图点击返回事件
//https://www.esmap.cn/escopemap/content/cn/develope/map-event.html
map.on('mapClickNode', function(event) {
    console.log(event);   //打印事件
}
```

可以发现，当点击地图的车位图标时，打印的车位对象如下图。

![image-20200410095433726](/images/image-20200410095433726.png)

因为点击地图其他图标(比如说楼梯、道路)也会触发地图点击事件，而在本案例中我们只要关注点击车位事件，所以我们可以以地图点击返回事件event中的name属性里是否包含"车位"二字来区分被点击的图标是否为车位，如果判断是车位，才会触发后续的方法changeModelColor()来改变被点击车位的颜色。

#### 方法2：改变房子颜色

```javascript
//改变房子颜色(一定要在地图加载完成事件之后才有效)
//id,name二选择一，都可以是数组, fnum可选择，参数color:'#FF0000'
//https://www.esmap.cn/escopemap/content/cn/develope/map-fun.html
map.changeModelColor({
     //id:[1,2],
     name:'车位1091',
     //fnum:[1],
     color:'#FF0000'}
) 
```

需要注意开发文档里指出的"一定要在地图加载完成事件之后才有效"。也就是说，changeModelColor()方法要在加载完成事件完成后调用，如下：

```javascript
//地图加载完成事件
//https://www.esmap.cn/escopemap/content/cn/develope/map-event.html  
map.on('loadComplete', function () {
       map.changeModelColor()方法要在此调用！！！
});

```

至此，我们就完成了地图页面展示，掌握了点击车位图标触发事件的方法和修改车位图标颜色的方法。

### 2. 创建停车位的数据表和后端api接口

在我创建的车位数据表中，有这样几个主要字段：

+ 车位标识`id`：由于在本案例我们使用的室内地图数据包中，所有车位对象的name属性在“车位1001~车位1102”，为了下一步地图车位对象和数据库记录的绑定，车位id的值也设置在此范围内。

+ 车位所在停车场标识`parkinglot_id`：由于我的项目涉及到多个停车场，需要此字段，如果只是有一个停车场可以不用此字段。 

+ 车位状态标识`in_use`：若车位是空闲状态，取值0，若占用取1。

+ 车牌号`car_id`：若车位占用，存车牌号，否则为空。

+ 停车时间`park_time`：若车位占用，存此车进入车位的时间戳，否则为0。

创建好车位数据表之后，生成一些假数据（我是通过写了一个java类，按照给定格式随机生成记录，然后拼接字符串打印这条记录的insert语句，然后到数据库执行，后来网上了解一下，或许可以尝试使用Python的伪造数据生成器Faker），生成的部分假数据如下图展示。

![image-20200410102612112](/images/image-20200410102612112.png)

   下一步，写一个Controller，完成车位查询的api接口，响应POST方法，为请求返回json数据，代码如下：

```java
/**
     *
     * @param parkinglot_id 停车场ID
     * @param parking_space_id 车位id
     * @return
     * 如果参数parking_space_id 为空，查找所有停车场ID为parkinglot_id的所有车位使用情况
     * 如果参数parking_space_id非空，根据停车场id和车位id 查询一条记录。
     */
    @ResponseBody
    @PostMapping("/parkinglot/{parkinglot_id}/space")
    private Object selectAllSpace(@PathVariable(name = "parkinglot_id") Integer parkinglot_id,
                                  @RequestParam(name = "parking_space_id",required = false) Integer parking_space_id
                                  ){
        ParkingSpaceExample example = new ParkingSpaceExample();
        if(parking_space_id!=null){
            example.createCriteria()
                    .andParkinglotIdEqualTo(parkinglot_id)
                    .andIdEqualTo(parking_space_id);
        }
        else {
            example.createCriteria()
                    .andParkinglotIdEqualTo(parkinglot_id);
        }
        List<ParkingSpace> parkingSpaces = parkingSpaceMapper.selectByExample(example);
        return parkingSpaces;
    }
```



如果请求时携带了parking_space_id参数，就只查询车位id等于parking_space_id的那条车位记录，否则，查询该停车场的全部车位记录。请求参数携带车位标识和不携带车位标识这两种请求应用场景分别是：加载页面时，需要查询全部车位的车位状态，而用户点击地图的某个车位图标时，只需要查询这一个车位状态。

当不携带车位标识时，某次请求响应的部分json数据如下图展示。

![image-20200410104637275](/images/image-20200410104637275.png)

至此，我们就完成了车位数据表的建立和查询。

### 3. 将地图显示和数据库“绑定”

这一步我们就完成可视化的任务，具体需要完成的功能有两个：功能一是加载车位详情页面的时候，查询车位数据表，根据车位状态为地图的车位图标“上色”（空位占用则红，车位空闲则为绿）；功能二是当用户点击车位图标时，查询数据库该车位的当前状态并显示出来（如果车位占用还会显示出当前的车牌号）。

有了步骤1、2的基础后，这一步就很简单了，简单地说，就是用步骤2的数据填充到步骤1的方法内。具体做法是在javascript脚本中使用ajax请求获取后端的车位数据，然后对这些数据按指定格式处理，作为调用Esmap sdk的配置参数。

以功能一的实现为例：

使用ajax向步骤2中编写好的用于查询车位的api接口发送请求，获得请求后的json数据后，遍历每条数据（对应车位数据表的一条记录），根据数据的inUse属性的取值来标识车位是占用还是空闲状态，然后拼接“车位”二字到数据的id属性的取值，就获得了这条记录对应在地图上车位图标对象的name属性（也就是说，数据库中的车位id为1001的车位记录对应地图上name="车位1001"的地图对象)。设置js数组inUseNameList和notInUseNameList，它们分别是全部**占用车位**对应的地图车位图标的name属性数组、全部**空闲车位**对应的地图车位图标的name属性数组。调用 map.changeModelColor()方法设置地图车位图标的颜色时，指定name参数为inUseNameList，颜色设置成红色，指定name参数为notInUseNameList，颜色设置成绿色。

代码如下：

```javascript
/**
 * 完成功能一：当地图加载完成时，设置停车位的颜色
 */
function loadSpace() {
    var inUseNameList = new Array();   //全部的占用车位对应的地图车位图标的name属性数组
    var notInUseNameList = new Array(); //全部的空闲车位对应的地图车位图标的name属性数组
    var pathname = window.location.pathname; 
    var parkinglot_id = pathname.charAt(pathname.length-1);
    console.log(pathname);
    $.ajax({ //使用ajax请求数据
        method: "POST",
        url: "/parkinglot/"+parkinglot_id+"/space",
    })
        .done(function( spaceList ) {
            // console.log(spaceList);              //打印全部车位信息
            $.each(spaceList,function(i,space){
                var spaceName = "车位"+space.id;
                if(space.inUse==1){                 //表示车位被使用了
                    inUseNameList.push(spaceName);
                }else {
                    notInUseNameList.push(spaceName);
                }

            });
            map.changeModelColor({    //占用车位设置成绿色
                name: inUseNameList,
                color: '#FF0000'}
            );
            map.changeModelColor({   //空闲车位设置成绿色
                name: notInUseNameList,
                color: '#00FF00'}
            );
        });
}
```




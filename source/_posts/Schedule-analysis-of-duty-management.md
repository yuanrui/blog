---
title: 勤务管理之排班分析
date: 2020-08-19 15:20:46
tags: 
  - 系统设计
  - 开发笔记
---
### 前言

排班是人力资源精细化控制的基础，一个好的排班管理不仅能调动人员的工作积极性，也能提高工作效率，提高劳动力生产力。

对于排班管理而言，需要考虑：时间、人员、地点、工作事项、工作设备和环境要求等几个要素。针对不同需求场景，包含的要素需要有所侧重，其中时间和人员是最为核心的要素，毕竟排班讲究的是合适的时间安排合适的人去做某某事情。当然还有可能包含合适的地点，但是在一些固定工作场所的前提下，地点相对时间、人员而言属性不是那么重要。

### 固定场所排班分析

假设，某交巡警支队后台视频监控部门排班管理，需要保障道路交通持续不断的安全运维，对工作人员要求按照早班、中班、夜班三个班次进行轮换上班上岗。针对这种场景，工作场所一般是固定的，每个员工的工作内容一般是固化下来的，每天只需要按部就班执行即可。那么在这里，主要包含时间、人员这两个要素。

该部门的排班表如下：

| 日期       | 班次                 | 人员                                 |
| ---------- | -------------------- | ------------------------------------ |
| 2020-08-15 | 早班（8:00～16:00）  | 令狐冲 郭靖 杨过       |
| 2020-08-15 | 中班（16:00～24:00） | 任盈盈 黄蓉 小龙女     |
| 2020-08-15 | 夜班（24:00～8:00）  | 乔峰 虚竹 段誉         |
| 2020-08-16 | 早班（8:00～16:00）  | 黄药师 一灯大师 周伯通    |
| 2020-08-16 | 中班（16:00～24:00） | 令狐冲 郭靖 杨过       |
| 2020-08-16 | 夜班（24:00～8:00）  | 任盈盈 黄蓉 小龙女     |
| ...    |                      |                                      |

从这个排班表中，我们可以抽象出：日期、班次(时段)、人员这三个属性。一般班次属性可单独抽象出来，班次包含：班次名称、开始时间、结束时间和作用范围。作用范围可以为工作日、周末、节假日等选项。

![](_images/勤务模板-班次设置.png)

结合到排列组合来讲，我们可以将人员进行按班次进行分组管理，每个分组对应一个班次即班组。有了班次和人员分组之后，我们可以设置班次的起始人员分组，然后按照班次顺序依次和人员分组进行循环对应。这也是自动排班管理的一种简单实现方式。

### 变化场所排班分析

#### 举例分析

通过上述分析，排班管理看上去做起来好像很容易？然而并没有那么简单，这里包含很多假定，班次时段是固定的，人员班组页面没有出现请假等情况。排班管理还需要考虑时间、人员、场所变化的情形，这个需要我们从变化的情景中找到合适的规律。下面我们根据一个时间和地点均存在变化的例子来分析排班。

某部门外勤某日排班表（2020年8月17日）如下：

| 部门   | 岗位类型   | 执勤地点 | 时段       | 人员                   |
| ------ | ---------- | -------- | ---------- | ---------------------- |
| 一大队 | 早晚高峰岗 | 四公里   | 7:00-10:00 | 令狐冲 郭靖 杨过       |
| 一大队 | 早晚高峰岗 | 五公里   | 7:00-10:00 | 任盈盈 黄蓉 小龙女     |
| 一大队 | 早晚高峰岗 | 六公里   | 7:00-10:00 | 乔峰 虚竹 段誉         |
| 一大队 | 路面巡逻岗 |          | 7:00-10:00 | 黄药师 一灯大师 周伯通 |
| 二大队 | 早晚高峰岗 | 茶园     | 7:00-9:30  | 令狐冲 郭靖 杨过       |
| 二大队 | 早晚高峰岗 | 长生桥   | 7:00-9:30  | 任盈盈 黄蓉 小龙女     |
| ...    |            |          |            |                        |

站在支队领导的角度，支队对各所属大队提出每日执勤要求包含：部门、岗位类型、执勤地点、时段、人员数量。一个部门下有多个岗位类型，同一岗位类型有多个执勤地点，部门、岗位类型、执勤地点可以设置不同的时段和多个人员。每日的排班表，会对岗位类型和执勤地点进行适当的调整。大致上，周一到周五的排班表内容基本相同，周末有所变化，节假日需做针对性安排。

外勤的业务模式具有支队安排，大队上报人员的特点，由支队安排需要执勤的岗位、地点、时段，由大队管理员上报每天的执勤人员。支队以周为单位，安排周一至周天的每天勤务安排，可设置节假日和指定日期的勤务。

如何根据这些规则进行的合适的勤务安排呢？

#### 勤务模板

勤务模板，勤务模板是需求落地通往实现的桥梁。模板的存在是对特定规则的一个抽象，需要我们抽象出模板的用途、分类、状态、规则等信息。就排班管理而言需要：模板名称、模板类型、模板状态、日期选项、时段规则等信息。注意这里的日期选项和时段规则是合集关系，一个日期选项可以包含多个时段规则，时段通俗的叫法也称班次。这里的日期选项可以归纳为工作日、周末、节假日、星期等范围选项。在人员安排时，可根据日期查找对应的日期选项，找到对应的时段规则，然后根据时段规则分配人员。

```
假设某单位有如下上班时间设置：
周一到周五每天八点到下午五点上班。
日期选项为：工作日，时段规则为：8:00-17:00

假设某单位有如下上班时间设置：
周一上午八点到十二点上班，下午休息。
周二到周五每天八点到到下午五点上班。
这里日期选项为：星期，星期包含：星期一、星期二、星期三、星期四、星期五、星期六、星期日。
规则：星期一 8:00-12:00 下午休息
     星期二 8:00-17:00
     星期三 8:00-17:00
     星期四 8:00-17:00
     星期五 8:00-17:00
     星期六 休息
     星期日 休息
```

#### 勤务模板特征

按照星期来执行时段班次的编排，是一个常用的途径。星期排班有以下几个特点：

1. 循环性，星期具有天然的循环滚动属性，总是一周接着一周，每周天数固定7天。
2. 直观性，可横向比较（比如周一和周二），也可纵向比较（比如本周一和上周一）。
3. 规则影响范围有限，一个周的排班规则设置好之后，修改指定星期时只影响未来，不影响过去。假设现在是周三，修改周四的规则表示修改明天的规则；修改周一的规则，表示修改下周的规则，对本周一不影响。
4. 工作日容易实现，按照国家法律规定实行五天工作制，周一到周五为工作日，周末休息。工作日的排班一般通过周一到周五来实现，但还需要考虑法定节假日范围的影响。

星期+周末+节假日能够满足一年中所有日期场景。

根据每周设置执勤岗位。

{% asset_img 勤务模板-每周.png %}

根据节假日设置执勤岗位，同理可对指定日期进行勤务安排。

{% asset_img 勤务模板-节假日.png %}

延伸，既然星期有连续性也可循环迭代，那是否还有其他日期选项可以供模板设置呢？根据按天循环勤务安排，也是属于一个备选方案。我们可以设置天数范围用于周期性循环，再设置一个起始日期作为生效的开始条件，即可模拟星期的循环性。按月天数范围一般不超过31天。同理可设置按年进行设置，天数范围不超过366天。

{% asset_img 勤务模板-每月.png %}

### 总结

总结，对于工作场所和班次固定的排班，可按工作日、周末、节假日设置相应的班次时间段。对于工作场所、日期和时段不固定的情况，推荐使用星期加节假日排班，按天数循环方案备选。星期的循环连续特性，能够满足大多数场景，当然也存在特殊情况。特殊情况应当特殊处理，比如设置特定日期的班次规则。需要指出的是，一般而言指定日期的设置优先级高于节假日，节假日的优先级高于星期设置，星期设置的优先级最低。

勤务模板管理可根据部门、工作场所、日期、时段的需要，配置不同类型的模板。

{% asset_img 勤务模板-模板列表.png %}

有了排班模板后，下一步就是根据将人员分配到日期上，同时满足对应的时段（班次）规则。另外再补充说明下，时段的范围需要考虑跨天的情景，单个时段的总时长不能超过24小时，多个时段的时长合计小于等于24小时。

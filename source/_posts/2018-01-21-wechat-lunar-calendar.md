---
title: 微信小程序的农历选择器
date: 2018-01-21 00:13:25
tags:
 - wechat
 - tech
 - javascript

---

![](http://ichengchao.oss-cn-hangzhou.aliyuncs.com/static/image/qrcode_birthday.jpg)

### 背景
小程序出来已经有一段时间了,能看到微信团队对小程序的期望非常大,API也越来越丰富.我个人也是非常看好这个方向,虽然现在还不是特别火,但只要有合适的场景,我相信小程序一定会爆发的(看看最近的"跳一跳"有多火).我自己也做了一个小程序,用于生日提醒(可以扫描上面的二维码试用一下).做的过程中碰到了一个小问题,就是小程序没有原生的农历日期选择器.于是就自己做了一个,有需要的朋友可以直接拿去用.


### Code

小程序官方组件中有个叫[picker](https://mp.weixin.qq.com/debug/wxadoc/dev/component/picker.html)的选择器组件,我的思路就是基于`mode = multiSelector`的方式来做的.

```xml
<view class="weui-cell__bd">
    <picker name="lunarPicker" mode="multiSelector" bindchange="bindLunarPickerChange" bindcolumnchange="bindLunarPickerColumnChange" value="{{lunarIndex}}" range="{{lunarArray}}" range-key="name">
        <view class="weui-input">{{lunarDate}}</view>
    </picker>
</view>
```
这里会用到两个方法

- bindLunarPickerChange
- bindLunarPickerColumnChange

在对应的js文件中是这样的:

```js

Page({
  data: {
    lunarArray: [],
    lunarIndex: [],
    lunarDate: "loading..."
  },

  bindLunarPickerChange: function (e) {
    let pickerIndex = e.detail.value;
    this.setData({
      lunarIndex: pickerIndex,
      lunarDate: lunarCalendar.getDisplayNameByPickerIndex(this.data.lunarArray, pickerIndex)
    });
  },

  bindLunarPickerColumnChange: function (e) {
    let list = this.data.lunarArray;
    if (e.detail.column == 0) {
      let year = list[0][e.detail.value].id;
      this.setData({
        lunarArray: lunarCalendar.rebuildPickerArrayOnYearChange(list, year),
        lunarIndex: [e.detail.value, 0, 0]
      });
    } else if (e.detail.column == 1) {
      let month = list[1][e.detail.value].id;
      let year = list[0][this.data.lunarIndex[0]].id;
      let isLeap = list[1][e.detail.value].name.charAt(0) == '闰' ? true : false;
      this.setData({
        lunarArray: lunarCalendar.rebuildPickerArrayOnMonthChange(list, year, month, isLeap),
        lunarIndex: [this.data.lunarIndex[0], e.detail.value, 0]
      });
    }
  },
  
  onLoad: function (options) {
    //默认初始化到1990-01-01
    let initIndex = [90, 0, 0];
    let initList = lunarCalendar.initPickerArray(1990);
    that.setData({
      lunarArray: initList,
      lunarIndex: initIndex,
      lunarDate: lunarCalendar.getDisplayNameByPickerIndex(initList, initIndex)
    });
  },

});

```




工具类:

```js

/**
 * 农历工具类,支持1900到2100
 */
var lunarCalendar = {

  /**
   * 农历1900-2100的信息表(包含闰月,以及月天数等信息)
   */
  metaArray: [
    0x04bd8, 0x04ae0, 0x0a570, 0x054d5, 0x0d260, 0x0d950, 0x16554, 0x056a0, 0x09ad0, 0x055d2, //1900-1909
    0x04ae0, 0x0a5b6, 0x0a4d0, 0x0d250, 0x1d255, 0x0b540, 0x0d6a0, 0x0ada2, 0x095b0, 0x14977, //1910-1919
    0x04970, 0x0a4b0, 0x0b4b5, 0x06a50, 0x06d40, 0x1ab54, 0x02b60, 0x09570, 0x052f2, 0x04970, //1920-1929
    0x06566, 0x0d4a0, 0x0ea50, 0x06e95, 0x05ad0, 0x02b60, 0x186e3, 0x092e0, 0x1c8d7, 0x0c950, //1930-1939
    0x0d4a0, 0x1d8a6, 0x0b550, 0x056a0, 0x1a5b4, 0x025d0, 0x092d0, 0x0d2b2, 0x0a950, 0x0b557, //1940-1949
    0x06ca0, 0x0b550, 0x15355, 0x04da0, 0x0a5b0, 0x14573, 0x052b0, 0x0a9a8, 0x0e950, 0x06aa0, //1950-1959
    0x0aea6, 0x0ab50, 0x04b60, 0x0aae4, 0x0a570, 0x05260, 0x0f263, 0x0d950, 0x05b57, 0x056a0, //1960-1969
    0x096d0, 0x04dd5, 0x04ad0, 0x0a4d0, 0x0d4d4, 0x0d250, 0x0d558, 0x0b540, 0x0b6a0, 0x195a6, //1970-1979
    0x095b0, 0x049b0, 0x0a974, 0x0a4b0, 0x0b27a, 0x06a50, 0x06d40, 0x0af46, 0x0ab60, 0x09570, //1980-1989
    0x04af5, 0x04970, 0x064b0, 0x074a3, 0x0ea50, 0x06b58, 0x055c0, 0x0ab60, 0x096d5, 0x092e0, //1990-1999
    0x0c960, 0x0d954, 0x0d4a0, 0x0da50, 0x07552, 0x056a0, 0x0abb7, 0x025d0, 0x092d0, 0x0cab5, //2000-2009
    0x0a950, 0x0b4a0, 0x0baa4, 0x0ad50, 0x055d9, 0x04ba0, 0x0a5b0, 0x15176, 0x052b0, 0x0a930, //2010-2019
    0x07954, 0x06aa0, 0x0ad50, 0x05b52, 0x04b60, 0x0a6e6, 0x0a4e0, 0x0d260, 0x0ea65, 0x0d530, //2020-2029
    0x05aa0, 0x076a3, 0x096d0, 0x04afb, 0x04ad0, 0x0a4d0, 0x1d0b6, 0x0d250, 0x0d520, 0x0dd45, //2030-2039
    0x0b5a0, 0x056d0, 0x055b2, 0x049b0, 0x0a577, 0x0a4b0, 0x0aa50, 0x1b255, 0x06d20, 0x0ada0, //2040-2049
    0x14b63, 0x09370, 0x049f8, 0x04970, 0x064b0, 0x168a6, 0x0ea50, 0x06b20, 0x1a6c4, 0x0aae0, //2050-2059
    0x0a2e0, 0x0d2e3, 0x0c960, 0x0d557, 0x0d4a0, 0x0da50, 0x05d55, 0x056a0, 0x0a6d0, 0x055d4, //2060-2069
    0x052d0, 0x0a9b8, 0x0a950, 0x0b4a0, 0x0b6a6, 0x0ad50, 0x055a0, 0x0aba4, 0x0a5b0, 0x052b0, //2070-2079
    0x0b273, 0x06930, 0x07337, 0x06aa0, 0x0ad50, 0x14b55, 0x04b60, 0x0a570, 0x054e4, 0x0d160, //2080-2089
    0x0e968, 0x0d520, 0x0daa0, 0x16aa6, 0x056d0, 0x04ae0, 0x0a9d4, 0x0a2d0, 0x0d150, 0x0f252, //2090-2099
    0x0d520
  ], //2100

  zodiacArray: ["猴", "鸡", "狗", "猪", "鼠", "牛", "虎", "兔", "龙", "蛇", "马", "羊"],
  monthArray: ['正月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '冬月', '腊月'],
  dayArray: ["初一", "初二", "初三", "初四", "初五", "初六", "初七", "初八", "初九", "初十", "十一", "十二", "十三", "十四", "十五", "十六", "十七", "十八", "十九", "二十", "廿一", "廿二", "廿三", "廿四", "廿五", "廿六", "廿七", "廿八", "廿九", "三十"],

  toChinaYear: function (y) {
    return lunarCalendar.zodiacArray[y % 12];
  },

  toChinaMonth: function (m) {
    return lunarCalendar.monthArray[m - 1];
  },

  toChinaDay: function (d) {
    return lunarCalendar.dayArray[d - 1];
  },

  /**
   * 计算指定年的闰月月份,如果该年没有闰月,则返回0
   */
  getLeapMonth: function (y) {
    return (lunarCalendar.metaArray[y - 1900] & 0xf);
  },


  /**
   * 计算指定年,月的天数 
   */
  getDayCountOfMonth: function (y, m, isLeap) {
    if (isLeap) {
      let leapMonth = lunarCalendar.getLeapMonth(y);
      if (leapMonth != 0 && leapMonth == m) {
        return ((lunarCalendar.metaArray[y - 1900] & 0x10000) ? 30 : 29);
      }
    } else {
      return ((lunarCalendar.metaArray[y - 1900] & (0x10000 >> m)) ? 30 : 29);
    }
  },

  /**
   * 默认生成1900到2100的年列表,这里改需要metaAArray的支持
   */
  buildYearList: function () {
    let list = [];
    for (let i = 1900; i <= 2100; i++) {
      list.push({
        id: i,
        name: i + "[" + lunarCalendar.toChinaYear(i) + "]"
      });
    }
    return list;
  },


  /**
   * 根据年生成月列表
   */
  buildMonthList: function (y) {
    let list = [];
    for (let i = 1; i <= 12; i++) {
      list.push({
        id: i,
        name: lunarCalendar.toChinaMonth(i)
      })
    }
    let leapMonth = lunarCalendar.getLeapMonth(y);
    if (leapMonth != 0) {
      list.splice(leapMonth, 0, {
        id: leapMonth,
        name: "闰" + lunarCalendar.toChinaMonth(leapMonth)
      })
    }
    return list;
  },

  buildDayList: function (y, m, isLeap) {
    let list = [];
    let dayCount = lunarCalendar.getDayCountOfMonth(y, m, isLeap);
    for (let i = 1; i <= dayCount; i++) {
      list.push({
        id: i,
        name: lunarCalendar.toChinaDay(i)
      })
    }
    return list;
  },

  initPickerArray: function (y) {
    let list = [];
    list[0] = lunarCalendar.buildYearList();
    list[1] = lunarCalendar.buildMonthList(y);
    list[2] = lunarCalendar.buildDayList(y, 1, false);
    return list;
  },

  rebuildPickerArrayOnYearChange: function (list, y) {
    list[1] = lunarCalendar.buildMonthList(y);
    list[2] = lunarCalendar.buildDayList(y, 1, false);
    return list;
  },
  rebuildPickerArrayOnMonthChange: function (list, y, m, isLeap) {
    list[2] = lunarCalendar.buildDayList(y, m, isLeap);
    return list;
  },

  getDisplayNameByPickerIndex: function (list, indexList) {
    return list[0][indexList[0]].name + " " + list[1][indexList[1]].name + " " + list[2][indexList[2]].name;
  }

};


```



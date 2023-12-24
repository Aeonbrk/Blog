---
title: NJUPT-automatic-evaluation-script
author: Haibin
pubDatetime: 2023-12-24T17:50:00+8:00
featured: false
draft: false
tags:
  - School
ogImage: ""
description: A configuration for Ubuntu WSL2.
---

## NJUPT

[MinzhiYoyo/NJUPT-automatic-evaluation-script: 南京邮电大学期末自动评价脚本 (github.com)](https://github.com/MinzhiYoyo/NJUPT-automatic-evaluation-script)

```js
/*
 * @Author: 梁敏智
 * @Date: 2022-05-28 18:38:53
 * @LastEditors: 梁敏智
 * @LastEditTime: 2022-05-28 18:40:08
 * @Description: 用来南京邮电大学满意度评价的脚本，教学质量评价->
 */

// 教学质量评价
var obj = frames["iframeautoheight"].contentDocument.getElementById("pjkc");
var length = obj.options.length;
console.log("你有", length, "门课程需要进行教学质量");
var finished = 0;
var task = window.setInterval(function () {
  if (finished == length - 1) {
    window.clearInterval(task);
  }
  var allselects =
    frames["iframeautoheight"].contentDocument.getElementsByTagName("select");
  for (var j = 1; j < allselects.length; j++) {
    if (j % 7 == 1) {
      // 每7个选项选一个为较好，剩下的都选好，可以自己在这里编辑
      allselects[j].value = "较好";
    } else allselects[j].value = "好";
  }
  finished++;
  frames["iframeautoheight"].contentDocument.getElementById("Button1").click();
  console.log("任务进度：", finished, " / ", length, " 门课");
}, 1000);
```

```js
/*
 * @Author: 梁敏智
 * @Date: 2022-05-28 18:31:27
 * @LastEditors: 梁敏智
 * @LastEditTime: 2022-05-28 18:40:25
 * @Description: 用来南京邮电大学满意度评价的脚本，问卷调查->课程评价
 */

// 满意度评价
var obj = frames["iframeautoheight"].contentDocument.getElementById("pjkc");
var length = obj.options.length;
console.log("你有", length, "门课程需要进行课程评价");
var finished = 0;
var task = window.setInterval(function () {
  if (finished == length - 1) {
    window.clearInterval(task);
  }
  var allselects =
    frames["iframeautoheight"].contentDocument.getElementsByTagName("select");
  for (var j = 1; j < allselects.length; j++) {
    if (j == allselects.length / 2 - 1 || j == allselects.length / 2 + 1) {
      allselects[j].value = "相对认同"; // 中间两个选相对认同，剩下的选认同
    } else allselects[j].value = "完全认同";
  }
  finished++;
  frames["iframeautoheight"].contentDocument.getElementById("Button1").click();
  console.log("任务进度：", finished, " / ", length, " 门课");
}, 1000);
```

---

## CRK

```js
var obj = document.getElementById("table1");
var elements = obj.querySelectorAll('[name="zbtd"]');

for (var i = 0; i < elements.length; i++) {
  var element = elements[i];
  var inputElements = element.getElementsByTagName("input");

  for (var j = 0; j < inputElements.length; j++) {
    var inputElement = inputElements[j];
    if (inputElement.type == "radio" && j % 8 == 0) {
      inputElement.checked = true;
    }
  }
}
```

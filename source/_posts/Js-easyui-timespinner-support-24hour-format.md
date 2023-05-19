---
title: 让EasyUI timespinner支持24小时格式
date: 2017-12-19 13:00:00
tags:
  - 开发笔记
  - JavaScript
categories:
  - JavaScript
toc: false
---
EasyUI中timespinner最大值为23:59，项目中有一个需求需要显示24:00. 调试了下源代码，发现EasyUI，在赋值时调用parser函数先将字符串转换为Date对象，然后用获取到的Date对象与最大值和最小值进行比较。小于最小值时，将当前值设置为最小值，大于最大值时，当前值等于最大值。在获取到Date对象后，调用formatter函数进行格式转换。
最终的解决方案如下。
```
	$.fn.timespinner.defaults.formatter = function(_23) {
		if (!_23) {
			return "";
		}
		var _24 = $(this).timespinner("options");
		var tt = [_25(_23.getDay() * 24 + _23.getHours()), _25(_23.getMinutes())];
		if (_24.showSeconds) {
			tt.push(_25(_23.getSeconds()));
		}
		return tt.join(_24.separator);
		function _25(_26) {
			return (_26 < 10 ? "0": "") + _26;
		};
	};

	$.fn.timespinner.defaults.parser = function(s) {
		var _27 = $(this).timespinner("options");
		var _28 = _29(s);
		if (_28) {
			var min = _29(_27.min || "00:00");
			var max = _29(_27.max || "24:00");
			if (min && min > _28) {
				_28 = min;
			}
			if (max && max < _28) {
				_28 = max;
			}
		}
		return _28;
		function _29(s) {
			if (!s) {
				return null;
			}
			var tt = s.split(_27.separator);
			return new Date(1900, 0, 0, parseInt(tt[0], 10) || 0, parseInt(tt[1], 10) || 0, parseInt(tt[2], 10) || 0);
		};
	};
```
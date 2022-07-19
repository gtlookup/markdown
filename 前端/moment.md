```javascript
moment.locale('zh-cn'); // 中文

moment().format('YYYY-MM-DD HH:mm:ss.SSSS'); // 当前时间格式化到毫秒
moment(new Date()).add(-1, 'months'); // 前一月
moment().toDate(); // 转换成日期类型
/* 两个日期相减(左大右小)
// years
// months
// days
// hours
// minutes
// seconds
// milliseconds*/
moment('2019-02-02').diff(moment('2018-01-01'), 'days')
moment(dateTime, 'YYYY/MM/DD HH:mm:ss.SSSS').toDate().getTime() //获取时间戳
moment(new Date()).utcOffset(分钟); // 时区
moment().day(); // 周几
moment('abcdef').isValid(); // 判断有效日期
```


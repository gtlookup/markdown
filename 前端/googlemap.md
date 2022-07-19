https://www.runoob.com/googleapi/google-maps-basic.html

参考菜鸟教程取得api key：AIzaSyDJW4jsPlNKgv6jFm3B5Edp5ywgdqLWdmc

```html
<div id="gmap"></div>
<!-- 导入库 -->
<script src="https://maps.googleapis.com/maps/api/js?&key=AIzaSyDJW4jsPlNKgv6jFm3B5Edp5ywgdqLWdmc&callback=initMap" async defer></script>
```

```javascript
function initMap() {
  //let addr = '国,省,市,区,街道';
  let addr = '中国,辽宁省,大连市';
  new google.maps.Geocoder().geocode({'address': addr}, function(rlt, status) {
    console.log(rlt); // 返回一个根据 addr 查找到的结果（数组），如果没查到则返回一个空数组
    let loc = rlt[0].geometry.location; // 取得结果集中第一个元素的经纬度
    new google.maps.Map(document.getElementById('gmap'), {
      center: {lat: loc.lat(), lng: loc.lng()}, // 根据经纬度定位地图位置
      zoom: 14
    });
  });
}
```


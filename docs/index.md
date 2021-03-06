## Google Earth Engine个人笔记：1 加载与显示影像

### 1、gee加载与显示单幅影像

```markdown
var dataset = ee.ImageCollection('MODIS/006/MOD13A1')
                  .filter(ee.Filter.date('2018-01-01', '2018-05-01'));
var ndvi = dataset.select('NDVI');
var ndviVis = {
  min: 0.0,
  max: 9000.0,
  palette: [
    'FFFFFF', 'CE7E45', 'DF923D', 'F1B555', 'FCD163', '99B718', '74A901',
    '66A000', '529400', '3E8601', '207401', '056201', '004C00', '023B01',
    '012E01', '011D01', '011301'],
};
Map.setCenter(6.746, 46.529, 2);
Map.addLayer(ndvi, ndviVis, 'NDVI');
```
### 2、gee加载与显示影像集

很多时候我们在gee需要加载很多影像制作为影像集合，并且显示这个影像集合中的所有影像。使用addLayer加载影像集合时，只会默认显示影像集合的第一幅影像，这时候可以借助循环语句进行加载和显示影像。
```markdown
// 产生一个2018到2020的list
var time = ee.List.sequence(2018, 2020);
// 加载MOD13A1.006 500m、16天EVI产品
var NDVI = time.map(function(year){
  var year1  = ee.Number(year).format("%04d").cat("-01-01");
  var year2  = ee.Number(year).format("%04d").cat("-12-31");
  var image = ee.ImageCollection("MODIS/006/MOD13A1")
    .select('EVI')
    .filterDate(year1,year2)
    .median() 
    .setMulti({"system:index":year1,'system:time_start':ee.Date(year1)});
    return image;
  });
var NDVI_gather = ee.ImageCollection(NDVI);
// 添加NDVI图层
var ndviVis = {
  min: 0.0,
  max: 9000.0,
  palette: [
    'FFFFFF', 'CE7E45', 'DF923D', 'F1B555', 'FCD163', '99B718', '74A901',
    '66A000', '529400', '3E8601', '207401', '056201', '004C00', '023B01',
    '012E01', '011D01', '011301' ],
};
var rawLayer = null;
var computedIds = NDVI_gather.reduceColumns(ee.Reducer.toList(), ['system:index'])
                        .get('list');
computedIds.evaluate(function(ids) {
  for (var i=0; i<ids.length; i++) {
    var key = ids[i];
    var image = ee.Image(NDVI_gather.filter(ee.Filter.eq("system:index", key)).first());
    Map.addLayer(image, ndviVis, key);
  }
});
```

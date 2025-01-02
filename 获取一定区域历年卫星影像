// 定义研究区，并将研究区显示在地图上

Map.centerObject(geometry2);

// 去云处理函数
function maskS2clouds(image) {
  var qa = image.select('QA60');
  // Bits 10和11分别是云和卷云
  var cloudBitMask = 1 << 10;
  var cirrusBitMask = 1 << 11;
  // Both flags should be set to zero, indicating clear conditions.
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0)
      .and(qa.bitwiseAnd(cirrusBitMask).eq(0));
  return image.updateMask(mask).divide(10000); // DN值转换为反射率
}

// 逐年处理影像
for (var year = 2016; year <= 2024; year++) {
  var startDate = ee.Date.fromYMD(year, 1, 1);
  var endDate = ee.Date.fromYMD(year, 12, 31);
  
  // 获取Sentinel-2 1C级影像
  var dataset = ee.ImageCollection('COPERNICUS/S2_HARMONIZED')
                  .filterDate(startDate, endDate)
                  .filterBounds(geometry2)
                  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 10)) // 云量小于10%
                  .map(maskS2clouds)
                  .select(['B4', 'B3', 'B2']); // 选择假彩色合成的波段
  
  // 计算每年的中值合成影像
  var medianImage = dataset.median().set('year', year);
  
  // 假彩色可视化参数
  var rgbVis = {
    min: 0.0,
    max: 0.3,
    bands: ['B4', 'B3', 'B2'],
  };
  
  // 可视化每年的影像
  Map.addLayer(medianImage.clip(geometry2), rgbVis, 'Median Image ' + year);
  
  // 导出每年的影像
  Export.image.toDrive({
    image: medianImage.clip(geometry2),
    description: 'Sentinel2_Median_' + year,
    folder: 'GEE_Exports',
    fileNamePrefix: 'Sentinel2_Median_' + year,
    region: geometry2,
    scale: 10,
    maxPixels: 1e13
  });
}

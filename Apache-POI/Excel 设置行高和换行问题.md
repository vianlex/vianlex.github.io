---
title: Excel 设置行高和换行问题
---

## Excel 单元格设置强制换行，使用符号 \r\n 可以强制换行

```java
//单元格数据
String data = "假如生活欺骗了你，不要悲伤，不要心急！忧郁的日子里需要镇静，相信吧，快乐的日子将会来临。"
// 创建行
Row row = sheet.createRow(0);
// 设置默认的行高
row.setHeight(10);
// 创建样式
HSSFCellStyle style = workbook.createCellStyle();
style.setWrapText(true);
// 创建单元格
Cell cell = row.createCell(0);
cell.setCellStyle(style);
//设置默认的列宽
sheet.setDefaultColumnWidth(20);
// 强制实现换行 data
StringBuilder sbData = new StringBuilder(data)
int length = sbData.length();
int rowHeightLine = 1; 
for(int i = width - 1; i < length; i = i + width + 1 ) {
    System.out.println("换行符位置：" + i);
    sbData.insert(i, "\r\n");
    rowHeightLine++;
}
row.setHeight(row.getHeight() * rowHeightLine)
cell.setCellValue(sbData.toString());

```


## 参考链接
https://stackoverflow.com/questions/19145628/auto-size-height-for-rows-in-apache-poi#
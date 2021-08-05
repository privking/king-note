# easyExcel

```java
package com.bigdatacd.adminenterpriseclaim.util.excel;

import com.alibaba.excel.enums.CellDataTypeEnum;
import com.alibaba.excel.metadata.CellData;
import com.alibaba.excel.metadata.Head;
import com.alibaba.excel.util.CollectionUtils;
import com.alibaba.excel.write.metadata.holder.WriteSheetHolder;
import com.alibaba.excel.write.style.column.AbstractColumnWidthStyleStrategy;
import org.apache.poi.ss.usermodel.Cell;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @author king
 * TIME: 2020/6/30 - 9:29
 * 自适应宽度
 **/
public class ColumnWidthWriteHandler extends AbstractColumnWidthStyleStrategy {



    private static final int MAX_COLUMN_WIDTH = 255;
    private  Map<Integer, Map<Integer, Integer>> CACHE = new HashMap(8);

    public ColumnWidthWriteHandler() {
    }

    protected void setColumnWidth(WriteSheetHolder writeSheetHolder, List<CellData> cellDataList, Cell cell, Head head, Integer relativeRowIndex, Boolean isHead) {
        boolean needSetWidth = isHead || !CollectionUtils.isEmpty(cellDataList);
        if (needSetWidth) {
            Map<Integer, Integer> maxColumnWidthMap = (Map)CACHE.get(writeSheetHolder.getSheetNo());
            if (maxColumnWidthMap == null) {
                maxColumnWidthMap = new HashMap(16);
                CACHE.put(writeSheetHolder.getSheetNo(), maxColumnWidthMap);
            }

            Integer columnWidth = this.dataLength(cellDataList, cell, isHead);
            if (columnWidth >= 0) {
                if (columnWidth > MAX_COLUMN_WIDTH) {
                    columnWidth = MAX_COLUMN_WIDTH;
                }

                Integer maxColumnWidth = (Integer)((Map)maxColumnWidthMap).get(cell.getColumnIndex());
                if (maxColumnWidth == null || columnWidth > maxColumnWidth) {
                    ((Map)maxColumnWidthMap).put(cell.getColumnIndex(), columnWidth);
                    writeSheetHolder.getSheet().setColumnWidth(cell.getColumnIndex(), columnWidth * 128);
                }

            }
        }
    }

    private Integer dataLength(List<CellData> cellDataList, Cell cell, Boolean isHead) {
        if (isHead) {
            return cell.getStringCellValue().getBytes().length;
        } else {
            CellData cellData = (CellData)cellDataList.get(0);
            CellDataTypeEnum type = cellData.getType();
            if (type == null) {
                return -1;
            } else {
                switch(type) {
                    case STRING:
                        return cellData.getStringValue().getBytes().length;
                    case BOOLEAN:
                        return cellData.getBooleanValue().toString().getBytes().length;
                    case NUMBER:
                        return cellData.getNumberValue().toString().getBytes().length;
                    default:
                        return -1;
                }
            }
        }
    }
}



```

```java
package com.bigdatacd.adminenterpriseclaim.util.excel;


import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.write.builder.ExcelWriterSheetBuilder;
import com.alibaba.excel.write.metadata.style.WriteCellStyle;
import com.alibaba.excel.write.style.HorizontalCellStyleStrategy;
import com.google.common.collect.Lists;

import java.io.IOException;
import java.io.OutputStream;
import java.util.List;

/**
 * @author king
 * TIME: 2020/6/29 - 16:53
 **/
public class EnterpriseInfoExcelUtil {
    public static List<List<String>> head(String firstTatle) {
        List<List<String>> headTitles = Lists.newArrayList();
        List<String> secondTitles = Lists.newArrayList("编号", "企业规模（等级）",
                "门店名称", "执照名称", "统一社会信用代码", "注册号", "行政区域（镇/街道)", "详细地址");
        secondTitles.forEach(title -> {
            headTitles.add(Lists.newArrayList(firstTatle, firstTatle, firstTatle, firstTatle, title, title, title));
        });
        headTitles.add(Lists.newArrayList(firstTatle, firstTatle, firstTatle, firstTatle, "经纬度", "经纬度", "经度"));
        headTitles.add(Lists.newArrayList(firstTatle, firstTatle, firstTatle, firstTatle, "经纬度", "经纬度", "纬度"));

        List<String> thirdTitles = Lists.newArrayList("生产状态", "认领状态", "填报状态");
        thirdTitles.forEach(title -> {
            headTitles.add(Lists.newArrayList(firstTatle, firstTatle, firstTatle, firstTatle, title, title, title));
        });
        return headTitles;
    }

    public static List<List<Object>> testData() {
        List<List<Object>> contentList = Lists.newArrayList();
        //这里一个List<Object>才代表一行数据，需要映射成每行数据填充，横向填充（把实体数据的字段设置成一个List<Object>）
        contentList.add(Lists.newArrayList("测试", "商品A", "苹果🍎", "aa", "aa", "aa", "aa", "aa", "aa", "aa", "aa", "aa", "aa"));
        contentList.add(Lists.newArrayList("测试", "商品B", "橙子🍊", "aa", "aa", "aa", "aa", "aa", "aa", "aa", "aa", "aa", "aa"));
        return contentList;
    }


    public static void write(OutputStream outputStream, String firstTatle, List<List<Object>> contentData) throws IOException {
        // 头的策略
        WriteCellStyle headWriteCellStyle = new WriteCellStyle();
        // 单元格策略
        WriteCellStyle contentWriteCellStyle = new WriteCellStyle();
        // 初始化表格样式
        HorizontalCellStyleStrategy horizontalCellStyleStrategy = new HorizontalCellStyleStrategy(headWriteCellStyle, contentWriteCellStyle);

        ExcelWriterSheetBuilder excelWriterSheetBuilder = EasyExcel.write(outputStream).head(head(firstTatle)).sheet("sheet1").registerWriteHandler(horizontalCellStyleStrategy);

        excelWriterSheetBuilder.registerWriteHandler(new TitleWriteHandler());
        excelWriterSheetBuilder.registerWriteHandler(new ColumnWidthWriteHandler());
        excelWriterSheetBuilder.doWrite(contentData);
        outputStream.flush();
        outputStream.close();
    }
}


```

```java
package com.bigdatacd.adminenterpriseclaim.util.excel;

import com.alibaba.excel.metadata.CellData;
import com.alibaba.excel.metadata.Head;
import com.alibaba.excel.util.StyleUtil;
import com.alibaba.excel.write.handler.CellWriteHandler;
import com.alibaba.excel.write.metadata.holder.WriteSheetHolder;
import com.alibaba.excel.write.metadata.holder.WriteTableHolder;
import com.alibaba.excel.write.metadata.style.WriteCellStyle;
import com.alibaba.excel.write.metadata.style.WriteFont;
import org.apache.poi.ss.usermodel.*;

import java.util.List;

/**
 * @author king
 * TIME: 2020/6/30 - 9:28
 **/
public class TitleWriteHandler implements CellWriteHandler {




    public TitleWriteHandler() {
    }

    @Override
    public void beforeCellCreate(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, Row row, Head head, Integer columnIndex, Integer relativeRowIndex, Boolean isHead) {
    }

    @Override
    public void afterCellCreate(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, Cell cell, Head head, Integer relativeRowIndex, Boolean isHead) {
    }

    @Override
    public void afterCellDataConverted(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, CellData cellData, Cell cell, Head head, Integer integer, Boolean aBoolean) {

    }

    @Override
    public void afterCellDispose(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, List<CellData> cellDataList, Cell cell, Head head, Integer relativeRowIndex, Boolean isHead) {
        //row  行
        if(cell.getRowIndex()<7 && cell.getColumnIndex()<13){
            // 设置列宽
            Sheet sheet = writeSheetHolder.getSheet();
            sheet.setColumnWidth(cell.getColumnIndex(), 20 * 256);
            // 设置行高
            writeSheetHolder.getSheet().getRow(0).setHeight((short)(2.0*256));
            // 获取workbook
            Workbook workbook = writeSheetHolder.getSheet().getWorkbook();
            // 获取样式实例
            WriteCellStyle headWriteCellStyle = new WriteCellStyle();
            // 获取字体实例
            WriteFont headWriteFont = new WriteFont();

            headWriteFont.setBold(true);
            headWriteCellStyle.setWriteFont(headWriteFont);
            // 设置背景颜色
            if(cell.getRowIndex()<4){
                // 设置字体样式
                headWriteFont.setFontName("宋体");
                // 设置字体大小
                headWriteFont.setFontHeightInPoints((short)20);
                headWriteCellStyle.setFillForegroundColor(IndexedColors.WHITE.getIndex());
            }else{
                // 设置字体样式
                headWriteFont.setFontName("宋体");
                // 设置字体大小
                headWriteFont.setFontHeightInPoints((short)14);
                headWriteCellStyle.setFillForegroundColor(IndexedColors.YELLOW.getIndex());
            }
            // 设置指定单元格字体自定义颜色
            headWriteFont.setColor(IndexedColors.BLACK.getIndex());

            // 获取样式实例
            CellStyle cellStyle = StyleUtil.buildHeadCellStyle(workbook, headWriteCellStyle);
            // 单元格设置样式
            cell.setCellStyle(cellStyle);
        }

    }
}
```

![demo](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/demo-1628176595-c941b5.png)


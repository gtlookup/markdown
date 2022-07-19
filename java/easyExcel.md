# 1，POI P7

- 结构

  - HSSF：03 版 excel，==最大65535行==
  - XSSF：07 版 excel，无最大行限制
  - HWPF：word 文档
  - HSLF：ppt 文档
  - HDGF：visio 文档

- maven 依赖

  ```xml
  <!-- xls(03) -->
  <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi</artifactId>
      <version>3.9</version>
  </dependency>
  <!-- xlsx(07) -->
  <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-ooxml</artifactId>
      <version>3.9</version>
  </dependency>
  <!-- 日期格式化工具 -->
  <dependency>
      <groupId>joda-time</groupId>
      <artifactId>joda-time</artifactId>
      <version>2.10.5</version>
  </dependency>
  ```

## 1.1 写 demo

```java
// 1. 创建工作薄(03版)
//    XSSFWorkbook(07版)
Workbook wb = new HSSFWorkbook();
// 2. 创建sheet
Sheet sheet = wb.createSheet("A");
// 3. 创建行
Row row = sheet.createRow(0);
// 4. 创建格
Cell cell = row.createCell(0);
// 5. 写内容
cell.setCellValue("haha");
// 创建.xls文件
FileOutputStream stream = new FileOutputStream(path + "\\a.xls");
// 写内容
wb.write(stream);
// 关闭流
stream.close();
```

## 1.2 大文件写入

- HSSF

  - 缺点：最多写入 65536 行
  - 优点：缓存中操作，最后才写入磁盘

- XSSF

  - 缺点：写时非常慢非常耗内存，数据太大内存溢出（OOM）
  - 优点：比 03 写入的多。20w

- 写更大的文件 SXSSF

  - **优点：百万+，速度快，省内存**

  - 过程中产生临时文件，事后要清理临时文件
  - 默认内存留100条，超过部分写入临时文件
  - 自定义内存预留条数：**new SXSSFWorkbook(预留数量)**

  ```java
  SXSSFWorkbook wb = new SXSSFWorkbook();
  // 读/写 操作
  ...;
  // 最后要关闭
  wb.dispose();
  ```



# 2，easyExcel

- **阿里**开源的，快速、简单、避免 OOM 的 java 处理 Excel 工具

- https://github.com/alibaba/easyexcel

- 文档：https://www.yuque.com/easyexcel/doc/easyexcel

- maven

  ```xml
  <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>easyexcel</artifactId>
      <version>2.2.3</version>
  </dependency>
  <!-- 用来解决 log4j 报错问题 -->
  <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-nop</artifactId>
      <version>1.7.2</version>
  </dependency>
  ```

## 2.1 写操作

```java
@Data
public class WriteModel {
    @ExcelProperty("col str")
    private String string;
    @ExcelProperty("col date")
    private Date date;
    @ExcelProperty("col double")
    private Double number;

    // 忽略字段
    @ExcelIgnore
    private String ignore;
    /******** 以上为实体类部分 *********/

    /******** 以下为测试部分 *********/
    public static String path = "D:\\GT\\code\\java\\idea\\easy-excel";

    // 数据源
    private List<WriteModel> data() {
        List<WriteModel> list = new ArrayList<WriteModel>();
        for (int i = 0; i < 10; i++) {
            WriteModel data = new WriteModel();
            data.setString("字符串" + i);
            data.setDate(new Date());
            data.setNumber(0.56);
            list.add(data);
        }
        return list;
    }
    @Test
    public void T1() {
        String fileName = path + "\\easy.xlsx";
        // 实体类 -> sheet -> 数据源
        EasyExcel.write(fileName, WriteModel.class).sheet("SheetA").doWrite(data());
    }
}
```

# 附录

## 获取公式

```java
if (cell.getCellType() == HSSFCell.CELL_TYPE_FORMULA) {
	// 只是获取公式的表达式字符串
    String formula = cell.getCellFormula();
    CellValue evaluate = new XSSFFormulaEvaluator(wb).evaluate(cell);
    // 公式计算后的值
    String value = evaluate.formatAsString();
}
```

## 解决关联公式无效

```java
// 1.重新设置相应公式和值(尤其值左上绿前头)
DataFormat df = wb.createDataFormat();
// 重新设置值
sheet.getRow(5).createCell(15).setCellValue(1);
// 重新设置值格式(类型)
CellStyle style = wb.sheet.getRow(5).getCell(15).getCellStyle();
style.setDataFormat(df.getFormat("#,#0"));
sheet.getRow(5).getCell(15).setCellStyle(style);
// 重新设置公式
sheet.getRow(2).createCell(15).setCellFormula("P6");
// 打开后公式重新计算
wb.setForceFormulaRecalculation(true);
```

## 根据字母获取行列

```java
CellReference cr = new CellReference("D2");
Cell cell = sheet.getRow(cr.getRow()).getCell(cr.getCol());
```

## 获取所有合并的单元格

```java
List<CellRangeAddress> list = sh.getMergedRegions();
```

## 根据列号取列名

```java
CellReference.convertNumToColString(1); // B
```

## 根据列名取列号

```java
CellReference.convertColStringToIndex("A"); // 0
```

## 获取有效行数

```java
int ct = sheet.getPhysicalNumberOfRows();
```

## 获取有效列数

```java
int ct = row.getPhysicalNumberOfCells();
```

## 判断是不是日期

```java
HSSFDateUtil.isCellDateFormatted(cell);
```

## 读文件

```java
FileInputStream stream = new FileInputStream(path + "\\xxx.xlsx");
XSSFWorkbook wb = new XSSFWorkbook(stream);
Cell cell = wb.getSheetAt(0).getRow(65536).getCell(0);
System.out.println(cell.getStringCellValue());
stream.close();
```

## 判断类型

```java
switch (cell.getCellType()) {
    case XSSFCell.CELL_TYPE_STRING:
        break;
    case  XSSFCell.CELL_TYPE_NUMERIC:
        break;
}
```

## 单元格背景色

```java
// 是 Foreground，不是 Background
cell.getCellStyle().getFillForegroundColor()
```

## 单元格字体颜色

```java
Workbook wb = cell.getSheet().getWorkbook();
wb.getFontAt(cell.getCellStyle().getFontIndexAsInt()).getColor();
```

## 合并单元格所占行/列数

```java
// row: 合并单元格包含的行号
// col: 合并单元格包含的列号
public static List<Integer> getRegionOfXyCount(Sheet sheet, Integer row, Integer col) {
    // getMergedRegions()：取得 sheet 里所有的合并单元格
    List<CellRangeAddress> list = sheet.getMergedRegions()
        // containsRow / containsColumn：包含的行列号
        .stream().filter(x -> x.containsRow(row) && x.containsColumn(col))
        .collect(Collectors.toList());
    if (list.size() > 0) {
        CellRangeAddress cra = list.get(0);
        // last - first + 1 = 所占的行/列数
        return Arrays.asList(cra.getLastRow() - cra.getFirstRow() + 1, cra.getLastColumn() - cra.getFirstColumn() + 1);
    }
    return null;
}
```


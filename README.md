# excel-io

封装poi对Office Excel的输入输出工具，简化简单的导入和导出Excel数据的操作。（暂不支持合并单元格）

```xml
<dependency>
    <groupId>com.github.developframework</groupId>
    <artifactId>excel-io</artifactId>
</dependency>
```

## 教程

假设存在实体`Customer`包装数据

```java
@Data
public class Customer {

    private String name;

    private Date birthday;

    private String mobile;

    private int money;

}
```

### ExcelIO

使用`ExcelIO`得到输入输出处理器

#### ExcelWriter

```java
ExcelWriter excelWriter = ExcelIO.writer(ExcelType.XLSX, "E:\\test.xlsx");
excelWriter.fillData(customers, tableDefinition).write();
```
#### ExcelReader

```java
ExcelReader excelReader = ExcelIO.reader(ExcelType.XLSX, "E:\\test.xlsx")；
List<Customer> customers = excelReader.readAndClose(Customer.class, readSize, tableDefinition);
```

### TableDefinition

该接口是表格的定义类，一个定义类代表了一个数据表

```java
public interface TableDefinition {

    boolean hasHeader();

    String sheetName();

    Integer sheet();

    int column();

    int row();

    ColumnDefinition[] columnDefinitions(Workbook workbook);

    void tableHeaderCellStyle(Workbook workbook, CellStyle cellStyle);

}
```

通过该接口可以设置表格的表头信息及格式和表格的左上角单元格位置（工作表、行、列）。不过通常不直接使用该类，而使用其子类`AbstractTableDefinition` ，提供了默认的参数值。

| 参数                 | 说明               | 默认值 |
| -------------------- | ------------------ | ------ |
| hasHeader            | 是否有表头         | true   |
| sheetName            | 工作表名称         | null   |
| sheet                | 工作表             | null   |
| column               | 起始列             | 0      |
| row                  | 起始行             | 0      |
| columnDefinitions    | 列定义             | 未实现 |
| tableHeaderCellStyle | 设置表头单元格格式 |        |

### ColumnDefinition

该抽象类是表格的列定义类，一个定义类代表了表中的某一列，指代了一个字段

包含参数：

| 参数      | 说明                                                       |
| --------- | ---------------------------------------------------------- |
| header    | 表头列名，如果TableDefinition中hasHeader = false，该值无效 |
| cellStyle | 单元格风格                                                 |
| cellType  | 单元格类型                                                 |
| fieldName | 数据引用字段名称                                           |

内置有如下实现类：

+ BasicColumnDefinition

默认单元格格式为加边框，文字居中，单元格类型为STRING

+ DateTimeColumnDefinition

默认单元格格式为加边框，文字居中，单元格类型为NUMERIC

- NumberColumnDefinition

默认单元格格式为加边框，文字居中，单元格类型为NUMERIC

## 导入数据到Excel

使用`excel-io`导入customer数据，表格左上角位置在第2行的第1列（从0计数）

```java
Customer[] customers = new Customer[100];
TableDefinition tableDefinition = new AbstractTableDefinition() {

    @Override
    public int row() {
        return 2;
    }

    @Override
    public int column() {
        return 1;
    }

    @Override
    public ColumnDefinition[] columnDefinitions(Workbook workbook) {
		// 申明列
        return new ColumnDefinition[] {
                new BasicColumnDefinition(workbook, "姓名", "name"),
                new BasicColumnDefinition(workbook, "手机号", "mobile"),
                new DateTimeColumnDefinition(workbook, "出生日期", "birthday", "yyyy-MM-dd"),
                new NumberColumnDefinition(workbook, "金额", "money", "￥0.00")
        };
    }
};

ExcelIO.writer(ExcelType.XLSX, "E:\\test.xlsx")
        .fillData(customers, tableDefinition)
        .write();
```



![](doc-images/image1.png)

## 从Excel导出数据

使用`excel-io`导出customer数据

```java
List<Customer> list =  ExcelIO.reader(ExcelType.XLSX, "E:\\test.xlsx")
        .readAndClose(Customer.class, null, new AbstractTableDefinition() {

            @Override
            public int row() {
                return 2;
            }

            @Override
            public int column() {
                return 1;
            }

            @Override
            public Integer sheet() {
                return 0;
            }

            @Override
            public ColumnDefinition[] columnDefinitions(Workbook workbook) {

                return new ColumnDefinition[] {
                        new BasicColumnDefinition(workbook, "姓名", "name"),
                        new BasicColumnDefinition(workbook, "手机号", "mobile"),
                        new DateTimeColumnDefinition(workbook, "出生日期", "birthday", "yyyy-MM-dd"),
                        new NumberColumnDefinition(workbook, "金额", "money", "￥0.00"),
                };
            }
        });
```


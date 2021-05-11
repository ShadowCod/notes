### POI和EasyExcel

- **应用场景**
  - 将用户信息导出称为excel表（导出数据...）
  - 将excel中的信息录入网站数据库（习题上传...）

- **流行的技术**
  - Apache POI
  - 阿里巴巴 easyExcel

#### POI

- 是apache软件基金会的开放源码库，提供API给java程序操作office的功能

- HSSF-提供读写03版excel的功能       03版最多只有65536行,后缀xls
- XSSF-提供读写07版excel的功能        07版没有行数限制,后缀xlsx
- HWPF-提供读写world的功能
- HSLF-提供读写ppt的功能
- HDGF-提供读写visio的功能

#### easyexcel

- 阿里巴巴开源的一个excel处理框架，==使用简单、节省内存==

- 官方文档：https://www.yuque.com/easyexcel/doc/easyexcel

  

#### 两者的区别

- poi是将所有要导入导出的excel数据先一次性全部写入内存中，数据量大了很容易出现内存溢出的问题
- easyexcel是一行一行的读写

### poi使用

- 导入依赖

  ```java
  <dependencies>
      <!--xls(03)-->
      <dependency>
      	<groupId>org.apache.poi</groupId>
      	<artifactId>poi</artifactId>
      	<version>3.9</version>
      </dependency>
      
      <!--xls(07)-->
      <dependency>
      	<groupId>org.apache.poi</groupId>
      	<artifactId>poi-ooxml</artifactId>
      	<version>3.9</version>
      </dependency>
      
      <!--日期格式化工具-->
      <dependency>
      	<groupId>joda-time</groupId>
      	<artifactId>joda-time</artifactId>
      	<version>2.10.1</version>
      </dependency>
      
      <!--test-->
      <dependency>
      	<groupId>junit</groupId>
      	<artifactId>junit</artifactId>
      	<version>4.12</version>
      </dependency>
  </dependencies>    
  ```

- excel具体的对象：工作簿、工作表、行、列、单元格

- 基本写操作

  ```java
  //03和07的区别：使用的对象（HSSFWorkbook）(XSSFWorkbook)不一样，excel后缀(xls)(xlsx)不一样
  package com.poi
  import org.apache.poi.hssf.usermodel.HSSFWorkbook;
  import org.apache.poi.ss.usermodel.Row;
  import org.apache.poi.ss.usermodel.Sheet;
  import org.apache.poi.ss.usermodel.Workbook;
  import org.apache.poi.xssf.usermodel.XSSFWorkbook;
  public class excelWriteTest{
  	public void testWrite03(){
          //1.创建工作簿对象
          Workbook workbook = new HSSFWorkbook();
          //2.创建工作表对象
          Sheet sheet = workbook.createSheet("表名");
          //3.创建行
          Row row = sheet.createRow(0);
          //4.创建列
          Cell cell = row.createCell(0);
          cell.setCellValue("第一行第一列");
          //excel保存路径
          String path='';
          //最后生成一张表，使用到IO流
          FileOutputStream fos = new FileOutputStream(path+"excel名.xls");
          //输出
          worlbook.write(fos);
          //关闭流
          fos.close();
      }
      public void testWrite07(){
          //1.创建工作簿对象
          Workbook workbook = new XSSFWorkbook();
          //2.创建工作表对象
          Sheet sheet = workbook.createSheet("表名");
          //3.创建行
          Row row = sheet.createRow(0);
          //4.创建列
          Cell cell = row.createCell(0);
          cell.setCellValue("第一行第一列");
          //excel保存路径
          String path='';
          //最后生成一张表，使用到IO流
          FileOutputStream fos = new FileOutputStream(path+"excel名.xlsx");
          //输出
          worlbook.write(fos);
          //关闭流
          fos.close();
      }  
  }
  ```

  

- 大文件写操作

  - 03版缺点：最多只能处理65536行，多了会抛出异常
  - 03版优点：过程中写入缓存中，不操作磁盘，最后一次性写入磁盘，速度快

  ```java
  package com.poi
  import org.apache.poi.hssf.usermodel.HSSFWorkbook;
  import org.apache.poi.ss.usermodel.Row;
  import org.apache.poi.ss.usermodel.Sheet;
  import org.apache.poi.ss.usermodel.Workbook;
  public class excelWriteTest{
  	public void testWrite03BigData(){
          //1.创建工作簿对象
          Workbook workbook = new HSSFWorkbook();
          //2.创建工作表对象
          Sheet sheet = workbook.createSheet("表名");
          for(rowNum=0;rowNum<=65536;rowNum++){
              Row row = sheet.createRow(rowNum);
              for(cellNum=0;cellNum<=10;cellNum++){
                  Cell cell = row.createCell(cellNum);
                  cell.setCellValue(cellNum);
              }
          }
          //excel保存路径
          String path='';
          //最后生成一张表，使用到IO流
          FileOutputStream fos = new FileOutputStream(path+"excel名.xlsx");
          //输出
          worlbook.write(fos);
          //关闭流
          fos.close();
      }  
  }
  //耗时：1.2S
  ```

  - 07版缺点：写数据时速度非常慢，非常耗内存，数量大了也会发生内存溢出

  - 07版优点：可以写较大的数据量

  ```java
  package com.poi
  import org.apache.poi.xssf.usermodel.XSSFWorkbook;
  import org.apache.poi.ss.usermodel.Row;
  import org.apache.poi.ss.usermodel.Sheet;
  import org.apache.poi.ss.usermodel.Workbook;
  public class excelWriteTest{
  	public void testWrite03BigData(){
          //1.创建工作簿对象
          Workbook workbook = new XSSFWorkbook();
          //2.创建工作表对象
          Sheet sheet = workbook.createSheet("表名");
          for(rowNum=0;rowNum<=65535;rowNum++){
              Row row = sheet.createRow(rowNum);
              for(cellNum=0;cellNum<=10;cellNum++){
                  Cell cell = row.createCell(cellNum);
                  cell.setCellValue(cellNum);
              }
          }
          //excel保存路径
          String path='';
          //最后生成一张表，使用到IO流
          FileOutputStream fos = new FileOutputStream(path+"excel名.xls");
          //输出
          worlbook.write(fos);
          //关闭流
          fos.close();
      }  
  }
  //耗时：7.2s
  ```

  - 07优化版：可以写大量的数据，写数据快，占用更少的内存
  - 注意：过程中会产出临时文件，需要清理临时文件

  ```java
  package com.poi
  import org.apache.poi.sxssf.usermodel.SXSSFWorkbook;
  import org.apache.poi.ss.usermodel.Row;
  import org.apache.poi.ss.usermodel.Sheet;
  import org.apache.poi.ss.usermodel.Workbook;
  public class excelWriteTest{
  	public void testWrite03BigData(){
          //1.创建工作簿对象
          Workbook workbook = new SXSSFWorkbook();
          //2.创建工作表对象
          Sheet sheet = workbook.createSheet("表名");
          for(rowNum=0;rowNum<=65535;rowNum++){
              Row row = sheet.createRow(rowNum);
              for(cellNum=0;cellNum<=10;cellNum++){
                  Cell cell = row.createCell(cellNum);
                  cell.setCellValue(cellNum);
              }
          }
          //excel保存路径
          String path='';
          //最后生成一张表，使用到IO流
          FileOutputStream fos = new FileOutputStream(path+"excel名.xls");
          //输出
          worlbook.write(fos);
          //关闭流
          fos.close();
          //清理临时文件
          ((SXSSFWrokbook) workbook).dispose()
      }  
  }
  //耗时：1.43s
  ```

  

- 基本读操作

  ```java
  package com.poi
  import org.apache.poi.hssf.usermodel.HSSFWorkbook;
  import org.apache.poi.ss.usermodel.Row;
  import org.apache.poi.ss.usermodel.Sheet;
  import org.apache.poi.ss.usermodel.Workbook;
  import org.apache.poi.xssf.usermodel.XSSFWorkbook;
  public class excelReadTest{
  	public void testread03(){
          //excel路径
          String PATH = "D:\\poi"
          //获取文件流
          FileInputStream input = new FileInputStream(PATH+"excel名称.xls")
          //1.获取工作簿对象
          Workbook workbook = new HSSFWorkbook(input);
          //2.获取表(根据表的索引读取)
          Sheet sheet=workbook.getSheetAt(0);
          //3.获取行
          Row row=workbook.getRow(0);
          //3.获取列
          Cell cell = row.getCell(0);
          //获取单元格内容（注意：不同类型需要使用到不同的方法）
          System.out.println(cell.getStringCellValue());//获取string类型
          System.out.println(cell.getNumbericCellValue());//获取数值类型
          //关闭流
          input.close();
      }
      
      public void testread07(){
          //excel路径
          String PATH = "D:\\poi"
          //获取文件流
          FileInputStream input = new FileInputStream(PATH+"excel名称.xls")
          //1.获取工作簿对象
          Workbook workbook = new XSSFWorkbook(input);
          //2.获取表
          Sheet sheet=workbook.getSheetAt(0);
          //3.获取行
          Row row=workbook.getRow(0);
          //3.获取列
          Cell cell = row.getCell(0);
          //获取单元格内容（注意：不同类型需要使用到不同的方法）
          System.out.println(cell.getStringCellValue());//获取string类型
          System.out.println(cell.getNumbericCellValue());//获取数值类型
          //关闭流
          input.close();
      }
      
  }
  ```

  

- 读取不同的类型

  ```java
  package com.poi
  import org.apache.poi.hssf.usermodel.HSSFWorkbook;
  import org.apache.poi.ss.usermodel.Row;
  import org.apache.poi.ss.usermodel.Sheet;
  import org.apache.poi.ss.usermodel.Workbook;
  public class excelReadTest{
  	public void testread03(){
          //excel路径
          String PATH = "D:\\poi"
          //获取文件流
          FileInputStream input = new FileInputStream(PATH+"excel名称.xls")
          //1.获取工作簿对象
          Workbook workbook = new HSSFWorkbook(input);
          //2.获取表(根据表的索引读取)
          Sheet sheet=workbook.getSheetAt(0);
          //3.获取行
          Row row=workbook.getRow(0);
          //获取行标题内容
          if(row !=null){
              //获取一行中一共有多少列
              int count = row.getPhysicalNumberOfCells();
              //遍历输出单元格内容
              for(int cellNum = 0;cellNum<count;cellNum++){
                  //3.获取列
          		Cell cell = row.getCell(cellNum);
                  if(cell!=null){
                      //获取值的类型
                      int type = cell.getCellType();
                      System.out.println(cell.getStringCellValue());//获取string类型
                  }
              }
          }
          //获取其他行中的内容
          int countRow = sheet.getPhysicalNumberRows();
          for(int rowNum=1;rowNum<countRow;rowNum++){
             //获取指定的行
              Row row = sheet.getRow(rowNum);
              if(row!=null){
                  //获取改行有多少列
                  int count = row.getPhysicalNumberCells();
                  //遍历输出单元格内容
              	for(int cellNum = 0;cellNum<count;cellNum++){
                  	//3.获取列
          			Cell cell = row.getCell(cellNum);
                  	if(cell!=null){
                      //获取值的类型
                      int type = cell.getCellType();
                      String cellValue = '';
                      //使用switch根据不同的类型进行处理
                          switch (type){
                              case HSSFCell.CELL_TYPE_STRING://字符串
                                  cellValue=cell.getStringCellValue();
                                  break;
                              case HSSFCell.CELL_TYPE_BOOLEAN://布尔
                                  cellValue=String.valueOf(cell.getBooleanCellValue());
                                  break;
                               case HSSFCell.CELL_TYPE_BLANK://空
                                  break;
                               case HSSFCell.CELL_TYPE_NUMBERIC://数字（日期和普通数字）
                                  if(HSSFDateUtil.isCellDateFormatted(cell)){//日期
                                      Date date=cell.getDateCellValue();
                                      cellValue=new DateTime(date).toString("yyy-mm-dd")
                                  }else{
                                      //防止数字过长
                                      cell.setCellType(HSSFCell.CELL_TYPE_STRING);
                                      cellValue=cell.toString();
                                  }
                                  break;
                               case HSSFCell.CELL_TYPE_ERROR://错误类型
                                  break;   
                          }
                      }
                  }
              }
          }
          //关闭流
          input.close();
      }
      
  }
  ```

  

- 公式

  ```java
  package com.poi
  import org.apache.poi.hssf.usermodel.HSSFWorkbook;
  import org.apache.poi.ss.usermodel.Row;
  import org.apache.poi.ss.usermodel.Sheet;
  import org.apache.poi.ss.usermodel.Workbook;
  public class excelReadTest{
  	public void testread03(){
          //excel路径
          String PATH = "D:\\poi"
          //获取文件流
          FileInputStream input = new FileInputStream(PATH+"excel名称.xls")
          //1.获取工作簿对象
          Workbook workbook = new HSSFWorkbook(input);
          //2.获取表(根据表的索引读取)
          Sheet sheet=workbook.getSheetAt(0);
          //3.获取行
          Row row=workbook.getRow(0);
          //4.获取列
          Cell cell=row.getCell(0);
          //5.获取表中计算公式
          FormulaEvaluator fe = new HSSFFormulaEvaluator((HSSFWorkbook)workbook);
          //6.获取单元格内容类型
          int type=cell.getCellType();
          switch(type){
                  case HSSFCell.CELL_TYPE_FORMULA://公式
                  //进行公式计算
                  CellValue evaluate = fe.evaluate(cell);
                  String cellValue=evaluate.formatAsString();
                  break;
          	}
          }
          //关闭流
          input.close();
      }
      
  }
  ```

  

### easyExcel使用

- 导入依赖

  ```java
  <dependencies>
      <!--xls(03)-->
      <dependency>
      	<groupId>com.alibaba</groupId>
      	<artifactId>easyexcek</artifactId>
      	<version>2.2.0-beta2</version>
      </dependency>
      
      <!--日期格式化工具-->
      <dependency>
      	<groupId>joda-time</groupId>
      	<artifactId>joda-time</artifactId>
      	<version>2.10.1</version>
      </dependency>
      
      <!--test-->
      <dependency>
      	<groupId>junit</groupId>
      	<artifactId>junit</artifactId>
      	<version>4.12</version>
      </dependency>
  </dependencies>   
  ```

  

- 具体操作看文档：https://www.yuque.com/easyexcel/doc/easyexcel
## 前言
本篇博客记录着在javaweb中批量导入Excel文件和导出模板

## 一、 导入Excel文件

### 1. 导入poi相关坐标

```xml
	<dependency>
		<groupId>org.apache.poi</groupId>
		<artifactId>poi</artifactId>
		<version>3.17</version>
	</dependency>
	<dependency>
		<groupId>org.apache.poi</groupId>
		<artifactId>poi-ooxml</artifactId>
		<version>3.17</version>
	</dependency>	
	<dependency>
		<groupId>org.apache.poi</groupId>
		<artifactId>poi-ooxml-schemas</artifactId>
		<version>3.17</version>
	</dependency>						
```
### 2. 工具类

```java
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.InputStream;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class ImportExcel {
    // abc.xls
    public static boolean isXls(String fileName){
        // (?i)忽略大小写
        if(fileName.matches("^.+\\.(?i)(xls)$")){
            return true;
        }else if(fileName.matches("^.+\\.(?i)(xlsx)$")){
            return false;
        }else{
            throw new RuntimeException("格式不对");
        }
    }

    public static List<Map<String, Object>> readExcel(String fileName, InputStream inputStream) throws Exception{
        
        boolean ret = isXls(fileName);
        Workbook workbook = null;
        // 根据后缀创建不同的对象
        if(ret){
            workbook = new HSSFWorkbook(inputStream);
        }else{
            workbook = new XSSFWorkbook(inputStream);
        }
        Sheet sheet = workbook.getSheetAt(0);
        // 得到标题行
        Row titleRow = sheet.getRow(0);
        
        int lastRowNum = sheet.getLastRowNum();
        int lastCellNum = titleRow.getLastCellNum();
        
        List<Map<String, Object>> list = new ArrayList<>();
        
        for(int i = 1; i <= lastRowNum; i++ ){
            Map<String, Object> map = new HashMap<>();
            Row row = sheet.getRow(i);
            for(int j = 0; j < lastCellNum; j++){
                // 得到列名
                String key = titleRow.getCell(j).getStringCellValue();
                Cell cell = row.getCell(j);
                cell.setCellType(CellType.STRING);
                
                map.put(key, cell.getStringCellValue());
            }
            list.add(map);
        }
        workbook.close();
        return list;
    
    }
}

```
### 3. controller层

```java

	@RequestMapping("/staff/import")
    @ResponseBody
    public String importExcel(@RequestParam MultipartFile mFile){
        try {
            String fileName = mFile.getOriginalFilename();
            // 获取上传文件的输入流
            InputStream inputStream = mFile.getInputStream();
            // 调用工具类中方法，读取excel文件中数据
            List<Map<String, Object>> sourceList = ImportExcel.readExcel(fileName, inputStream);

            // 将对象先转为json格式字符串，然后再转为List<SysUser> 对象
            ObjectMapper objMapper = new ObjectMapper();
            String infos = objMapper.writeValueAsString(sourceList);

            // json字符串转对象
            // Staff是我们写的成员实体类po
            List<Staff> staffList = objMapper.readValue(infos, new TypeReference<List<Staff>>() {});

            // 批量添加
            // 这是我们写的批量添加成员的服务类
            staffService.addStaffBatch(staffList);

            return "操作成功";

        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();

            return "操作失败";
        }

    }

```
### 4. CommonsMultipartResolver类配置

```xml

<!-- 文件上传的解析器  id的值不能改-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 上传文件的最大大小 ，单位字节 ，比如 1024 * 1024 = 1M-->
    <property name="maxUploadSize" value="1048576"></property>

</bean>

```
### 5. 批量插入mapper

```xml

  <insert id="addStaffBatch" parameterType="java.util.List" >
    insert into t_staff
    (
      username,phone_number,type,create_user,create_date,
      modify_user,modify_date
     )
     values   
    <foreach collection="staffList" item="staff" index="index" separator=",">
      (
		 #{staff.username,jdbcType=VARCHAR},
	     #{staff.phoneNumber,jdbcType=VARCHAR},
	     #{staff.type,jdbcType=INTEGER},
	     #{staff.createUser,jdbcType=INTEGER},
	     #{staff.createDate,jdbcType=TIMESTAMP},
	     #{staff.modifyUser,jdbcType=INTEGER},
	     #{staff.modifyDate,jdbcType=TIMESTAMP}       
	  )
    </foreach>
  </insert>
```
### 6. jsp代码

```jsp

<form id="importForm" method="post" action="/staff/import" enctype="multipart/form-data">
    <input type="file" class="btn btn-primary" size="60" name="xlsFile" id="xlsFile"/>
    <input type="submit" class="btn btn-primary fa fa-search"  size="60" value="批量导入"/>
</form>
```

## 二、 下载Excel模板

```java
/**
	 * 描述：下载外部案件导入模板
	 * @param response
	 * @param request
	 */
	@RequestMapping("/downloadExcel")
	@ResponseBody
	public void downloadExcel(HttpServletResponse response,HttpServletRequest request) {
		//方法一：直接下载路径下的文件模板
		try {
            //获取要下载的模板名称
            String fileName = "ApplicationImportTemplate.xlsx";
            //设置要下载的文件的名称
            response.setHeader("Content-disposition", "attachment;fileName=" + fileName);
            //通知客服文件的MIME类型
            response.setContentType("application/vnd.ms-excel;charset=UTF-8");
            //获取文件的路径
            String filePath = getClass().getResource("/template/" + fileName).getPath();
            FileInputStream input = new FileInputStream(filePath);
            OutputStream out = response.getOutputStream();
            byte[] b = new byte[2048];
            int len;
            while ((len = input.read(b)) != -1) {
                out.write(b, 0, len);
            }
            //修正 Excel在“xxx.xlsx”中发现不可读取的内容。是否恢复此工作薄的内容？如果信任此工作簿的来源，请点击"是"
            response.setHeader("Content-Length", String.valueOf(input.getChannel().size()));
            input.close();
            //return Response.ok("应用导入模板下载完成");
        } catch (Exception ex) {
            logger.error("getApplicationTemplate :", ex);
            //return Response.ok("应用导入模板下载失败！");
        }
		
		
		//方法二：可以采用POI导出excel，但是比较麻烦
		/*try {
			Workbook workbook = new HSSFWorkbook();
			request.setCharacterEncoding("UTF-8");
	        response.setCharacterEncoding("UTF-8");
	        response.setContentType("application/x-download");
	
	        
	        String filedisplay = "外部案件导入模板.xls";
	     
	        filedisplay = URLEncoder.encode(filedisplay, "UTF-8");
	        response.addHeader("Content-Disposition", "attachment;filename="+ filedisplay);
	        
			// 第二步，在webbook中添加一个sheet,对应Excel文件中的sheet  
			Sheet sheet = workbook.createSheet("外部案件导入模板");
			// 第三步，在sheet中添加表头第0行
			Row row = sheet.createRow(0);
			// 第四步，创建单元格，并设置值表头 设置表头居中 
			CellStyle style = workbook.createCellStyle();  
	        style.setAlignment(CellStyle.ALIGN_CENTER); // 创建一个居中格式 
			
			Cell cell = row.createCell(0);  
	        cell.setCellValue("商品名称");  
	        cell.setCellStyle(style); 
	        sheet.setColumnWidth(0, (25 * 256));  //设置列宽，50个字符宽
	        
	        cell = row.createCell(1);  
	        cell.setCellValue("商品编码");  
	        cell.setCellStyle(style); 
	        sheet.setColumnWidth(1, (20 * 256));  //设置列宽，50个字符宽
	        
	        cell = row.createCell(2);  
	        cell.setCellValue("商品价格");  
	        cell.setCellStyle(style);  
	        sheet.setColumnWidth(2, (15 * 256));  //设置列宽，50个字符宽
	        
	        cell = row.createCell(3);  
	        cell.setCellValue("商品规格");  
	        cell.setCellStyle(style);  
	        sheet.setColumnWidth(3, (15 * 256));  //设置列宽，50个字符宽
	        
	        // 第五步，写入实体数据 实际应用中这些数据从数据库得到
			row = sheet.createRow(1);
			row.createCell(0, Cell.CELL_TYPE_STRING).setCellValue(1);  
			row.createCell(1, Cell.CELL_TYPE_STRING).setCellValue(2); 
			row.createCell(2, Cell.CELL_TYPE_STRING).setCellValue(3);   //商品价格
			row.createCell(3, Cell.CELL_TYPE_STRING).setCellValue(4);  //规格
        
			// 第六步，将文件存到指定位置  
	        try  
	        {  
	        	OutputStream out = response.getOutputStream();
	        	workbook.write(out);
	            out.close();  
	        }  
	        catch (Exception e)  
	        {  
	            e.printStackTrace();  
	        }  
		} catch (Exception e) {
			e.printStackTrace();
		}*/
    }

```
模块位置： resources/template/ApplicationImportTemplate.xlsx



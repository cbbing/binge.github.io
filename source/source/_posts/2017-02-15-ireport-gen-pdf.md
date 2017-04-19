title: iReport一键生成PDF报告指南
date: 2017-02-15 10:10:06
category: Java
tags:
	- ireport
	- pdf
	- 报告
---


项目中需要生成PDF和Word文件的报告，文件中包含图片和表格。基于Java的解决方案有Freemarker模板引擎，是通过XML文件将填入的内容放上\${}占位符。这种方式对于简单的文本是没问题，但如果占位符中有字符跟\${}冲突，就比较难处理了。

iReport是一款可视化报表设计工具，看软件界面跟Qt的风格有几分相似，内置丰富的图表，能够创建复杂的报表。相比XML模板的方式，更加灵活和稳定。
- 主界面
<!-- more -->
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport1.png)

---

本文以一个实际的导出PDF报告案例来讲解。

# 一、ireport可视化布局
## 1，插入图片
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport2.png)

从组件面板拖动Image到中间的页面Designer区域。
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport3.png)

在右下角的Image属性列表中，设置Image Expression为$F{netImage}。

不过，首先得在Fields定义好变量：netImage，netImage可以在Java工程中传值传过来，也可以是静态图片或url。
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport4.png)


**Image Expression支持的三种方式：**
1. $F{netImage};
2. "D://net.png"
3. "https://dn-abc.qbox.me/logo.png"

生成的pdf效果：
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport5.png)


## 2，插入表格
为了便于扩展，表格放到Subreport中；在组件面板中新建一个Subreport，进入subreport设计表格；
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport6.png)
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport7.png)


这里有几个要点：
### a, 中文字符的显示：
需要在属性中设置
```
Pdf Font: STSong-Light
Pdf Embedded: 勾选
Pdf Encoding: UniGB-UCS2-H(Chinese Simplified)
```
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport8.png)

### b, 数字的格式化
由于Java传过来的值在ireport中默认都是String，在Expression Class中设置为java.lang.Double也没有用。
需要在Text Field Expression中将字符串转成Double，然后在格式化，Pattern中设置“###0.0000”，表示保留4为小数。
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport9.png)

### c, 数字百分比的显示
```
Text Field Expression: ($F{val}.equals("") ? "--" : new Double($F{val}))
Pattern：#,##0.00%
```
如果val为空，则用显示--。按百分比格式化，保留2位小数。
效果如下：
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport10.png)
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport11.png)



## 3，生成编译文件
ireport源文件是jrxml格式，点击下图中的锤子，编译后生成jasper文件。
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport12.png)

# 二、Java项目配置
将生成的jasper文件导入到java工程
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-15/ireport13.png)

## 1，生成图片
项目中的方案是从echarts中截图，将截图保存到本地文件夹
```
/**
* 调用js
* 产生图片
*/
function genPic(callback){
    var data = "pic="+encodeURIComponent(mainChart.getDataURL(
    {
       type:"png",
       pixelRatio:1, 
       excludeComponents:['toolbox', 'dataZoom']
    }));  
            
    var xmlhttp;          
    if (window.XMLHttpRequest) { // code for IE7+, Firefox, Chrome, Opera, Safari  
        xmlhttp = new XMLHttpRequest();  
    } else { // code for IE6, IE5  
        xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");  
    }  
            
    xmlhttp.open("POST",ctx+"/productReport/genNetPic?fundId="+$('#netfundId').val(),true);  
    xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");  
    xmlhttp.onreadystatechange = function() {  
        if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {  
            // 回调生成结果
            callback(true);  
        }  else if (xmlhttp.status!=200){
            callback(false); 
        }
    }  
    xmlhttp.send(data); 
}
```

Controller中的函数：
```
@RequestMapping(value = "genNetPic", method = RequestMethod.POST)
@ResponseBody
public BaseResponse genNetPic(HttpServletRequest request){
    logger.info("开始生成图片");
    String pic = request.getParameter("pic"); 
    String fundId = request.getParameter("fundId"); 
    BaseResponse baseResponse = null;
    try{
        String[] url = pic.split(",");
        String u = url[1];  
        byte[] b = new BASE64Decoder().decodeBuffer(u);  
        // 创建图片生成的路径:日期+产品id
        Date curDate = new Date();
        StringBuilder filePath = new StringBuilder("");
        filePath.append(upload_path).append(DateUtil.formatDate(curDate, "yyyy-MM-dd")).append("/").append(fundId);
        FileUtil.createDictionary(filePath.toString());
        // 创建图片文件
        OutputStream out = new FileOutputStream(new File(filePath.toString()+"/net.png"));  
                    out.write(b);  
                    out.flush();  
                    out.close();
                    baseResponse = new BaseResponse(true);
                    logger.info("完成生成图片");
    } catch(Exception e){
        logger.error("生成图片失败，失败原因:{}",e.getMessage());
        baseResponse = new BaseResponse(false, "图片生成失败");
    }
    return baseResponse;
}
```

## 2，构建报告参数
```
/**
* 构建报表的参数
* @return
*/
private FundReportDto getReportParams(String fundId, String fundName, Date curDate){
    String dateStr = DateUtil.formatDate(curDate, "yyyy-MM-dd");
    //
    FundReportDto fundReportDto = new FundReportDto();
    // 得到图片路径
    StringBuilder netImage = new StringBuilder(jasper_chart_path);
    netImage.append(dateStr).append("/").append(fundId).append("/").append("net.png");
    fundReportDto.setNetImage(netImage.toString());
    ...

    return fundReportDto;
}
```
返回参数用对象FundReportDto封装，FundReportDto是可序列化的对象：
```
import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;

public class FundReportDto implements Serializable{
	/**
	 * 
	 */
	private static final long serialVersionUID = -3023588547102105811L;
	
	/**
	 * 统计日期
	 */
	private String statisticDate;
        /*
	 * 图片
	 */
	private String netImage;
       
       ...
  }
```

## 3，生成报告
报告支持pdf和word两种格式，生成步骤是先将报告保存在本地文件夹，然后调用浏览器下载。

```
/**
* 一建生成报告
*/
import net.sf.jasperreports.engine.JRExporterParameter;
import net.sf.jasperreports.engine.JasperExportManager;
import net.sf.jasperreports.engine.JasperFillManager;
import net.sf.jasperreports.engine.JasperPrint;
import net.sf.jasperreports.engine.JasperReport;
import net.sf.jasperreports.engine.data.JRBeanCollectionDataSource;
import net.sf.jasperreports.engine.export.JRRtfExporter;
import net.sf.jasperreports.engine.util.JRLoader;

@RequestMapping(value = "exportReport", method = RequestMethod.GET)
public void exportReport(String fundId, String fundName, String fileFormat, HttpServletResponse response){
   Date curDate = new Date();
   String dateStr = DateUtil.formatDate(curDate, "yyyy-MM-dd");
   // 文件路径 
   StringBuilder filePath = new StringBuilder("");
   filePath.append(upload_path).append(dateStr).append("/").append(fundId);
   FileUtil.createDictionary(filePath.toString());
   // 文件名 
   StringBuilder fileName = new StringBuilder("");
   fileName.append(fundName).append("评价报告_").append(dateStr).append(".").append(fileFormat);
   // 文件全路径
   StringBuilder fullName = new StringBuilder("");
   fullName.append(filePath).append("/").append(fileName);
   // 加载报表对象
   InputStream is = null;
   JasperReport jasperReport = null;
   JasperPrint jasperPrint = null;
   is = ProductDetailController.class.getClassLoader().getResourceAsStream(analysis_jasper_path);
   try{
        // 响应头部信息设置
        response.setContentType("text/plain");
        response.setHeader("Content-Disposition", "attachment; filename="+FileUtil.getAttachName(fileName.toString()));
        //
        jasperReport = (JasperReport) JRLoader.loadObject(is);
        Map<String,Object> params = new HashMap<String,Object>();
        params.put("SUBREPORT_DIR", getClassPath(subreport_dir));
        params.put("fundName", fundName);
        params.put("reportTitle", fundName+"报告");
          
        // 
        List<FundReportDto> fundReportDtos = new ArrayList<FundReportDto>();
        FundReportDto fundReportDto = getReportParams(fundId, fundName,curDate);
        fundReportDtos.add(fundReportDto);
        params.put("statisticDate", fundReportDto.getStatisticDate());
        // 
        jasperPrint = JasperFillManager.fillReport(jasperReport, params, new JRBeanCollectionDataSource(fundReportDtos));
        if (fileFormat.equals("pdf")) {
            JasperExportManager.exportReportToPdfFile(jasperPrint, fullName.toString());
        }else{
            JRRtfExporter docReport = new JRRtfExporter();
            docReport.setParameter(JRExporterParameter.OUTPUT_FILE_NAME,fullName.toString());
            docReport.setParameter(JRExporterParameter.JASPER_PRINT, jasperPrint);
            docReport.exportReport();
        }
        
        FileUtil.download(fullName.toString(), response.getOutputStream());
                
    } catch(Exception e){
        e.printStackTrace();
        logger.error("生成报告失败，失败原因:{}",e.getMessage());
    } finally{
        try{
            if (is!=null) is.close();
        }catch(Exception e){
            logger.error("关闭文件输入流异常:{}",e.getMessage());
        }
    }
        
}
```
其中analysis_jasper_path为jasper文件的路径。
这样，一份完整的pdf或word报告就制作出来了。



- 相关资料
   - [ireport5.6 windows安装包](http://kekefund.com/soft/iReport-5.6.0.zip)
   - [ireport用户手册](http://kekefund.com/soft/ireport-ultimate-guide.pdf)

- 参考:
> http://blog.csdn.net/jackfrued/article/details/39449021
> http://blog.csdn.net/wlwlwlwl015/article/details/51312853
> http://zxs19861202.iteye.com/blog/1171118


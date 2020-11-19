## 前言
本文为记录XML和Map相互转换的工具类

## 导入依赖
```
<dependency>
   <groupId>org.dom4j</groupId>
      <artifactId>dom4j</artifactId>
   <version>2.0.0</version>         
</dependency>
```

## 工具类

```java
package com.unicom.admin.util;

import java.io.StringReader;
import java.io.StringWriter;
import java.util.*;

import org.dom4j.*;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.SAXReader;
import org.dom4j.io.XMLWriter;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.alibaba.fastjson.JSONObject;

/**
 * 
 * @ClassName: CommonUtils
 * @Description: 公共工具类
 * @author: dave
 * @date: 2020年8月5日 下午1:49:03
 */
public class CommonUtils {
	
	private static final Logger logger = LoggerFactory.getLogger(CommonUtils.class);

	/**
	 * 
	 * @Title:        isWeekend 
	 * @Description:  判断当前时间是否为周末 
	 * @param:        @return    
	 * @return:       boolean    
	 * @author        dave
	 * @date          2020年8月5日 下午1:51:16
	 */
	public static boolean isWeekend() {	
		Calendar today = Calendar.getInstance();
		if(today.get(Calendar.DAY_OF_WEEK) == Calendar.SATURDAY || today.get(Calendar.DAY_OF_WEEK) == Calendar.SUNDAY ){
			return true;
		}else{
			return false;
		}
			
	}

    /**
     * (多层)xml格式字符串转换为map
     *
     * @param xml xml字符串
     * @return 第一个为Root节点，Root节点之后为Root的元素，如果为多层，可以通过key获取下一层Map
     */
    public static Map<String, Object> multilayerXmlToMap(String xml) {
        Document doc = null;
        try {
            doc = DocumentHelper.parseText(xml);
        } catch (DocumentException e) {
            logger.error("xml字符串解析，失败 --> {}", e);
        }
        Map<String, Object> map = new HashMap<>();
        if (null == doc) {
            return map;
        }
        // 获取根元素
        Element rootElement = doc.getRootElement();
        recursionXmlToMap(rootElement,map);
        return map;
    }

    /**
     * multilayerXmlToMap核心方法，递归调用，不能解决节点元素重合问题
     *
     * @param element 节点元素
     * @param outmap 用于存储xml数据的map
     */
    private static void recursionXmlToMap2(Element element, Map<String, Object> outmap) {
        // 得到根元素下的子元素列表
        List<Element> list = element.elements();
        int size = list.size();
        if (size == 0) {
            // 如果没有子元素,则将其存储进map中
            outmap.put(element.getName(), element.getTextTrim());
        } else {
            // innermap用于存储子元素的属性名和属性值
            Map<String, Object> innermap = new HashMap<>();
            // 遍历子元素
            list.forEach(childElement -> recursionXmlToMap2(childElement, innermap));
            outmap.put(element.getName(), innermap);
        }
    }

	/**
	 * @Title:        recursionXmlToMap
	 * @Description:   multilayerXmlToMap核心方法，递归调用，可以解决节点元素重合问题
	 * @param elmt
	 * @param map
	 * @return:       void
	 * @author        dave
	 * @date          2020/10/12 11:02
	 */
	private static void recursionXmlToMap(Element elmt, Map<String, Object> map) {
		if (null == elmt) {
			return;
		}
		String name = elmt.getName();
		// 当前元素是最小元素
		if (elmt.isTextOnly()) {
			// 查看map中是否已经有当前节点
			Object f = map.get(name);
			// 用于存放元素属性
			Map<String, Object> m = new HashMap<String, Object>();
			// 遍历元素中的属性
			Iterator ai = elmt.attributeIterator();
			// 用于第一次获取该元素数据
			boolean aiHasNex = false;
			while (ai.hasNext()) {
				aiHasNex = true;
				// 拿到属性值
				Attribute next = (Attribute) ai.next();
				m.put(name + "." + next.getName(), next.getValue());
			}
			// 第一次获取该元素
			if (f == null) {
				// 判断如果有属性
				if (aiHasNex) {
					// 将属性map存入解析map中
					m.put(name, elmt.getText());
					map.put(name, m);
				} else {
					// 没有属性，直接存入相应的值
					map.put(name, elmt.getText());
				}
			} else {
				// 解析map中已经有相同的节点
				// 如果当前值是list
				if (f instanceof List<?>) {
					// list中添加此元素
					m.put(name, elmt.getText());
					((List) f).add(m);
				} else {
					// 如果不是，说明解析map中只存在一个与此元素名相同的对象
					// 存放元素
					List<Object> listSub = new ArrayList<Object>();
					// 如果解析map中的值为string，说明第一个元素没有属性
					if (f instanceof String) {
						// 转换为map对象，
						Map<String, Object> m1 = new HashMap<String, Object>();
						m1.put(name, f);
						// 添加到list中
						listSub.add(m1);
					} else {
						// 否则直接添加值
						listSub.add(f);
					}
					// 将当前的值包含的属性值放入list中
					m.put(name, elmt.getText());
					listSub.add(m);
					// 解析map中存入list
					map.put(name, listSub);
				}

			}
		} else {
			// 存放子节点元素
			Map<String, Object> mapSub = new HashMap<String, Object>();
			// 遍历当前元素的属性存入子节点map中
			attributeIterator(elmt, mapSub);
			// 获取所有子节点
			List<Element> elements = (List<Element>) elmt.elements();
			// 遍历子节点
			for (Element elmtSub : elements) {
				// 递归调用转换map
				recursionXmlToMap(elmtSub, mapSub);
			}
			// 当前元素没有子节点后 获取当前map中的元素名所对应的值
			Object first = map.get(name);
			if (null == first) {
				// 如果没有将值存入map中
				map.put(name, mapSub);
			} else {
				// 如果有，则为数组对象
				if (first instanceof List<?>) {
					attributeIterator(elmt, mapSub);
					((List) first).add(mapSub);
				} else {
					List<Object> listSub = new ArrayList<Object>();
					listSub.add(first);
					attributeIterator(elmt, mapSub);
					listSub.add(mapSub);
					map.put(name, listSub);
				}
			}
		}
	}

	private static void attributeIterator(Element elmt, Map<String, Object> map) {
		if (elmt != null) {
			Iterator ai = elmt.attributeIterator();
			while (ai.hasNext()) {
				Attribute next = (Attribute) ai.next();
				map.put(elmt.getName() + "." + next.getName(), next.getValue());
			}
		}
	}


	/**
     * (多层)map转换为xml格式字符串
     *
     * @param map 需要转换为xml的map
     * @param isCDATA 是否加入CDATA标识符 true:加入 false:不加入
     * @return xml字符串
     */
    public static String multilayerMapToXml(Map<String, Object> map, boolean isCDATA){
        String parentName = "UNI_ECB_INFO_INPUT";
        Document doc = DocumentHelper.createDocument();
        doc.addElement(parentName);
        String xml = recursionMapToXml(doc.getRootElement(), parentName, map, isCDATA);
        return formatXML(xml);
    }

    /**
     * multilayerMapToXml核心方法，递归调用
     *
     * @param element 节点元素
     * @param parentName 根元素属性名
     * @param map 需要转换为xml的map
     * @param isCDATA 是否加入CDATA标识符 true:加入 false:不加入
     * @return xml字符串
     */
    @SuppressWarnings("unchecked")
    private static String recursionMapToXml(Element element, String parentName, Map<String, Object> map, boolean isCDATA) {
        Element xmlElement = element.addElement(parentName);
        map.keySet().forEach(key -> {
            Object obj = map.get(key);
            if (obj instanceof Map) {
                recursionMapToXml(xmlElement, key, (Map<String, Object>)obj, isCDATA);
            } else {
                String value = obj == null ? "" : obj.toString();
                if (isCDATA) {
                    xmlElement.addElement(key).addCDATA(value);
                } else {
                    xmlElement.addElement(key).addText(value);
                }
            }
        });
        return xmlElement.asXML();
    }

    /**
     * 格式化xml,显示为容易看的XML格式
     *
     * @param xml 需要格式化的xml字符串
     * @return
     */
    public static String formatXML(String xml) {
        String requestXML = null;
        try {
            // 拿取解析器
            SAXReader reader = new SAXReader();
            Document document = reader.read(new StringReader(xml));
            if (null != document) {
                StringWriter stringWriter = new StringWriter();
                // 格式化,每一级前的空格
                OutputFormat format = new OutputFormat("    ", true);
                // xml声明与内容是否添加空行
                format.setNewLineAfterDeclaration(false);
                // 是否设置xml声明头部
                format.setSuppressDeclaration(false);
                // 是否分行
                format.setNewlines(true);
                XMLWriter writer = new XMLWriter(stringWriter, format);
                writer.write(document);
                writer.flush();
                writer.close();
                requestXML = stringWriter.getBuffer().toString();
            }
            return requestXML;
        } catch (Exception e) {
            logger.error("格式化xml，失败 --> {}", e);
            return null;
        }
    }
	
```
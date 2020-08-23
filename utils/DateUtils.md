## 前言
本篇博客记录着工作中常用的 date 工具类

## dateUtils

```java
package com.unicom.common.utils;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

/**
 * 日期时间相关工具
 * @author Louis
 * @date Sep 23, 2018
 */
public class DateTimeUtils {

	/**
	 * 标准格式
	 */
	public static final String DATE_FORMAT = "yyyy-MM-dd HH:mm:ss";
	
	/**
	 * 短时间字符串格式
	 */
	public static final String DATE_FORMAT_SHORT = "yyyy-MM-dd";
	/**
	 * 没有格式的日期
	 */
	public static final String DATE_FORMAT_FORMLESS = "yyyyMMddHHmmss";
	/**
	 * 精确到毫秒的格式
	 */
	public static final String DATE_FORMAT_MILLI_SECOND = "yyyyMMddHHmmssSSS";
	/**
	 * 只有时钟和分钟
	 */
	public static final String DATE_FORMAT_HHMM = "HHmm";
	/**
	 * 只有年月
	 */
	public static final String DATE_FORMAT_YYYYMM = "yyyyMM";
	/**
	 * 只有年月日
	 */
	public static final String DATE_FORMAT_YYYYMMDD = "yyyyMMdd";
	
	/**
	 * 精确到分的格式
	 */
	public static final String DATE_FORMAT_FORMINUTE = "yyyyMMddHHmm";
	
	/**
	 * 获取当前标准格式化日期时间
	 * @param date
	 * @return
	 */
	public static String getDateTime() {
		return getDateTime(new Date());
	}
	
	/**
	 * 标准格式化日期时间
	 * @param date
	 * @return
	 */
	public static String getDateTime(Date date) {
		return (new SimpleDateFormat(DATE_FORMAT)).format(date);
	}
	/**
	 * 
	 * @Title: getDateFromateTime
	 * @Description: 描述这个方法的作用
	 * @param: @param format
	 * @param: @param date
	 * @param: @return 参数说明
	 * @return: String 返回类型
	 * @throws
	 */
	public static String getDateFromateTime(String format, Date date) {
		return (new SimpleDateFormat(format)).format(date);
	}
	
	/**
	 * 
	 * @Title: getDate
	 * @Description: 字符串时间 转为Date日期
	 * @param: @param date 字符串时间
	 * @param: @param format 字符串格式
	 * @param: @return
	 * @param: @throws ParseException 参数说明
	 * @return: Date 返回类型
	 * @throws
	 */
	public static Date getDate(String date,String format) {
		if( date == null){
			return null;
		}
		SimpleDateFormat dateFormat = new SimpleDateFormat(format); 
		try {
			return  dateFormat.parse(date);
		} catch (ParseException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return null;
	}
	
	/**
	 * 
	 * @Title: getDateFrom 
	 * @Description: 距离 date多久之前 或 多久之后
	 * @param: @param date  某个日期
	 * @param: @param unit  距离某个日期 Calendar.MONTH 几个月前 Calendar.HOUR 几小时前 Calendar.DAY_OF_MONTH 几天前 Calendar.YEAR 几年前
	 * @param: @param count 小于0 代表之前 大于0 代表之后
	 * @param: @return 参数说明
	 * @return: Date 返回类型
	 * @throws
	 */
	public static Date getDateFrom(Date date,int unit, int count) {  
		Calendar calendar = Calendar.getInstance();
		if(date==null) date = new Date();
        calendar.setTime(date);
        calendar.add(unit, count);
        date = calendar.getTime();
        return date;
    }  
	
	/**
	 * 时间格式带毫秒(例如：20190514200343348)
	 * @return
	 */
	public static String getTime(){
		Date currentTime = new Date();
		SimpleDateFormat formatter = new SimpleDateFormat("yyyyMMddHHmmssSSS");
		String dateString = formatter.format(currentTime);
		return dateString;
	}
	
	/**
	 * 获取subts时间(例如：20190517200340)
	 * @return
	 */
	public static String getSubtsTime(){
		Date currentTime = new Date();
		SimpleDateFormat formatter = new SimpleDateFormat("yyyyMMddHHmmss");
		String dateString = formatter.format(currentTime);
		return dateString;
	}

	/**
	 * 时间反格式化
	 * @return
	 */
	public static Date dateParse() throws ParseException {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddHHmmss");
        return sdf.parse("20190517200340");
	}

	 /** 
     * 两个时间之间相差距离多少天 
     * @param one 时间参数 1： 
     * @param two 时间参数 2： 
     * @return 相差天数 
     */ 
        public static long getDistanceDays(String starttime, String endtime) throws Exception{  
            DateFormat df = new SimpleDateFormat("yyyy-MM-dd");  
            Date one;  
            Date two;  
            long days=0;  
            try {  
                one = df.parse(starttime);  
                two = df.parse(endtime);  
                long time1 = one.getTime();  
                long time2 = two.getTime();  
                long diff ;  
                if(time1<time2) {  
                    diff = time2 - time1;  
                } else {  
                    diff = time1 - time2;  
                }  
                days = diff / (1000 * 60 * 60 * 24);  
            } catch (ParseException e) {  
                e.printStackTrace();  
            }  
            return days;//返回相差多少天  
        }  
          
        
        /** 
         * 两个时间相差距离多少天多少小时多少分多少秒 
         * @param str1 时间参数 1 格式：1990-01-01 12:00:00 
         * @param str2 时间参数 2 格式：2009-01-01 12:00:00 
         * @return long[] 返回值为：{天, 时, 分, 秒} 
         */  
        public static long[] getDistanceTimes(String starttime, String endtime) {  
            DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
            Date one;  
            Date two;  
            long day = 0;  
            long hour = 0;  
            long min = 0;  
            long sec = 0;  
            try {  
                one = df.parse(starttime);  
                two = df.parse(endtime);  
                long time1 = one.getTime();  
                long time2 = two.getTime();  
                long diff ;  
                if(time1<time2) {  
                    diff = time2 - time1;  
                } else {  
                    diff = time1 - time2;  
                }  
                day = diff / (24 * 60 * 60 * 1000);  
                hour = (diff / (60 * 60 * 1000) - day * 24);  
                min = ((diff / (60 * 1000)) - day * 24 * 60 - hour * 60);  
                sec = (diff/1000-day*24*60*60-hour*60*60-min*60);  
            } catch (ParseException e) {  
                e.printStackTrace();  
            }  
            long[] times = {day, hour, min, sec};  
            return times;  
        }  
        
        
        
        /** 
         * 两个时间相差距离多少天多少小时多少分多少秒 
         * @param str1 时间参数 1 格式：1990-01-01 12:00:00 
         * @param str2 时间参数 2 格式：2009-01-01 12:00:00 
         * @return String 返回值为：xx天xx小时xx分xx秒 
         */  
        public static String getDistanceTime(String starttime, String endtime) {  
            DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
            Date one;  
            Date two;  
            long day = 0;  
            long hour = 0;  
            long min = 0;  
            long sec = 0;  
            try {  
                one = df.parse(starttime);  
                two = df.parse(endtime);  
                long time1 = one.getTime();  
                long time2 = two.getTime();  
                long diff ;  
                if(time1<time2) {  
                    diff = time2 - time1;  
                } else {  
                    diff = time1 - time2;  
                }  
                day = diff / (24 * 60 * 60 * 1000);  
                hour = (diff / (60 * 60 * 1000) - day * 24);  
                min = ((diff / (60 * 1000)) - day * 24 * 60 - hour * 60);  
                sec = (diff/1000-day*24*60*60-hour*60*60-min*60);  
            } catch (ParseException e) {  
                e.printStackTrace();  
            }  
            return day + "天" + hour + "小时" + min + "分" + sec + "秒";  
        }  
	}
}

```
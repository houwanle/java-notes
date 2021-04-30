## Java：日志处理相关工具类

修饰符和类型 | 方法和描述
--- | ---
static Logger | getLogger(String tag) 获取日志的logger
static void | logDebug(String tag, String text, Throwable t) 输出debug级别的日志，支持输出调用堆栈
static void | logDebug(String tag, String text) 输出debug级别的日志
static void | logDebug(String tag, String text, Object... obj) 输出debug级别的日志
static void | logInfo(String tag, String text, Throwable t) 输出info级别的日志，支持输出调用堆栈
static void | logInfo(String tag, String text) 输出info级别的日志
static void | logInfo(String tag, String text, Object obj1) 输出info级别的日志
static void | logInfo(String tag, String text, Object obj1, Object obj2) 输出info级别的日志
static void | logInfo(String tag, String text, Object... obj) 输出info级别的日志
static void | logWarn(String tag, String text, Throwable t) 输出warn级别的日志，支持输出调用堆栈
static void | logWarn(String tag, String text) 输出warn级别的日志
static void | logWarn(String tag, String text, Object... obj) 输出warn级别的日志
static void | logError(String tag, String text, Throwable t) 输出error级别的日志，支持输出调用堆栈
static void | logError(String tag, String text) 输出error级别的日志
static void | logError(String tag, String text, Object obj) 输出error级别的日志
static void | logError(String tag, String text, Object obj1, Object obj2) 输出error级别的日志
static void | logError(String tag, String text, Object... obj) 输出error级别的日志
static void | logError(String tag, Exception e)
static String | makeLogForMapObj(String info, Map<String, Object> map, Exception e) 将 Map<String, Object>中的数据生成日志类型的文字描述
static String | makeLogForMapStr(String info, Map<String, String> map, Exception e) 将 Map<String, String>中的数据生成日志类型的文字描述
static String | makeLogForListStr(String info, List<String> list, Exception e) 将 List<String>中的数据生成日志类型的文字描述
static String | makeLogForListInt(String info, List<Integer> list, Exception e) 将 List<Integer>中的数据生成日志类型的文字描述

**源码**
```java
import java.io.PrintWriter;
import java.io.StringWriter;
import java.util.List;
import java.util.Map;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;


/**
 * 日志处理相关的工具类
 */
public class BaseLogMethod {
	private static final String TAG = "BaseLogMethod";

	/**
	 * 获取日志的logger
	 * @param tag	日志标签
	 */
	public static Logger getLogger(String tag) {
		return LoggerFactory.getLogger(tag);
	}

	/**
	 * 输出debug级别的日志，支持输出调用堆栈
	 * @param tag	日志标签
	 * @param text	日志内容
	 */
	public static void logDebug(String tag, String text, Throwable t) {
		Logger logger = getLogger(tag);
		if (logger.isDebugEnabled()) {
			logger.debug(text, t);
		}
	}

	/**
	 * 输出debug级别的日志
	 * @param tag	日志标签
	 * @param text	日志内容
	 */
	public static void logDebug(String tag, String text) {
		Logger logger = getLogger(tag);
		if (logger.isDebugEnabled()) {
			logger.debug(text);
		}
	}

	/**
	 * 输出debug级别的日志
	 * @param tag  日志标签
	 * @param text 日志内容
	 * @param obj  占位符匹配
	 */
	public static void logDebug(String tag, String text, Object... obj) {
		Logger logger = getLogger(tag);
		if (logger.isDebugEnabled()) {
			logger.debug(text, obj);
		}
	}

	/**
	 * 输出info级别的日志，支持输出调用堆栈
	 * @param tag	日志标签
	 * @param text	日志内容
	 */
	public static void logInfo(String tag, String text, Throwable t) {
		Logger logger = getLogger(tag);
		if (logger.isInfoEnabled()) {
			logger.info(text, t);
		}
	}

	/**
	 * 输出info级别的日志
	 * @param tag	日志标签
	 * @param text	日志内容
	 */
	public static void logInfo(String tag, String text) {
		Logger logger = getLogger(tag);
		if (logger.isInfoEnabled()) {
			logger.info(text);
		}
	}

	/**
	 * 输出info级别的日志
	 * @param tag	日志标签
	 * @param text	日志内容
	 * @param obj1  占位符匹配
	 */
	public static void logInfo(String tag, String text, Object obj1) {
		Logger logger = getLogger(tag);
		if (logger.isInfoEnabled()) {
			logger.info(text, obj1);
		}
	}

	/**
	 * 输出info级别的日志
	 * @param tag	日志标签
	 * @param text	日志内容
	 * @param obj1  占位符匹配
	 * @param obj2  占位符匹配
	 */
	public static void logInfo(String tag, String text, Object obj1, Object obj2) {
		Logger logger = getLogger(tag);
		if (logger.isInfoEnabled()) {
			logger.info(text, obj1, obj2);
		}
	}

	/**
	 * 输出info级别的日志
	 * @param tag	日志标签
	 * @param text	日志内容
	 * @param obj  占位符匹配
	 */
	public static void logInfo(String tag, String text, Object... obj) {
		Logger logger = getLogger(tag);
		if (logger.isInfoEnabled()) {
			logger.info(text, obj);
		}
	}

	/**
	 * 输出warn级别的日志，支持输出调用堆栈
	 * @param tag	日志标签
	 * @param text	日志内容
	 */
	public static void logWarn(String tag, String text, Throwable t) {
		Logger logger = getLogger(tag);
		if (logger.isWarnEnabled()) {
			logger.warn(text, t);
		}
	}

	/**
	 * 输出warn级别的日志
	 * @param tag	日志标签
	 * @param text	日志内容
	 */
	public static void logWarn(String tag, String text) {
		Logger logger = getLogger(tag);
		if (logger.isWarnEnabled()) {
			logger.warn(text);
		}
	}

	/**
	 * 输出warn级别的日志
	 * @param tag	日志标签
	 * @param text	日志内容
	 * @param obj  占位符匹配
	 */
	public static void logWarn(String tag, String text, Object... obj) {
		Logger logger = getLogger(tag);
		if (logger.isWarnEnabled()) {
			logger.warn(text, obj);
		}
	}

	/**
	 * 输出error级别的日志，支持输出调用堆栈
	 * @param tag	日志标签
	 * @param text	日志内容
	 */
	public static void logError(String tag, String text, Throwable t) {
		Logger logger = getLogger(tag);
		if (logger.isErrorEnabled()) {
			logger.error(text, t);
		}
	}

	/**
	 * 输出error级别的日志
	 * @param tag  日志标签
	 * @param text 日志内容
	 */
	public static void logError(String tag, String text) {
		Logger logger = getLogger(tag);
		if (logger.isErrorEnabled()) {
			logger.error(text);
		}
	}

	/**
	 * 输出error级别的日志
	 * @param tag	日志标签
	 * @param text	日志内容
	 * @param obj  占位符匹配
	 */
	public static void logError(String tag, String text, Object obj) {
		Logger logger = getLogger(tag);
		if (logger.isErrorEnabled()) {
			logger.error(text, obj);
		}
	}

	/**
	 * 输出error级别的日志
	 * @param tag	日志标签
	 * @param text	日志内容
	 * @param obj1  占位符匹配
	 * @param obj2  占位符匹配
	 */
	public static void logError(String tag, String text, Object obj1, Object obj2) {
		Logger logger = getLogger(tag);
		if (logger.isErrorEnabled()) {
			logger.error(text, obj1, obj2);
		}
	}

	/**
	 * 输出error级别的日志
	 * @param tag	日志标签
	 * @param text	日志内容
	 * @param obj  占位符匹配
	 */
	public static void logError(String tag, String text, Object... obj) {
		Logger logger = getLogger(tag);
		if (logger.isErrorEnabled()) {
			logger.error(text, obj);
		}
	}

	public static void logError(String tag, Exception e) {
		Logger logger = getLogger(tag);
		if (logger.isErrorEnabled()) {
			StringWriter sw = new StringWriter();
			e.printStackTrace(new PrintWriter(sw));
			logger.error(sw.toString());
		}
	}

	/**
	 * 将 Map<String, Object>中的数据生成日志类型的文字描述
	 * @param info	要添加到日志中的文字
	 * @param map	要处理的对象
	 * @param e		要添加的异常描述信息
	 * @return		生成的日志类型的文字描述
	 */
	public static String makeLogForMapObj(String info, Map<String, Object> map, Exception e) {
		StringBuilder sb = new StringBuilder();
		sb.append(info + "(");

		if (map != null) {
			try {
				map.forEach((key, value)->sb.append(key + "=" + value.toString() + " : "));
			} catch(Exception ex) {
				BaseLogMethod.logError(TAG, "makeLogForMapObj : " + ex.getMessage(), ex);
			}
		}

		sb.append(")");
		sb.append(e.getMessage());

		return sb.toString();
	}

	/**
	 * 将 Map<String, String>中的数据生成日志类型的文字描述
	 * @param info	要添加到日志中的文字
	 * @param map	要处理的对象
	 * @param e		要添加的异常描述信息
	 * @return		生成的日志类型的文字描述
	 */
	public static String makeLogForMapStr(String info, Map<String, String> map, Exception e) {
		StringBuilder sb = new StringBuilder();
		sb.append(info + "(");

		if (map != null) {
			map.forEach((key, value)->sb.append(key + "=" + value + " : "));
		}

		sb.append(")");
		sb.append(e.getMessage());

		return sb.toString();
	}

	/**
	 * 将 List<String>中的数据生成日志类型的文字描述
	 * @param info	要添加到日志中的文字
	 * @param list	要处理的对象
	 * @param e		要添加的异常描述信息
	 * @return		生成的日志类型的文字描述
	 */
	public static String makeLogForListStr(String info, List<String> list, Exception e) {
		StringBuilder sb = new StringBuilder();
		sb.append(info + "(");

		if (list != null) {
			for (String str : list) {
				sb.append(str + " : ");
			}
		}

		sb.append(")");
		sb.append(e.getMessage());

		return sb.toString();
	}

	/**
	 * 将 List<Integer>中的数据生成日志类型的文字描述
	 * @param info	要添加到日志中的文字
	 * @param list	要处理的对象
	 * @param e		要添加的异常描述信息
	 * @return		生成的日志类型的文字描述
	 */
	public static String makeLogForListInt(String info, List<Integer> list, Exception e) {
		StringBuilder sb = new StringBuilder();
		sb.append(info + "(");

		if (list != null) {
			for (Integer i : list) {
				sb.append(Integer.toString(i) + " : ");
			}
		}

		sb.append(")");
		sb.append(e.getMessage());

		return sb.toString();
	}

}

```

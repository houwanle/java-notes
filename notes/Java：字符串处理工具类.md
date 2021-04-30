## Java：字符串处理工具类

修饰符和类型 | 方法和描述
---|---
static boolean | isEmpty(String text)  判断字符串是否为空
static boolean | isNotEmpty(String text) 判断字符串是否不为空
static String | setValueWhenNull(String originalValue, String defaultValue) 当字符串为null设置一个缺省值
static Integer | strToInt(String val) 字符串类型转整数类型
static Integer | strToInt(String val, Integer defaultValue) 字符串类型转整数类型
static int | strToIntEx(String val, int defaultValue) 字符串类型转整数类型
static boolean | strToBool(String val, boolean defaultValue) 字符串类型转布尔类型
static Date | strToFormatDate(String val, Date defaultValue) 字符串类型转Date类型,要求字符串格式为: yyyy-MM-dd HH:mm:ss
static Long | strToLong(String val, Long defaultValue) 字符串类型转长整型
static Float | strToFloat(String val, Float defaultValue) 字符串类型转Float
static Double | strToDouble(String val, Double defaultValue) 字符串类型转Double


**源码**

```java
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * 字符串处理相关的工具类
 */
public class BaseStrMethod {
	private static final String TAG = "BaseStrMethod";

	/**
	 * 判断字符串是否为空
	 * @param text 要判断的字符串
	 * @return 空为true，非空为false
	 */
	public static boolean isEmpty(String text) {
		if (text == null || text.length() <= 0) {
			return true;
		} else {
			return false;
		}
	}

	/**
	 * 判断字符串是否不为空
	 * @param text 要判断的字符串
	 * @return 非空为true，空为false
	 */
	public static boolean isNotEmpty(String text) {
		return !isEmpty(text);
	}

	/**
	 * 当字符串为null设置一个缺省值
	 * @param originalValue 要处理的字符窜
	 * @param defaultValue 要设置的缺省值
	 * @return 非null时返回字符串本身，null时返回缺省值
	 */
	public static String setValueWhenNull(String originalValue, String defaultValue) {
		if (originalValue == null) {
			return defaultValue;
		} else {
			return originalValue;
		}
	}

	/**
	 * 字符串类型转整数类型
	 * @param val 要转换的字符串
	 * @return 转换后的整数类型
	 */
	public static Integer strToInt(String val) {
		return strToInt(val, 0);
	}

	/**
	 * 字符串类型转整数类型
	 * @param val          要转换的字符串
	 * @param defaultValue 转换失败后返回的值
	 * @return 转换后的整数类型
	 */
	public static Integer strToInt(String val, Integer defaultValue) {
		Integer result = defaultValue;
		try {
			if (isNotEmpty(val)) {
				result = Integer.parseInt(val);
			}
		} catch (Exception e) {
			BaseLogMethod.logError(TAG, "stringToInt(" + val + ") : " + e.getMessage(), e);
		}
		return result;
	}

	/**
	 * 字符串类型转整数类型
	 * @param val          要转换的字符串
	 * @param defaultValue 转换失败后返回的值
	 * @return 转换后的整数类型
	 */
	public static int strToIntEx(String val, int defaultValue) {
		int result = defaultValue;
		try {
			if (isNotEmpty(val)) {
				result = Integer.parseInt(val);
			}
		} catch (Exception e) {
			BaseLogMethod.logError(TAG, "strToIntEx(" + val + ") : " + e.getMessage(), e);
		}
		return result;
	}

	/**
	 * 字符串类型转布尔类型
	 * @param val          要转换的字符串
	 * @param defaultValue 转换失败时返回的值
	 * @return 转换后的类型
	 */
	public static boolean strToBool(String val, boolean defaultValue) {
		boolean result = defaultValue;
		try {
			if (isNotEmpty(val)) {
				result = Boolean.parseBoolean(val);
			}
		} catch (Exception e) {
			BaseLogMethod.logError(TAG, "strToBool(" + val + ") : " + e.getMessage(), e);
		}
		return result;
	}

	/**
	 * 字符串类型转Date类型,要求字符串格式为: yyyy-MM-dd HH:mm:ss
	 * @param val          要转换的字符串
	 * @param defaultValue 转换失败时返回的值
	 * @return 转换后的类型
	 */
	public static Date strToFormatDate(String val, Date defaultValue) {
		Date result = defaultValue;
		try {
			if (isNotEmpty(val)) {
				SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
				result = simpleDateFormat.parse(val);
			}
		} catch (Exception e) {
			BaseLogMethod.logError(TAG, "strToDate(" + val + ") : " + e.getMessage(), e);
		}
		return result;
	}

	/**
	 * 字符串类型转长整型
	 * @param val          要转换的字符串
	 * @param defaultValue 转换失败后返回的值
	 * @return 转换后的长整类型
	 */
	public static Long strToLong(String val, Long defaultValue) {
		Long result = defaultValue;
		try {
			if (isNotEmpty(val)) {
				result = Long.parseLong(val);
			}
		} catch (Exception e) {
			BaseLogMethod.logError(TAG, "strToLong(" + val + ") : " + e.getMessage(), e);
		}
		return result;
	}

	/**
	 * 字符串类型转Float
	 * @param val          要转换的字符串
	 * @param defaultValue 转换失败后返回的值
	 * @return 转换后的长整类型
	 */
	public static Float strToFloat(String val, Float defaultValue) {
		Float result = defaultValue;
		try {
			if (isNotEmpty(val)) {
				result = Float.parseFloat(val);
			}
		} catch (Exception e) {
			BaseLogMethod.logError(TAG, "strToFloat(" + val + ") : " + e.getMessage(), e);
		}
		return result;
	}

	/**
	 * 字符串类型转Double
	 * @param val          要转换的字符串
	 * @param defaultValue 转换失败后返回的值
	 * @return 转换后的长整类型
	 */
	public static Double strToDouble(String val, Double defaultValue) {
		Double result = defaultValue;
		try {
			if (isNotEmpty(val)) {
				result = Double.parseDouble(val);
			}
		} catch (Exception e) {
			BaseLogMethod.logError(TAG, "strToDouble(" + val + ") : " + e.getMessage(), e);
		}
		return result;
	}
}
```

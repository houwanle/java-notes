## Java：对象转换--序列化

修饰符和类型 | 方法和描述
--- | ---
static Map<String, String> | toMap(Object obj) 将对象转换为Map格式
static Map<String, Object> | toObjectMap(Object obj) 将对象转换为Map格式
static Map<String, Object> | toObjectMapSuportNull(Object obj) 将对象转换为Map格式，支持null
static String | objectToBase64(Object obj) 将对象序列化后进行Base64编码
static Object | base64ToObject(String text) 将进行Base64解编后再反序列化为对象

**源码**
```java
import org.apache.commons.beanutils.BeanMap;
import org.apache.commons.codec.binary.Base64;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * Bean处理相关的工具类
 */
public class BaseBeanMethod {
	private static final String TAG = "BaseBeanMethod";

	/**
	 * 将对象转换为Map格式
	 * @param obj 要转换的对象
	 * @return 转换后的数据
	 */
	public static Map<String, String> toMap(Object obj) {
		if (obj == null) {
			BaseLogMethod.logError(TAG, "toMap : obj is NULL");
			return null;
		}

		try {
			Map<String, String> result = new HashMap<String, String>();

			SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
			BeanMap bm = new BeanMap(obj);
			bm.forEach((key, value) -> {
				if (value != null) {
					if (value instanceof Date) {
						result.put(key.toString(), format.format(value));
					} else {
						result.put(key.toString(), value.toString());
					}
				}
			});

			// BeanMap函数自动生成的，因此去掉
			result.remove("class");

			return result;
		} catch (Exception e) {
			BaseLogMethod.logError(TAG, "toMap : " + e.getMessage(), e);
		}

		return null;
	}

	/**
	 * 将对象转换为Map格式
	 * @param obj 要转换的对象
	 * @return 转换后的数据
	 */
	public static Map<String, Object> toObjectMap(Object obj) {
		if (obj == null) {
			BaseLogMethod.logError(TAG, "toObjectMap : obj is NULL");
			return null;
		}

		try {
			BeanMap bm = new BeanMap(obj);
			Map<String, Object> result = new HashMap<String, Object>(bm.entrySet().size());
			bm.forEach((key, value) -> {
				if (value != null) {
					result.put(key.toString(), value);
				}
			});

			// BeanMap函数自动生成的，因此去掉
			result.remove("class");

			return result;
		} catch (Exception e) {
			BaseLogMethod.logError(TAG, "toObjectMap : " + e.getMessage(), e);
		}

		return null;
	}

	/**
	 * 将对象转换为Map格式，支持null
	 * @param obj 要转换的对象
	 * @return 转换后的数据
	 */
	public static Map<String, Object> toObjectMapSuportNull(Object obj) {
		if (obj == null) {
			BaseLogMethod.logError(TAG, "toObjectMapSuportNull : obj is NULL");
			return null;
		}

		try {
			BeanMap bm = new BeanMap(obj);
			Map<String, Object> result = new HashMap<String, Object>(bm.entrySet().size());
			bm.forEach((key, value) -> {
				result.put(key.toString(), value);
			});

			// BeanMap函数自动生成的，因此去掉
			result.remove("class");

			return result;
		} catch (Exception e) {
			BaseLogMethod.logError(TAG, "toObjectMapSuportNull : " + e.getMessage(), e);
		}

		return null;
	}

	/**
	 * 将对象序列化后进行Base64编码
	 * @param obj 要处理的对象
	 * @return 转换后的数据
	 */
	public static String objectToBase64(Object obj) {
		String result = null;
		if (obj == null) {
			BaseLogMethod.logError(TAG, "objectToBase64 : obj is NULL");
			return null;
		}

		try {
			ByteArrayOutputStream bos = new ByteArrayOutputStream();
			ObjectOutputStream oos = new ObjectOutputStream(bos);
			oos.writeObject(obj);
			result = Base64.encodeBase64String(bos.toByteArray());
		} catch (Exception e) {
			BaseLogMethod.logError(TAG, "objectToBase64 : " + obj.toString() + " : " + e.getMessage(), e);
		}

		return result;
	}

	/**
	 * 将进行Base64解编后再反序列化为对象
	 * @param text 要处理的对象
	 * @return 转换后的对象
	 */
	public static Object base64ToObject(String text) {
		Object result = null;
		if (BaseStrMethod.isEmpty(text)) {
			BaseLogMethod.logError(TAG, "base64ToObject : text is NULL");
			return null;
		}

		try {
			byte[] input = Base64.decodeBase64(text);
			ByteArrayInputStream bis = new ByteArrayInputStream(input);
			ObjectInputStream ois = new ObjectInputStream(bis);
			result = ois.readObject();
		} catch (Exception e) {
			BaseLogMethod.logError(TAG, "base64ToObject : " + text + " : " + e.getMessage(), e);
		}

		return result;
	}

}

```

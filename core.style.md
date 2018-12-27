# Style

## ValueStyler.java

```java
package org.springframework.core.style;
/**
 * 值样式器
 */
public interface ValueStyler {

	String style(Object value);

}

```

### DefaultValueStyler.java

```java
package org.springframework.core.style;

import java.lang.reflect.Method;
import java.util.Collection;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.Set;

import org.springframework.util.ClassUtils;
import org.springframework.util.ObjectUtils;

/**
 * 默认的值样式器
 */
public class DefaultValueStyler implements ValueStyler {

	private static final String EMPTY = "[empty]";
	private static final String NULL = "[null]";
	private static final String COLLECTION = "collection";
	private static final String SET = "set";
	private static final String LIST = "list";
	private static final String MAP = "map";
	private static final String ARRAY = "array";


	@Override
	public String style(Object value) {
		if (value == null) {
			return NULL;
		}
		else if (value instanceof String) {
			return "\'" + value + "\'";
		}
		else if (value instanceof Class) {
			return ClassUtils.getShortName((Class<?>) value);
		}
		else if (value instanceof Method) {
			Method method = (Method) value;
			return method.getName() + "@" + ClassUtils.getShortName(method.getDeclaringClass());
		}
		else if (value instanceof Map) {
			return style((Map<?, ?>) value);
		}
		else if (value instanceof Map.Entry) {
			return style((Map.Entry<? ,?>) value);
		}
		else if (value instanceof Collection) {
			return style((Collection<?>) value);
		}
		else if (value.getClass().isArray()) {
			return styleArray(ObjectUtils.toObjectArray(value));
		}
		else {
			return String.valueOf(value);
		}
	}

	private <K, V> String style(Map<K, V> value) {
		StringBuilder result = new StringBuilder(value.size() * 8 + 16);
		result.append(MAP + "[");
		for (Iterator<Map.Entry<K, V>> it = value.entrySet().iterator(); it.hasNext();) {
			Map.Entry<K, V> entry = it.next();
			result.append(style(entry));
			if (it.hasNext()) {
				result.append(',').append(' ');
			}
		}
		if (value.isEmpty()) {
			result.append(EMPTY);
		}
		result.append("]");
		return result.toString();
	}

	private String style(Map.Entry<?, ?> value) {
		return style(value.getKey()) + " -> " + style(value.getValue());
	}

	private String style(Collection<?> value) {
		StringBuilder result = new StringBuilder(value.size() * 8 + 16);
		result.append(getCollectionTypeString(value)).append('[');
		for (Iterator<?> i = value.iterator(); i.hasNext();) {
			result.append(style(i.next()));
			if (i.hasNext()) {
				result.append(',').append(' ');
			}
		}
		if (value.isEmpty()) {
			result.append(EMPTY);
		}
		result.append("]");
		return result.toString();
	}

	private String getCollectionTypeString(Collection<?> value) {
		if (value instanceof List) {
			return LIST;
		}
		else if (value instanceof Set) {
			return SET;
		}
		else {
			return COLLECTION;
		}
	}

	private String styleArray(Object[] array) {
		StringBuilder result = new StringBuilder(array.length * 8 + 16);
		result.append(ARRAY + "<").append(ClassUtils.getShortName(array.getClass().getComponentType())).append(">[");
		for (int i = 0; i < array.length - 1; i++) {
			result.append(style(array[i]));
			result.append(',').append(' ');
		}
		if (array.length > 0) {
			result.append(style(array[array.length - 1]));
		}
		else {
			result.append(EMPTY);
		}
		result.append("]");
		return result.toString();
	}

}

```

### Demo

```java
	public static void main(String[] args) throws Exception {
		ValueStyler valueStyler = new DefaultValueStyler();
		System.out.println(valueStyler.style(null));
		System.out.println(valueStyler.style("Hello,World!"));
		System.out.println(valueStyler.style(Object.class));
		System.out.println(valueStyler.style(Object.class.getDeclaredMethod("notify")));
		Map<Object, Object> map = new HashMap<>();
		System.out.println(valueStyler.style(map));
		map.put("now", new Date());
		System.out.println(valueStyler.style(map));
		System.out.println(valueStyler.style(map.entrySet().iterator().next()));
		Collection<Object> collection = new ArrayList<>();
		System.out.println(valueStyler.style(collection));
		collection.add(new Date());
		System.out.println(valueStyler.style(collection));
		System.out.println(valueStyler.style(new String[] { "a", "b", "c" }));
		System.out.println(valueStyler.style(new Date()));
	}
```

```console
[null]
'Hello,World!'
Object
notify@Object
map[[empty]]
map['now' -> Wed Dec 26 13:02:24 CST 2018]
'now' -> Wed Dec 26 13:02:24 CST 2018
list[[empty]]
list[Wed Dec 26 13:02:24 CST 2018]
array<String>['a', 'b', 'c']
Wed Dec 26 13:02:24 CST 2018
```



## ToStringStyler.java

```java
package org.springframework.core.style;
/**
 * ToString样式器
 */
public interface ToStringStyler {

	void styleStart(StringBuilder buffer, Object obj);

	void styleEnd(StringBuilder buffer, Object obj);

	void styleField(StringBuilder buffer, String fieldName, Object value);

	void styleValue(StringBuilder buffer, Object value);

	void styleFieldSeparator(StringBuilder buffer);

}

```

### DefaultToStringStyler.java

```java
package org.springframework.core.style;

import org.springframework.util.Assert;
import org.springframework.util.ClassUtils;
import org.springframework.util.ObjectUtils;

/**
 * 默认的ToString样式器
 */
public class DefaultToStringStyler implements ToStringStyler {

	private final ValueStyler valueStyler;

	public DefaultToStringStyler(ValueStyler valueStyler) {
		Assert.notNull(valueStyler, "ValueStyler must not be null");
		this.valueStyler = valueStyler;
	}

	protected final ValueStyler getValueStyler() {
		return this.valueStyler;
	}


	@Override
	public void styleStart(StringBuilder buffer, Object obj) {
		if (!obj.getClass().isArray()) {
			buffer.append('[').append(ClassUtils.getShortName(obj.getClass()));
			styleIdentityHashCode(buffer, obj);
		}
		else {
			buffer.append('[');
			styleIdentityHashCode(buffer, obj);
			buffer.append(' ');
			styleValue(buffer, obj);
		}
	}

	private void styleIdentityHashCode(StringBuilder buffer, Object obj) {
		buffer.append('@');
		buffer.append(ObjectUtils.getIdentityHexString(obj));
	}

	@Override
	public void styleEnd(StringBuilder buffer, Object o) {
		buffer.append(']');
	}

	@Override
	public void styleField(StringBuilder buffer, String fieldName, Object value) {
		styleFieldStart(buffer, fieldName);
		styleValue(buffer, value);
		styleFieldEnd(buffer, fieldName);
	}

	protected void styleFieldStart(StringBuilder buffer, String fieldName) {
		buffer.append(' ').append(fieldName).append(" = ");
	}

	protected void styleFieldEnd(StringBuilder buffer, String fieldName) {
	}

	@Override
	public void styleValue(StringBuilder buffer, Object value) {
		buffer.append(this.valueStyler.style(value));
	}

	@Override
	public void styleFieldSeparator(StringBuilder buffer) {
		buffer.append(',');
	}

}

```

## StylerUtils.java

```java
package org.springframework.core.style;

public abstract class StylerUtils {

	static final ValueStyler DEFAULT_VALUE_STYLER = new DefaultValueStyler();

	public static String style(Object value) {
		return DEFAULT_VALUE_STYLER.style(value);
	}

}

```

## ToStringCreator.java

```java
package org.springframework.core.style;

import org.springframework.util.Assert;

public class ToStringCreator {

	private static final ToStringStyler DEFAULT_TO_STRING_STYLER =
			new DefaultToStringStyler(StylerUtils.DEFAULT_VALUE_STYLER);


	private final StringBuilder buffer = new StringBuilder(256);

	private final ToStringStyler styler;

	private final Object object;

	private boolean styledFirstField;

	public ToStringCreator(Object obj) {
		this(obj, (ToStringStyler) null);
	}

	public ToStringCreator(Object obj, ValueStyler styler) {
		this(obj, new DefaultToStringStyler(styler != null ? styler : StylerUtils.DEFAULT_VALUE_STYLER));
	}

	public ToStringCreator(Object obj, ToStringStyler styler) {
		Assert.notNull(obj, "The object to be styled must not be null");
		this.object = obj;
		this.styler = (styler != null ? styler : DEFAULT_TO_STRING_STYLER);
		this.styler.styleStart(this.buffer, this.object);
	}

	public ToStringCreator append(String fieldName, byte value) {
		return append(fieldName, Byte.valueOf(value));
	}

	public ToStringCreator append(String fieldName, short value) {
		return append(fieldName, Short.valueOf(value));
	}

	public ToStringCreator append(String fieldName, int value) {
		return append(fieldName, Integer.valueOf(value));

	public ToStringCreator append(String fieldName, long value) {
		return append(fieldName, Long.valueOf(value));
	}

	public ToStringCreator append(String fieldName, float value) {
		return append(fieldName, Float.valueOf(value));
	}

	public ToStringCreator append(String fieldName, double value) {
		return append(fieldName, Double.valueOf(value));
	}

	public ToStringCreator append(String fieldName, boolean value) {
		return append(fieldName, Boolean.valueOf(value));
	}

	public ToStringCreator append(String fieldName, Object value) {
		printFieldSeparatorIfNecessary();
		this.styler.styleField(this.buffer, fieldName, value);
		return this;
	}

	private void printFieldSeparatorIfNecessary() {
		if (this.styledFirstField) {
			this.styler.styleFieldSeparator(this.buffer);
		}
		else {
			this.styledFirstField = true;
		}
	}

	public ToStringCreator append(Object value) {
		this.styler.styleValue(this.buffer, value);
		return this;
	}

	@Override
	public String toString() {
		this.styler.styleEnd(this.buffer, this.object);
		return this.buffer.toString();
	}

}

```

## Demo

```java
package com.zby;

import java.util.Arrays;
import java.util.Date;
import java.util.List;

import org.springframework.core.style.ToStringCreator;
import org.springframework.util.ObjectUtils;

public class User {
	private Long id;
	private String name;
	private Date birthday;
	private List<String> hobby;

	public User(Long id, String name, Date birthday, List<String> hobby) {
		super();
		this.id = id;
		this.name = name;
		this.birthday = birthday;
		this.hobby = hobby;
	}

	@Override
	public String toString() {
		return new ToStringCreator(this).append("id", this.id).append("name", this.name).append("birthday", this.birthday)
				.append("hobby", ObjectUtils.nullSafeToString(this.hobby)).toString();
	}

	public static void main(String[] args) {
		User user = new User(1L, "zby", new Date(), Arrays.asList("eat", "sleep", "code"));
		System.out.println(user);
	}

}

```

```console
[User@7440e464 id = 1, name = 'zby', birthday = Wed Dec 26 13:18:21 CST 2018, hobby = '[eat, sleep, code]']
```


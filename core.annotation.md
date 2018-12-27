# Annotation

## Order.java

```java
package org.springframework.core.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.core.Ordered;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.FIELD})
@Documented
public @interface Order {

	int value() default Ordered.LOWEST_PRECEDENCE;

}

```

## OrderUtils.java

```java
package org.springframework.core.annotation;

import java.lang.annotation.Annotation;

import org.springframework.util.ClassUtils;

/**
 * 顺序工具类。用于获取类型Order注解或者Priority注解的顺序。
 */
@SuppressWarnings("unchecked")
public abstract class OrderUtils {

	private static Class<? extends Annotation> priorityAnnotationType = null;

	static {
		try {
			priorityAnnotationType = (Class<? extends Annotation>)
					ClassUtils.forName("javax.annotation.Priority", OrderUtils.class.getClassLoader());
		}
		catch (Throwable ex) {
			// javax.annotation.Priority not available, or present but not loadable (on JDK 6)
		}
	}

	public static Integer getOrder(Class<?> type) {
		return getOrder(type, null);
	}

	public static Integer getOrder(Class<?> type, Integer defaultOrder) {
		Order order = AnnotationUtils.findAnnotation(type, Order.class);
		if (order != null) {
			return order.value();
		}
		Integer priorityOrder = getPriority(type);
		if (priorityOrder != null) {
			return priorityOrder;
		}
		return defaultOrder;
	}

	public static Integer getPriority(Class<?> type) {
		if (priorityAnnotationType != null) {
			Annotation priority = AnnotationUtils.findAnnotation(type, priorityAnnotationType);
			if (priority != null) {
				return (Integer) AnnotationUtils.getValue(priority);
			}
		}
		return null;
	}

}

```



## AliasFor.java

```java
package org.springframework.core.annotation;

import java.lang.annotation.Annotation;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface AliasFor {

	@AliasFor("attribute")
	String value() default "";

	@AliasFor("value")
	String attribute() default "";

	Class<? extends Annotation> annotation() default Annotation.class;

}

```



## AnnotationAttributeExtractor.java

```java
package org.springframework.core.annotation;

import java.lang.annotation.Annotation;
import java.lang.reflect.Method;

/**
 * 注解属性提取器
 */
interface AnnotationAttributeExtractor<S> {

	Class<? extends Annotation> getAnnotationType();

	Object getAnnotatedElement();

	S getSource();

	Object getAttributeValue(Method attributeMethod);

}

```

### AbstractAliasAwareAnnotationAttributeExtractor.java

```java
package org.springframework.core.annotation;

import java.lang.annotation.Annotation;
import java.lang.reflect.Method;
import java.util.List;
import java.util.Map;

import org.springframework.util.Assert;
import org.springframework.util.ObjectUtils;

/**
 * 支持@AliasFor注解注解属性提取器抽象实现
 */
abstract class AbstractAliasAwareAnnotationAttributeExtractor<S> implements AnnotationAttributeExtractor<S> {

	private final Class<? extends Annotation> annotationType;

	private final Object annotatedElement;

	private final S source;

	private final Map<String, List<String>> attributeAliasMap;

	AbstractAliasAwareAnnotationAttributeExtractor(
			Class<? extends Annotation> annotationType, Object annotatedElement, S source) {

		Assert.notNull(annotationType, "annotationType must not be null");
		Assert.notNull(source, "source must not be null");
		this.annotationType = annotationType;
		this.annotatedElement = annotatedElement;
		this.source = source;
		this.attributeAliasMap = AnnotationUtils.getAttributeAliasMap(annotationType);
	}


	@Override
	public final Class<? extends Annotation> getAnnotationType() {
		return this.annotationType;
	}

	@Override
	public final Object getAnnotatedElement() {
		return this.annotatedElement;
	}

	@Override
	public final S getSource() {
		return this.source;
	}

	@Override
	public final Object getAttributeValue(Method attributeMethod) {
		String attributeName = attributeMethod.getName();
		Object attributeValue = getRawAttributeValue(attributeMethod);

		List<String> aliasNames = this.attributeAliasMap.get(attributeName);
		if (aliasNames != null) {
			Object defaultValue = AnnotationUtils.getDefaultValue(this.annotationType, attributeName);
			for (String aliasName : aliasNames) {
				Object aliasValue = getRawAttributeValue(aliasName);

				if (!ObjectUtils.nullSafeEquals(attributeValue, aliasValue) &&
						!ObjectUtils.nullSafeEquals(attributeValue, defaultValue) &&
						!ObjectUtils.nullSafeEquals(aliasValue, defaultValue)) {
					String elementName = (this.annotatedElement != null ? this.annotatedElement.toString() : "unknown element");
					throw new AnnotationConfigurationException(String.format(
							"In annotation [%s] declared on %s and synthesized from [%s], attribute '%s' and its " +
							"alias '%s' are present with values of [%s] and [%s], but only one is permitted.",
							this.annotationType.getName(), elementName, this.source, attributeName, aliasName,
							ObjectUtils.nullSafeToString(attributeValue), ObjectUtils.nullSafeToString(aliasValue)));
				}

				// If the user didn't declare the annotation with an explicit value,
				// use the value of the alias instead.
				if (ObjectUtils.nullSafeEquals(attributeValue, defaultValue)) {
					attributeValue = aliasValue;
				}
			}
		}

		return attributeValue;
	}

	protected abstract Object getRawAttributeValue(Method attributeMethod);

	protected abstract Object getRawAttributeValue(String attributeName);

}

```

#### DefaultAnnotationAttributeExtractor.java

```java
package org.springframework.core.annotation;

import java.lang.annotation.Annotation;
import java.lang.reflect.Method;

import org.springframework.util.ReflectionUtils;

/**
 * 默认的注解属性提取器。以annotation作为源获取属性。
 */
class DefaultAnnotationAttributeExtractor extends AbstractAliasAwareAnnotationAttributeExtractor<Annotation> {


	DefaultAnnotationAttributeExtractor(Annotation annotation, Object annotatedElement) {
		super(annotation.annotationType(), annotatedElement, annotation);
	}


	@Override
	protected Object getRawAttributeValue(Method attributeMethod) {
		ReflectionUtils.makeAccessible(attributeMethod);
		return ReflectionUtils.invokeMethod(attributeMethod, getSource());
	}

	@Override
	protected Object getRawAttributeValue(String attributeName) {
		Method attributeMethod = ReflectionUtils.findMethod(getAnnotationType(), attributeName);
		return getRawAttributeValue(attributeMethod);
	}

}

```

#### MapAnnotationAttributeExtractor.java

```java
package org.springframework.core.annotation;

import java.lang.annotation.Annotation;
import java.lang.reflect.AnnotatedElement;
import java.lang.reflect.Array;
import java.lang.reflect.Method;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

import org.springframework.util.Assert;
import org.springframework.util.ClassUtils;

/**
 * Map支持的注解属性提取器。以Map作为源获取属性
 */
class MapAnnotationAttributeExtractor extends AbstractAliasAwareAnnotationAttributeExtractor<Map<String, Object>> {

	MapAnnotationAttributeExtractor(Map<String, Object> attributes, Class<? extends Annotation> annotationType,
			AnnotatedElement annotatedElement) {

		super(annotationType, annotatedElement, enrichAndValidateAttributes(attributes, annotationType));
	}


	@Override
	protected Object getRawAttributeValue(Method attributeMethod) {
		return getRawAttributeValue(attributeMethod.getName());
	}

	@Override
	protected Object getRawAttributeValue(String attributeName) {
		return getSource().get(attributeName);
	}


	/**
	 * 增强并验证属性。包含别名解析，类型转化等
	 */
	@SuppressWarnings("unchecked")
	private static Map<String, Object> enrichAndValidateAttributes(
			Map<String, Object> originalAttributes, Class<? extends Annotation> annotationType) {

		Map<String, Object> attributes = new LinkedHashMap<String, Object>(originalAttributes);
		Map<String, List<String>> attributeAliasMap = AnnotationUtils.getAttributeAliasMap(annotationType);

		for (Method attributeMethod : AnnotationUtils.getAttributeMethods(annotationType)) {
			String attributeName = attributeMethod.getName();
			Object attributeValue = attributes.get(attributeName);

			// if attribute not present, check aliases
			if (attributeValue == null) {
				List<String> aliasNames = attributeAliasMap.get(attributeName);
				if (aliasNames != null) {
					for (String aliasName : aliasNames) {
						Object aliasValue = attributes.get(aliasName);
						if (aliasValue != null) {
							attributeValue = aliasValue;
							attributes.put(attributeName, attributeValue);
							break;
						}
					}
				}
			}

			// if aliases not present, check default
			if (attributeValue == null) {
				Object defaultValue = AnnotationUtils.getDefaultValue(annotationType, attributeName);
				if (defaultValue != null) {
					attributeValue = defaultValue;
					attributes.put(attributeName, attributeValue);
				}
			}

			// if still null
			if (attributeValue == null) {
				throw new IllegalArgumentException(String.format(
						"Attributes map %s returned null for required attribute '%s' defined by annotation type [%s].",
						attributes, attributeName, annotationType.getName()));
			}

			// finally, ensure correct type
			Class<?> requiredReturnType = attributeMethod.getReturnType();
			Class<? extends Object> actualReturnType = attributeValue.getClass();

			if (!ClassUtils.isAssignable(requiredReturnType, actualReturnType)) {
				boolean converted = false;

				// Single element overriding an array of the same type?
				if (requiredReturnType.isArray() && requiredReturnType.getComponentType() == actualReturnType) {
					Object array = Array.newInstance(requiredReturnType.getComponentType(), 1);
					Array.set(array, 0, attributeValue);
					attributes.put(attributeName, array);
					converted = true;
				}

				// Nested map representing a single annotation?
				else if (Annotation.class.isAssignableFrom(requiredReturnType) &&
						Map.class.isAssignableFrom(actualReturnType)) {
					Class<? extends Annotation> nestedAnnotationType =
							(Class<? extends Annotation>) requiredReturnType;
					Map<String, Object> map = (Map<String, Object>) attributeValue;
					attributes.put(attributeName, AnnotationUtils.synthesizeAnnotation(map, nestedAnnotationType, null));
					converted = true;
				}

				// Nested array of maps representing an array of annotations?
				else if (requiredReturnType.isArray() && actualReturnType.isArray() &&
						Annotation.class.isAssignableFrom(requiredReturnType.getComponentType()) &&
						Map.class.isAssignableFrom(actualReturnType.getComponentType())) {
					Class<? extends Annotation> nestedAnnotationType =
							(Class<? extends Annotation>) requiredReturnType.getComponentType();
					Map<String, Object>[] maps = (Map<String, Object>[]) attributeValue;
					attributes.put(attributeName, AnnotationUtils.synthesizeAnnotationArray(maps, nestedAnnotationType));
					converted = true;
				}

				if (!converted) {
					throw new IllegalArgumentException(String.format(
							"Attributes map %s returned a value of type [%s] for attribute '%s', " +
							"but a value of type [%s] is required as defined by annotation type [%s].",
							attributes, actualReturnType.getName(), attributeName, requiredReturnType.getName(),
							annotationType.getName()));
				}
			}
		}

		return attributes;
	}

}

```

## SynthesizedAnnotation.java

```java
package org.springframework.core.annotation;

/**
 * 合成注解的标记接口
 */
public interface SynthesizedAnnotation {
}

```


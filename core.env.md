# Environment

Spring中的一个重要抽象，用于将属性源跟引用关联，可以方便的使用系统变量、命令行参数等。

## PropertySource.java

```java
package org.springframework.core.env;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import org.springframework.util.Assert;
import org.springframework.util.ObjectUtils;

/**
 * 属性源，封装一个键值对。获取属性由子类实现。source不是具体的属性，而是属性集合，比如Map或者Properties。
 * 相当于name->{propertyName->value},具体获取属性由子类实现。
 */
public abstract class PropertySource<T> {

	protected final Log logger = LogFactory.getLog(getClass());

	protected final String name;

	protected final T source;

	public PropertySource(String name, T source) {
		Assert.hasText(name, "Property source name must contain at least one character");
		Assert.notNull(source, "Property source must not be null");
		this.name = name;
		this.source = source;
	}

	@SuppressWarnings("unchecked")
	public PropertySource(String name) {
		this(name, (T) new Object());
	}

	public String getName() {
		return this.name;
	}

	public T getSource() {
		return this.source;
	}

	public boolean containsProperty(String name) {
		return (getProperty(name) != null);
	}

	public abstract Object getProperty(String name);


	@Override
	public boolean equals(Object obj) {
		return (this == obj || (obj instanceof PropertySource &&
				ObjectUtils.nullSafeEquals(this.name, ((PropertySource<?>) obj).name)));
	}

	@Override
	public int hashCode() {
		return ObjectUtils.nullSafeHashCode(this.name);
	}

	@Override
	public String toString() {
		if (logger.isDebugEnabled()) {
			return getClass().getSimpleName() + "@" + System.identityHashCode(this) +
					" {name='" + this.name + "', properties=" + this.source + "}";
		}
		else {
			return getClass().getSimpleName() + " {name='" + this.name + "'}";
		}
	}

	public static PropertySource<?> named(String name) {
		return new ComparisonPropertySource(name);
	}


	/**
	 * 存根属性源，用于在上下文初始化时无法获取，但是需要用来保持顺序的时候，先用这个占位，等实际属性源初始化完成后再替换。
	 */
	public static class StubPropertySource extends PropertySource<Object> {

		public StubPropertySource(String name) {
			super(name, new Object());
		}

		@Override
		public String getProperty(String name) {
			return null;
		}
	}


	/**
	 * 比较属性源。可以看到PropertySource重写的equals、toString、hashcode都是基于name，可以通过构造本类实例，指定一个名字，
	 * 判断该属性源是否与另一个实际的属性源是否相等。例：
	 *  List<PropertySource<?>> sources = new ArrayList<PropertySource<?>>();
	 * sources.add(new MapPropertySource("sourceA", mapA));
 	 * sources.add(new MapPropertySource("sourceB", mapB));
 	 * assert sources.contains(PropertySource.named("sourceA"));
 	 * assert sources.contains(PropertySource.named("sourceB"));
 	 * assert !sources.contains(PropertySource.named("sourceC"));
	 */
	static class ComparisonPropertySource extends StubPropertySource {

		private static final String USAGE_ERROR =
				"ComparisonPropertySource instances are for use with collection comparison only";

		public ComparisonPropertySource(String name) {
			super(name);
		}

		@Override
		public Object getSource() {
			throw new UnsupportedOperationException(USAGE_ERROR);
		}

		@Override
		public boolean containsProperty(String name) {
			throw new UnsupportedOperationException(USAGE_ERROR);
		}

		@Override
		public String getProperty(String name) {
			throw new UnsupportedOperationException(USAGE_ERROR);
		}
	}

}
```

### EnumerablePropertySource.java

```java
package org.springframework.core.env;

import org.springframework.util.ObjectUtils;

/**
 * 可枚举的属性源
 */
public abstract class EnumerablePropertySource<T> extends PropertySource<T> {

	public EnumerablePropertySource(String name, T source) {
		super(name, source);
	}

	protected EnumerablePropertySource(String name) {
		super(name);
	}

	@Override
	public boolean containsProperty(String name) {
		return ObjectUtils.containsElement(getPropertyNames(), name);
	}

	public abstract String[] getPropertyNames();

}
 
```



#### MapPropertySource.java

```java
package org.springframework.core.env;

import java.util.Map;

import org.springframework.util.StringUtils;

/**
 * 使用Map作为source的属性源
 */
public class MapPropertySource extends EnumerablePropertySource<Map<String, Object>> {

	public MapPropertySource(String name, Map<String, Object> source) {
		super(name, source);
	}


	@Override
	public Object getProperty(String name) {
		return this.source.get(name);
	}

	@Override
	public boolean containsProperty(String name) {
		return this.source.containsKey(name);
	}

	@Override
	public String[] getPropertyNames() {
		return StringUtils.toStringArray(this.source.keySet());
	}

}
```
##### PropertiesPropertySource.java

```java
package org.springframework.core.env;

import java.util.Map;
import java.util.Properties;

/**
 * 使用Properties作为source的属性源
 */
public class PropertiesPropertySource extends MapPropertySource {

	@SuppressWarnings({"unchecked", "rawtypes"})
	public PropertiesPropertySource(String name, Properties source) {
		super(name, (Map) source);
	}

	protected PropertiesPropertySource(String name, Map<String, Object> source) {
		super(name, source);
	}

}

```
##### SystemEnvironmentPropertySource.java

```java
package org.springframework.core.env;

import java.util.Map;

import org.springframework.util.Assert;

/**
 * 系统环境属性源
 * foo.bar - the original name 
 * foo_bar - with underscores for periods (if any) 
 * FOO.BAR - original, with upper case 
 * FOO_BAR - with underscores and upper case 	
 */
public class SystemEnvironmentPropertySource extends MapPropertySource {

	public SystemEnvironmentPropertySource(String name, Map<String, Object> source) {
		super(name, source);
	}

	@Override
	public boolean containsProperty(String name) {
		return (getProperty(name) != null);
	}

	@Override
	public Object getProperty(String name) {
		String actualName = resolvePropertyName(name);
		if (logger.isDebugEnabled() && !name.equals(actualName)) {
			logger.debug("PropertySource '" + getName() + "' does not contain property '" + name +
					"', but found equivalent '" + actualName + "'");
		}
		return super.getProperty(actualName);
	}

	private String resolvePropertyName(String name) {
		Assert.notNull(name, "Property name must not be null");
		String resolvedName = checkPropertyName(name);
		if (resolvedName != null) {
			return resolvedName;
		}
		String uppercasedName = name.toUpperCase();
		if (!name.equals(uppercasedName)) {
			resolvedName = checkPropertyName(uppercasedName);
			if (resolvedName != null) {
				return resolvedName;
			}
		}
		return name;
	}

	private String checkPropertyName(String name) {
		// Check name as-is
		if (containsKey(name)) {
			return name;
		}
		// Check name with just dots replaced
		String noDotName = name.replace('.', '_');
		if (!name.equals(noDotName) && containsKey(noDotName)) {
			return noDotName;
		}
		// Check name with just hyphens replaced
		String noHyphenName = name.replace('-', '_');
		if (!name.equals(noHyphenName) && containsKey(noHyphenName)) {
			return noHyphenName;
		}
		// Check name with dots and hyphens replaced
		String noDotNoHyphenName = noDotName.replace('-', '_');
		if (!noDotName.equals(noDotNoHyphenName) && containsKey(noDotNoHyphenName)) {
			return noDotNoHyphenName;
		}
		// Give up
		return null;
	}

	private boolean containsKey(String name) {
		return (isSecurityManagerPresent() ? this.source.keySet().contains(name) : this.source.containsKey(name));
	}

	protected boolean isSecurityManagerPresent() {
		return (System.getSecurityManager() != null);
	}

}
```


#### CommandLinePropertySource.java

```java
package org.springframework.core.env;

import java.util.Collection;
import java.util.List;

import org.springframework.util.StringUtils;

/**
 * 抽象的命令行属性源，默认名称commandLineArgs
 * 非选项参数option作为一个整体，通过名称获取一个数组。名称可以设置，默认nonOptionArgs。
 * 选项参数--name[=value]
 */
public abstract class CommandLinePropertySource<T> extends EnumerablePropertySource<T> {

	public static final String COMMAND_LINE_PROPERTY_SOURCE_NAME = "commandLineArgs";

	public static final String DEFAULT_NON_OPTION_ARGS_PROPERTY_NAME = "nonOptionArgs";

	private String nonOptionArgsPropertyName = DEFAULT_NON_OPTION_ARGS_PROPERTY_NAME;

	public CommandLinePropertySource(T source) {
		super(COMMAND_LINE_PROPERTY_SOURCE_NAME, source);
	}

	public CommandLinePropertySource(String name, T source) {
		super(name, source);
	}

	public void setNonOptionArgsPropertyName(String nonOptionArgsPropertyName) {
		this.nonOptionArgsPropertyName = nonOptionArgsPropertyName;
	}

	@Override
	public final boolean containsProperty(String name) {
		if (this.nonOptionArgsPropertyName.equals(name)) {
			return !this.getNonOptionArgs().isEmpty();
		}
		return this.containsOption(name);
	}

	@Override
	public final String getProperty(String name) {
		if (this.nonOptionArgsPropertyName.equals(name)) {
			Collection<String> nonOptionArguments = this.getNonOptionArgs();
			if (nonOptionArguments.isEmpty()) {
				return null;
			}
			else {
				return StringUtils.collectionToCommaDelimitedString(nonOptionArguments);
			}
		}
		Collection<String> optionValues = this.getOptionValues(name);
		if (optionValues == null) {
			return null;
		}
		else {
			return StringUtils.collectionToCommaDelimitedString(optionValues);
		}
	}

	protected abstract boolean containsOption(String name);

	protected abstract List<String> getOptionValues(String name);

	protected abstract List<String> getNonOptionArgs();

}
```
##### CommandLineArgs.java

```java
package org.springframework.core.env;

import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;

/**
 * 对命令行参数的简单封装，把命令行参数分为选项参数--name[=value]和非选项参数option
 * 选项参数封装为Map<String, List<String>>,非选项参数封装为List<String>
 */
class CommandLineArgs {

	private final Map<String, List<String>> optionArgs = new HashMap<String, List<String>>();
	private final List<String> nonOptionArgs = new ArrayList<String>();

	public void addOptionArg(String optionName, String optionValue) {
		if (!this.optionArgs.containsKey(optionName)) {
			this.optionArgs.put(optionName, new ArrayList<String>());
		}
		if (optionValue != null) {
			this.optionArgs.get(optionName).add(optionValue);
		}
	}

	public Set<String> getOptionNames() {
		return Collections.unmodifiableSet(this.optionArgs.keySet());
	}

	public boolean containsOption(String optionName) {
		return this.optionArgs.containsKey(optionName);
	}

	public List<String> getOptionValues(String optionName) {
		return this.optionArgs.get(optionName);
	}

	public void addNonOptionArg(String value) {
		this.nonOptionArgs.add(value);
	}

	public List<String> getNonOptionArgs() {
		return Collections.unmodifiableList(this.nonOptionArgs);
	}

}

```

##### SimpleCommandLineArgsParser.java

```java
package org.springframework.core.env;

/**
 * 把命令行参数解析为CommandLineArgs对象
 */
class SimpleCommandLineArgsParser {

	public CommandLineArgs parse(String... args) {
		CommandLineArgs commandLineArgs = new CommandLineArgs();
		for (String arg : args) {
			if (arg.startsWith("--")) {
				String optionText = arg.substring(2, arg.length());
				String optionName;
				String optionValue = null;
				if (optionText.contains("=")) {
					optionName = optionText.substring(0, optionText.indexOf('='));
					optionValue = optionText.substring(optionText.indexOf('=')+1, optionText.length());
				}
				else {
					optionName = optionText;
				}
				if (optionName.isEmpty() || (optionValue != null && optionValue.isEmpty())) {
					throw new IllegalArgumentException("Invalid argument syntax: " + arg);
				}
				commandLineArgs.addOptionArg(optionName, optionValue);
			}
			else {
				commandLineArgs.addNonOptionArg(arg);
			}
		}
		return commandLineArgs;
	}

}

```

##### SimpleCommandLinePropertySource.java

```java
package org.springframework.core.env;

import java.util.List;

import org.springframework.util.StringUtils;

/**
 * 简单命令刚属性源，简单封装命令行参数为选项参数和非选项参数
 */
public class SimpleCommandLinePropertySource extends CommandLinePropertySource<CommandLineArgs> {

	public SimpleCommandLinePropertySource(String... args) {
		super(new SimpleCommandLineArgsParser().parse(args));
	}

	public SimpleCommandLinePropertySource(String name, String[] args) {
		super(name, new SimpleCommandLineArgsParser().parse(args));
	}

	@Override
	public String[] getPropertyNames() {
		return StringUtils.toStringArray(this.source.getOptionNames());
	}

	@Override
	protected boolean containsOption(String name) {
		return this.source.containsOption(name);
	}

	@Override
	protected List<String> getOptionValues(String name) {
		return this.source.getOptionValues(name);
	}

	@Override
	protected List<String> getNonOptionArgs() {
		return this.source.getNonOptionArgs();
	}

}

```

##### JOptCommandLinePropertySource.java

```java
package org.springframework.core.env;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import joptsimple.OptionSet;
import joptsimple.OptionSpec;

import org.springframework.util.StringUtils;

/**
 * 使用jopt-simple解析命令行参数，需要对应jar支持
 */
public class JOptCommandLinePropertySource extends CommandLinePropertySource<OptionSet> {

	public JOptCommandLinePropertySource(OptionSet options) {
		super(options);
	}

	public JOptCommandLinePropertySource(String name, OptionSet options) {
		super(name, options);
	}


	@Override
	protected boolean containsOption(String name) {
		return this.source.has(name);
	}

	@Override
	public String[] getPropertyNames() {
		List<String> names = new ArrayList<String>();
		for (OptionSpec<?> spec : this.source.specs()) {
			List<String> aliases = spec.options();
			if (!aliases.isEmpty()) {
				// Only the longest name is used for enumerating
				names.add(aliases.get(aliases.size() - 1));
			}
		}
		return StringUtils.toStringArray(names);
	}

	@Override
	public List<String> getOptionValues(String name) {
		List<?> argValues = this.source.valuesOf(name);
		List<String> stringArgValues = new ArrayList<String>();
		for (Object argValue : argValues) {
			stringArgValues.add(argValue.toString());
		}
		if (stringArgValues.isEmpty()) {
			return (this.source.has(name) ? Collections.<String>emptyList() : null);
		}
		return Collections.unmodifiableList(stringArgValues);
	}

	@Override
	protected List<String> getNonOptionArgs() {
		List<?> argValues = this.source.nonOptionArguments();
		List<String> stringArgValues = new ArrayList<String>();
		for (Object argValue : argValues) {
			stringArgValues.add(argValue.toString());
		}
		return (stringArgValues.isEmpty() ? Collections.<String>emptyList() :
				Collections.unmodifiableList(stringArgValues));
	}

}

```

### CompositePropertySource.java

```java
package org.springframework.core.env;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collection;
import java.util.LinkedHashSet;
import java.util.List;
import java.util.Set;

import org.springframework.util.StringUtils;

/**
 * 组合属性源
 */
public class CompositePropertySource extends EnumerablePropertySource<Object> {

	private final Set<PropertySource<?>> propertySources = new LinkedHashSet<PropertySource<?>>();

	public CompositePropertySource(String name) {
		super(name);
	}


	@Override
	public Object getProperty(String name) {
		for (PropertySource<?> propertySource : this.propertySources) {
			Object candidate = propertySource.getProperty(name);
			if (candidate != null) {
				return candidate;
			}
		}
		return null;
	}

	@Override
	public boolean containsProperty(String name) {
		for (PropertySource<?> propertySource : this.propertySources) {
			if (propertySource.containsProperty(name)) {
				return true;
			}
		}
		return false;
	}

	@Override
	public String[] getPropertyNames() {
		Set<String> names = new LinkedHashSet<String>();
		for (PropertySource<?> propertySource : this.propertySources) {
			if (!(propertySource instanceof EnumerablePropertySource)) {
				throw new IllegalStateException(
						"Failed to enumerate property names due to non-enumerable property source: " + propertySource);
			}
			names.addAll(Arrays.asList(((EnumerablePropertySource<?>) propertySource).getPropertyNames()));
		}
		return StringUtils.toStringArray(names);
	}

	public void addPropertySource(PropertySource<?> propertySource) {
		this.propertySources.add(propertySource);
	}

	public void addFirstPropertySource(PropertySource<?> propertySource) {
		List<PropertySource<?>> existing = new ArrayList<PropertySource<?>>(this.propertySources);
		this.propertySources.clear();
		this.propertySources.add(propertySource);
		this.propertySources.addAll(existing);
	}

	public Collection<PropertySource<?>> getPropertySources() {
		return this.propertySources;
	}


	@Override
	public String toString() {
		return String.format("%s [name='%s', propertySources=%s]",
				getClass().getSimpleName(), this.name, this.propertySources);
	}

}

```



## PropertySources.java

```java
package org.springframework.core.env;

/**
 * 保存一个或者多个PropertySource
 */
public interface PropertySources extends Iterable<PropertySource<?>> {

	boolean contains(String name);

	PropertySource<?> get(String name);

}
```

### MutablePropertySources.class

```java
package org.springframework.core.env;

import java.util.Iterator;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

/**
 * PropertySources的默认实现，属性源需要保持顺序。定义了属性源的增删改查操作。
 */
public class MutablePropertySources implements PropertySources {

	private final Log logger;

	private final List<PropertySource<?>> propertySourceList = new CopyOnWriteArrayList<PropertySource<?>>();

	public MutablePropertySources() {
		this.logger = LogFactory.getLog(getClass());
	}

	public MutablePropertySources(PropertySources propertySources) {
		this();
		for (PropertySource<?> propertySource : propertySources) {
			addLast(propertySource);
		}
	}

	MutablePropertySources(Log logger) {
		this.logger = logger;
	}


	@Override
	public boolean contains(String name) {
		return this.propertySourceList.contains(PropertySource.named(name));
	}

	@Override
	public PropertySource<?> get(String name) {
		int index = this.propertySourceList.indexOf(PropertySource.named(name));
		return (index != -1 ? this.propertySourceList.get(index) : null);
	}

	@Override
	public Iterator<PropertySource<?>> iterator() {
		return this.propertySourceList.iterator();
	}

	public void addFirst(PropertySource<?> propertySource) {
		if (logger.isDebugEnabled()) {
			logger.debug("Adding PropertySource '" + propertySource.getName() + "' with highest search precedence");
		}
		removeIfPresent(propertySource);
		this.propertySourceList.add(0, propertySource);
	}

	public void addLast(PropertySource<?> propertySource) {
		if (logger.isDebugEnabled()) {
			logger.debug("Adding PropertySource '" + propertySource.getName() + "' with lowest search precedence");
		}
		removeIfPresent(propertySource);
		this.propertySourceList.add(propertySource);
	}

	public void addBefore(String relativePropertySourceName, PropertySource<?> propertySource) {
		if (logger.isDebugEnabled()) {
			logger.debug("Adding PropertySource '" + propertySource.getName() +
					"' with search precedence immediately higher than '" + relativePropertySourceName + "'");
		}
		assertLegalRelativeAddition(relativePropertySourceName, propertySource);
		removeIfPresent(propertySource);
		int index = assertPresentAndGetIndex(relativePropertySourceName);
		addAtIndex(index, propertySource);
	}

	public void addAfter(String relativePropertySourceName, PropertySource<?> propertySource) {
		if (logger.isDebugEnabled()) {
			logger.debug("Adding PropertySource '" + propertySource.getName() +
					"' with search precedence immediately lower than '" + relativePropertySourceName + "'");
		}
		assertLegalRelativeAddition(relativePropertySourceName, propertySource);
		removeIfPresent(propertySource);
		int index = assertPresentAndGetIndex(relativePropertySourceName);
		addAtIndex(index + 1, propertySource);
	}

	public int precedenceOf(PropertySource<?> propertySource) {
		return this.propertySourceList.indexOf(propertySource);
	}

	public PropertySource<?> remove(String name) {
		if (logger.isDebugEnabled()) {
			logger.debug("Removing PropertySource '" + name + "'");
		}
		int index = this.propertySourceList.indexOf(PropertySource.named(name));
		return (index != -1 ? this.propertySourceList.remove(index) : null);
	}

	public void replace(String name, PropertySource<?> propertySource) {
		if (logger.isDebugEnabled()) {
			logger.debug("Replacing PropertySource '" + name + "' with '" + propertySource.getName() + "'");
		}
		int index = assertPresentAndGetIndex(name);
		this.propertySourceList.set(index, propertySource);
	}

	public int size() {
		return this.propertySourceList.size();
	}

	@Override
	public String toString() {
		return this.propertySourceList.toString();
	}

	protected void assertLegalRelativeAddition(String relativePropertySourceName, PropertySource<?> propertySource) {
		String newPropertySourceName = propertySource.getName();
		if (relativePropertySourceName.equals(newPropertySourceName)) {
			throw new IllegalArgumentException(
					"PropertySource named '" + newPropertySourceName + "' cannot be added relative to itself");
		}
	}

	protected void removeIfPresent(PropertySource<?> propertySource) {
		this.propertySourceList.remove(propertySource);
	}

	private void addAtIndex(int index, PropertySource<?> propertySource) {
		removeIfPresent(propertySource);
		this.propertySourceList.add(index, propertySource);
	}

	private int assertPresentAndGetIndex(String name) {
		int index = this.propertySourceList.indexOf(PropertySource.named(name));
		if (index == -1) {
			throw new IllegalArgumentException("PropertySource named '" + name + "' does not exist");
		}
		return index;
	}

}
```

## PropertyResolver.class

```java
package org.springframework.core.env;

/**
 * 属性解析器，用于解析底层资源属性的接口
 */
public interface PropertyResolver {

	boolean containsProperty(String key);

	String getProperty(String key);
	String getProperty(String key, String defaultValue);
	<T> T getProperty(String key, Class<T> targetType);
	<T> T getProperty(String key, Class<T> targetType, T defaultValue);
	@Deprecated
	<T> Class<T> getPropertyAsClass(String key, Class<T> targetType);
	
	String getRequiredProperty(String key) throws IllegalStateException;
	<T> T getRequiredProperty(String key, Class<T> targetType) throws IllegalStateException;

	String resolvePlaceholders(String text);
	String resolveRequiredPlaceholders(String text) throws IllegalArgumentException;

}

```

### AbstractPropertyResolver.java

```java
package org.springframework.core.env;

import java.util.LinkedHashSet;
import java.util.Set;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import org.springframework.core.convert.ConversionService;
import org.springframework.core.convert.support.ConfigurableConversionService;
import org.springframework.core.convert.support.DefaultConversionService;
import org.springframework.util.Assert;
import org.springframework.util.ClassUtils;
import org.springframework.util.PropertyPlaceholderHelper;
import org.springframework.util.SystemPropertyUtils;

/**
 * 基于任何源解析属性的基础抽象实现
 */
public abstract class AbstractPropertyResolver implements ConfigurablePropertyResolver {

	protected final Log logger = LogFactory.getLog(getClass());
	private volatile ConfigurableConversionService conversionService;
	private PropertyPlaceholderHelper nonStrictHelper;
	private PropertyPlaceholderHelper strictHelper;
	private boolean ignoreUnresolvableNestedPlaceholders = false;
	private String placeholderPrefix = SystemPropertyUtils.PLACEHOLDER_PREFIX;
	private String placeholderSuffix = SystemPropertyUtils.PLACEHOLDER_SUFFIX;
	private String valueSeparator = SystemPropertyUtils.VALUE_SEPARATOR;
	private final Set<String> requiredProperties = new LinkedHashSet<String>();


	@Override
	public ConfigurableConversionService getConversionService() {
		if (this.conversionService == null) {
			synchronized (this) {
				if (this.conversionService == null) {
					this.conversionService = new DefaultConversionService();
				}
			}
		}
		return conversionService;
	}

	@Override
	public void setConversionService(ConfigurableConversionService conversionService) {
		Assert.notNull(conversionService, "ConversionService must not be null");
		this.conversionService = conversionService;
	}

	@Override
	public void setPlaceholderPrefix(String placeholderPrefix) {
		Assert.notNull(placeholderPrefix, "'placeholderPrefix' must not be null");
		this.placeholderPrefix = placeholderPrefix;
	}

	@Override
	public void setPlaceholderSuffix(String placeholderSuffix) {
		Assert.notNull(placeholderSuffix, "'placeholderSuffix' must not be null");
		this.placeholderSuffix = placeholderSuffix;
	}

	@Override
	public void setValueSeparator(String valueSeparator) {
		this.valueSeparator = valueSeparator;
	}

	@Override
	public void setIgnoreUnresolvableNestedPlaceholders(boolean ignoreUnresolvableNestedPlaceholders) {
		this.ignoreUnresolvableNestedPlaceholders = ignoreUnresolvableNestedPlaceholders;
	}

	@Override
	public void setRequiredProperties(String... requiredProperties) {
		if (requiredProperties != null) {
			for (String key : requiredProperties) {
				this.requiredProperties.add(key);
			}
		}
	}

	@Override
	public void validateRequiredProperties() {
		MissingRequiredPropertiesException ex = new MissingRequiredPropertiesException();
		for (String key : this.requiredProperties) {
			if (this.getProperty(key) == null) {
				ex.addMissingRequiredProperty(key);
			}
		}
		if (!ex.getMissingRequiredProperties().isEmpty()) {
			throw ex;
		}
	}

	@Override
	public boolean containsProperty(String key) {
		return (getProperty(key) != null);
	}

	@Override
	public String getProperty(String key) {
		return getProperty(key, String.class);
	}

	@Override
	public String getProperty(String key, String defaultValue) {
		String value = getProperty(key);
		return (value != null ? value : defaultValue);
	}

	@Override
	public <T> T getProperty(String key, Class<T> targetType, T defaultValue) {
		T value = getProperty(key, targetType);
		return (value != null ? value : defaultValue);
	}

	@Override
	@Deprecated
	public <T> Class<T> getPropertyAsClass(String key, Class<T> targetValueType) {
		throw new UnsupportedOperationException();
	}

	@Override
	public String getRequiredProperty(String key) throws IllegalStateException {
		String value = getProperty(key);
		if (value == null) {
			throw new IllegalStateException("Required key '" + key + "' not found");
		}
		return value;
	}

	@Override
	public <T> T getRequiredProperty(String key, Class<T> valueType) throws IllegalStateException {
		T value = getProperty(key, valueType);
		if (value == null) {
			throw new IllegalStateException("Required key '" + key + "' not found");
		}
		return value;
	}

	@Override
	public String resolvePlaceholders(String text) {
		if (this.nonStrictHelper == null) {
			this.nonStrictHelper = createPlaceholderHelper(true);
		}
		return doResolvePlaceholders(text, this.nonStrictHelper);
	}

	@Override
	public String resolveRequiredPlaceholders(String text) throws IllegalArgumentException {
		if (this.strictHelper == null) {
			this.strictHelper = createPlaceholderHelper(false);
		}
		return doResolvePlaceholders(text, this.strictHelper);
	}

	protected String resolveNestedPlaceholders(String value) {
		return (this.ignoreUnresolvableNestedPlaceholders ?
				resolvePlaceholders(value) : resolveRequiredPlaceholders(value));
	}

	private PropertyPlaceholderHelper createPlaceholderHelper(boolean ignoreUnresolvablePlaceholders) {
		return new PropertyPlaceholderHelper(this.placeholderPrefix, this.placeholderSuffix,
				this.valueSeparator, ignoreUnresolvablePlaceholders);
	}

	private String doResolvePlaceholders(String text, PropertyPlaceholderHelper helper) {
		return helper.replacePlaceholders(text, new PropertyPlaceholderHelper.PlaceholderResolver() {
			@Override
			public String resolvePlaceholder(String placeholderName) {
				return getPropertyAsRawString(placeholderName);
			}
		});
	}

	@SuppressWarnings("unchecked")
	protected <T> T convertValueIfNecessary(Object value, Class<T> targetType) {
		if (targetType == null) {
			return (T) value;
		}
		ConversionService conversionServiceToUse = this.conversionService;
		if (conversionServiceToUse == null) {
			// Avoid initialization of shared DefaultConversionService if
			// no standard type conversion is needed in the first place...
			if (ClassUtils.isAssignableValue(targetType, value)) {
				return (T) value;
			}
			conversionServiceToUse = DefaultConversionService.getSharedInstance();
		}
		return conversionServiceToUse.convert(value, targetType);
	}

	protected abstract String getPropertyAsRawString(String key);

}

```

#### PropertySourcesPropertyResolver.java

```java
package org.springframework.core.env;

import org.springframework.core.convert.ConversionException;
import org.springframework.util.ClassUtils;

/**
 * 使用PropertySources解析属性
 */
public class PropertySourcesPropertyResolver extends AbstractPropertyResolver {

	private final PropertySources propertySources;

	public PropertySourcesPropertyResolver(PropertySources propertySources) {
		this.propertySources = propertySources;
	}


	@Override
	public boolean containsProperty(String key) {
		if (this.propertySources != null) {
			for (PropertySource<?> propertySource : this.propertySources) {
				if (propertySource.containsProperty(key)) {
					return true;
				}
			}
		}
		return false;
	}

	@Override
	public String getProperty(String key) {
		return getProperty(key, String.class, true);
	}

	@Override
	public <T> T getProperty(String key, Class<T> targetValueType) {
		return getProperty(key, targetValueType, true);
	}

	@Override
	protected String getPropertyAsRawString(String key) {
		return getProperty(key, String.class, false);
	}

	protected <T> T getProperty(String key, Class<T> targetValueType, boolean resolveNestedPlaceholders) {
		if (this.propertySources != null) {
			for (PropertySource<?> propertySource : this.propertySources) {
				if (logger.isTraceEnabled()) {
					logger.trace("Searching for key '" + key + "' in PropertySource '" +
							propertySource.getName() + "'");
				}
				Object value = propertySource.getProperty(key);
				if (value != null) {
					if (resolveNestedPlaceholders && value instanceof String) {
						value = resolveNestedPlaceholders((String) value);
					}
					logKeyFound(key, propertySource, value);
					return convertValueIfNecessary(value, targetValueType);
				}
			}
		}
		if (logger.isDebugEnabled()) {
			logger.debug("Could not find key '" + key + "' in any property source");
		}
		return null;
	}

	@Override
	@Deprecated
	public <T> Class<T> getPropertyAsClass(String key, Class<T> targetValueType) {
		if (this.propertySources != null) {
			for (PropertySource<?> propertySource : this.propertySources) {
				if (logger.isTraceEnabled()) {
					logger.trace(String.format("Searching for key '%s' in [%s]", key, propertySource.getName()));
				}
				Object value = propertySource.getProperty(key);
				if (value != null) {
					logKeyFound(key, propertySource, value);
					Class<?> clazz;
					if (value instanceof String) {
						try {
							clazz = ClassUtils.forName((String) value, null);
						}
						catch (Exception ex) {
							throw new ClassConversionException((String) value, targetValueType, ex);
						}
					}
					else if (value instanceof Class) {
						clazz = (Class<?>) value;
					}
					else {
						clazz = value.getClass();
					}
					if (!targetValueType.isAssignableFrom(clazz)) {
						throw new ClassConversionException(clazz, targetValueType);
					}
					@SuppressWarnings("unchecked")
					Class<T> targetClass = (Class<T>) clazz;
					return targetClass;
				}
			}
		}
		if (logger.isDebugEnabled()) {
			logger.debug(String.format("Could not find key '%s' in any property source", key));
		}
		return null;
	}

	protected void logKeyFound(String key, PropertySource<?> propertySource, Object value) {
		if (logger.isDebugEnabled()) {
			logger.debug("Found key '" + key + "' in PropertySource '" + propertySource.getName() +
					"' with value of type " + value.getClass().getSimpleName());
		}
	}


	@SuppressWarnings("serial")
	@Deprecated
	private static class ClassConversionException extends ConversionException {

		public ClassConversionException(Class<?> actual, Class<?> expected) {
			super(String.format("Actual type %s is not assignable to expected type %s",
					actual.getName(), expected.getName()));
		}

		public ClassConversionException(String actual, Class<?> expected, Exception ex) {
			super(String.format("Could not find/load class %s during attempt to convert to %s",
					actual, expected.getName()), ex);
		}
	}

}

```



### ConfigurablePropertyResolver.java

```java
package org.springframework.core.env;

import org.springframework.core.convert.support.ConfigurableConversionService;

/**
 * 可配置的属性解析器
 */
public interface ConfigurablePropertyResolver extends PropertyResolver {
	//配置转换服务
	ConfigurableConversionService getConversionService();
	void setConversionService(ConfigurableConversionService conversionService);
	//设置占位符前后缀
	void setPlaceholderPrefix(String placeholderPrefix);
	void setPlaceholderSuffix(String placeholderSuffix);
	//设置分隔符
	void setValueSeparator(String valueSeparator);
  	//设置未解决的内嵌占位符
	void setIgnoreUnresolvableNestedPlaceholders(boolean ignoreUnresolvableNestedPlaceholders);
  	//设置必须的属性
	void setRequiredProperties(String... requiredProperties);
  	//验证必须的属性
	void validateRequiredProperties() throws MissingRequiredPropertiesException;

}

```

### Environment.java

```java
package org.springframework.core.env;

/**
 * 应用程序的环境，包括配置文件和属性文件两个方面
 */
public interface Environment extends PropertyResolver {

	//获取激活的配置
	String[] getActiveProfiles();
  	//获取默认的配置
	String[] getDefaultProfiles();
  	//是否可以接受指定的配置
	boolean acceptsProfiles(String... profiles);

}

```

#### ConfigurableEnvironment.java

```java
package org.springframework.core.env;

import java.util.Map;

/**
 * 可配置的环境
 */
public interface ConfigurableEnvironment extends Environment, ConfigurablePropertyResolver {


	void setActiveProfiles(String... profiles);
	void addActiveProfile(String profile);
	void setDefaultProfiles(String... profiles);
	MutablePropertySources getPropertySources();
	Map<String, Object> getSystemEnvironment();
	Map<String, Object> getSystemProperties();
	void merge(ConfigurableEnvironment parent);

}

```

##### AbstractEnvironment.java

```java
package org.springframework.core.env;

import java.security.AccessControlException;
import java.util.Arrays;
import java.util.Collections;
import java.util.LinkedHashSet;
import java.util.Map;
import java.util.Set;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import org.springframework.core.SpringProperties;
import org.springframework.core.convert.support.ConfigurableConversionService;
import org.springframework.util.Assert;
import org.springframework.util.ObjectUtils;
import org.springframework.util.StringUtils;

/**
 * Environment的基本抽象实现,子类主要通过customizePropertySources自定义属性源
 *
 */
public abstract class AbstractEnvironment implements ConfigurableEnvironment {

	public static final String IGNORE_GETENV_PROPERTY_NAME = "spring.getenv.ignore";
	public static final String ACTIVE_PROFILES_PROPERTY_NAME = "spring.profiles.active";
	public static final String DEFAULT_PROFILES_PROPERTY_NAME = "spring.profiles.default";
	protected static final String RESERVED_DEFAULT_PROFILE_NAME = "default";


	protected final Log logger = LogFactory.getLog(getClass());
	private final Set<String> activeProfiles = new LinkedHashSet<String>();
	private final Set<String> defaultProfiles = new LinkedHashSet<String>(getReservedDefaultProfiles());
	private final MutablePropertySources propertySources = new MutablePropertySources(this.logger);
	private final ConfigurablePropertyResolver propertyResolver =
			new PropertySourcesPropertyResolver(this.propertySources);

	public AbstractEnvironment() {
		customizePropertySources(this.propertySources);
		if (logger.isDebugEnabled()) {
			logger.debug("Initialized " + getClass().getSimpleName() + " with PropertySources " + this.propertySources);
		}
	}

	protected void customizePropertySources(MutablePropertySources propertySources) {
	}

	protected Set<String> getReservedDefaultProfiles() {
		return Collections.singleton(RESERVED_DEFAULT_PROFILE_NAME);
	}

	@Override
	public String[] getActiveProfiles() {
		return StringUtils.toStringArray(doGetActiveProfiles());
	}

	protected Set<String> doGetActiveProfiles() {
		synchronized (this.activeProfiles) {
			if (this.activeProfiles.isEmpty()) {
				String profiles = getProperty(ACTIVE_PROFILES_PROPERTY_NAME);
				if (StringUtils.hasText(profiles)) {
					setActiveProfiles(StringUtils.commaDelimitedListToStringArray(
							StringUtils.trimAllWhitespace(profiles)));
				}
			}
			return this.activeProfiles;
		}
	}

	@Override
	public void setActiveProfiles(String... profiles) {
		Assert.notNull(profiles, "Profile array must not be null");
		if (logger.isDebugEnabled()) {
			logger.debug("Activating profiles " + Arrays.asList(profiles));
		}
		synchronized (this.activeProfiles) {
			this.activeProfiles.clear();
			for (String profile : profiles) {
				validateProfile(profile);
				this.activeProfiles.add(profile);
			}
		}
	}

	@Override
	public void addActiveProfile(String profile) {
		if (logger.isDebugEnabled()) {
			logger.debug("Activating profile '" + profile + "'");
		}
		validateProfile(profile);
		doGetActiveProfiles();
		synchronized (this.activeProfiles) {
			this.activeProfiles.add(profile);
		}
	}


	@Override
	public String[] getDefaultProfiles() {
		return StringUtils.toStringArray(doGetDefaultProfiles());
	}

	protected Set<String> doGetDefaultProfiles() {
		synchronized (this.defaultProfiles) {
			if (this.defaultProfiles.equals(getReservedDefaultProfiles())) {
				String profiles = getProperty(DEFAULT_PROFILES_PROPERTY_NAME);
				if (StringUtils.hasText(profiles)) {
					setDefaultProfiles(StringUtils.commaDelimitedListToStringArray(
							StringUtils.trimAllWhitespace(profiles)));
				}
			}
			return this.defaultProfiles;
		}
	}

	@Override
	public void setDefaultProfiles(String... profiles) {
		Assert.notNull(profiles, "Profile array must not be null");
		synchronized (this.defaultProfiles) {
			this.defaultProfiles.clear();
			for (String profile : profiles) {
				validateProfile(profile);
				this.defaultProfiles.add(profile);
			}
		}
	}

	@Override
	public boolean acceptsProfiles(String... profiles) {
		Assert.notEmpty(profiles, "Must specify at least one profile");
		for (String profile : profiles) {
			if (StringUtils.hasLength(profile) && profile.charAt(0) == '!') {
				if (!isProfileActive(profile.substring(1))) {
					return true;
				}
			}
			else if (isProfileActive(profile)) {
				return true;
			}
		}
		return false;
	}

	protected boolean isProfileActive(String profile) {
		validateProfile(profile);
		Set<String> currentActiveProfiles = doGetActiveProfiles();
		return (currentActiveProfiles.contains(profile) ||
				(currentActiveProfiles.isEmpty() && doGetDefaultProfiles().contains(profile)));
	}

	protected void validateProfile(String profile) {
		if (!StringUtils.hasText(profile)) {
			throw new IllegalArgumentException("Invalid profile [" + profile + "]: must contain text");
		}
		if (profile.charAt(0) == '!') {
			throw new IllegalArgumentException("Invalid profile [" + profile + "]: must not begin with ! operator");
		}
	}

	@Override
	public MutablePropertySources getPropertySources() {
		return this.propertySources;
	}

	@Override
	@SuppressWarnings({"unchecked", "rawtypes"})
	public Map<String, Object> getSystemEnvironment() {
		if (suppressGetenvAccess()) {
			return Collections.emptyMap();
		}
		try {
			return (Map) System.getenv();
		}
		catch (AccessControlException ex) {
			return (Map) new ReadOnlySystemAttributesMap() {
				@Override
				protected String getSystemAttribute(String attributeName) {
					try {
						return System.getenv(attributeName);
					}
					catch (AccessControlException ex) {
						if (logger.isInfoEnabled()) {
							logger.info("Caught AccessControlException when accessing system environment variable '" +
									attributeName + "'; its value will be returned [null]. Reason: " + ex.getMessage());
						}
						return null;
					}
				}
			};
		}
	}

	protected boolean suppressGetenvAccess() {
		return SpringProperties.getFlag(IGNORE_GETENV_PROPERTY_NAME);
	}

	@Override
	@SuppressWarnings({"unchecked", "rawtypes"})
	public Map<String, Object> getSystemProperties() {
		try {
			return (Map) System.getProperties();
		}
		catch (AccessControlException ex) {
			return (Map) new ReadOnlySystemAttributesMap() {
				@Override
				protected String getSystemAttribute(String attributeName) {
					try {
						return System.getProperty(attributeName);
					}
					catch (AccessControlException ex) {
						if (logger.isInfoEnabled()) {
							logger.info("Caught AccessControlException when accessing system property '" +
									attributeName + "'; its value will be returned [null]. Reason: " + ex.getMessage());
						}
						return null;
					}
				}
			};
		}
	}

	@Override
	public void merge(ConfigurableEnvironment parent) {
		for (PropertySource<?> ps : parent.getPropertySources()) {
			if (!this.propertySources.contains(ps.getName())) {
				this.propertySources.addLast(ps);
			}
		}
		String[] parentActiveProfiles = parent.getActiveProfiles();
		if (!ObjectUtils.isEmpty(parentActiveProfiles)) {
			synchronized (this.activeProfiles) {
				for (String profile : parentActiveProfiles) {
					this.activeProfiles.add(profile);
				}
			}
		}
		String[] parentDefaultProfiles = parent.getDefaultProfiles();
		if (!ObjectUtils.isEmpty(parentDefaultProfiles)) {
			synchronized (this.defaultProfiles) {
				this.defaultProfiles.remove(RESERVED_DEFAULT_PROFILE_NAME);
				for (String profile : parentDefaultProfiles) {
					this.defaultProfiles.add(profile);
				}
			}
		}
	}

	@Override
	public ConfigurableConversionService getConversionService() {
		return this.propertyResolver.getConversionService();
	}

	@Override
	public void setConversionService(ConfigurableConversionService conversionService) {
		this.propertyResolver.setConversionService(conversionService);
	}

	@Override
	public void setPlaceholderPrefix(String placeholderPrefix) {
		this.propertyResolver.setPlaceholderPrefix(placeholderPrefix);
	}

	@Override
	public void setPlaceholderSuffix(String placeholderSuffix) {
		this.propertyResolver.setPlaceholderSuffix(placeholderSuffix);
	}

	@Override
	public void setValueSeparator(String valueSeparator) {
		this.propertyResolver.setValueSeparator(valueSeparator);
	}

	@Override
	public void setIgnoreUnresolvableNestedPlaceholders(boolean ignoreUnresolvableNestedPlaceholders) {
		this.propertyResolver.setIgnoreUnresolvableNestedPlaceholders(ignoreUnresolvableNestedPlaceholders);
	}

	@Override
	public void setRequiredProperties(String... requiredProperties) {
		this.propertyResolver.setRequiredProperties(requiredProperties);
	}

	@Override
	public void validateRequiredProperties() throws MissingRequiredPropertiesException {
		this.propertyResolver.validateRequiredProperties();
	}

	@Override
	public boolean containsProperty(String key) {
		return this.propertyResolver.containsProperty(key);
	}

	@Override
	public String getProperty(String key) {
		return this.propertyResolver.getProperty(key);
	}

	@Override
	public String getProperty(String key, String defaultValue) {
		return this.propertyResolver.getProperty(key, defaultValue);
	}

	@Override
	public <T> T getProperty(String key, Class<T> targetType) {
		return this.propertyResolver.getProperty(key, targetType);
	}

	@Override
	public <T> T getProperty(String key, Class<T> targetType, T defaultValue) {
		return this.propertyResolver.getProperty(key, targetType, defaultValue);
	}

	@Override
	@Deprecated
	public <T> Class<T> getPropertyAsClass(String key, Class<T> targetType) {
		return this.propertyResolver.getPropertyAsClass(key, targetType);
	}

	@Override
	public String getRequiredProperty(String key) throws IllegalStateException {
		return this.propertyResolver.getRequiredProperty(key);
	}

	@Override
	public <T> T getRequiredProperty(String key, Class<T> targetType) throws IllegalStateException {
		return this.propertyResolver.getRequiredProperty(key, targetType);
	}

	@Override
	public String resolvePlaceholders(String text) {
		return this.propertyResolver.resolvePlaceholders(text);
	}

	@Override
	public String resolveRequiredPlaceholders(String text) throws IllegalArgumentException {
		return this.propertyResolver.resolveRequiredPlaceholders(text);
	}


	@Override
	public String toString() {
		return getClass().getSimpleName() + " {activeProfiles=" + this.activeProfiles +
				", defaultProfiles=" + this.defaultProfiles + ", propertySources=" + this.propertySources + "}";
	}

}
```

###### StandardEnvironment.java

```java
package org.springframework.core.env;

/**
 * 标准环境，使用 System.getenv()和System.getProperties()作为属性源
 */
public class StandardEnvironment extends AbstractEnvironment {

	public static final String SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME = "systemEnvironment";
	public static final String SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME = "systemProperties";

	@Override
	protected void customizePropertySources(MutablePropertySources propertySources) {
		propertySources.addLast(new MapPropertySource(SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME, getSystemProperties()));
		propertySources.addLast(new SystemEnvironmentPropertySource(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, getSystemEnvironment()));
	}

}

```

## MissingRequiredPropertiesException.java

```java
/*
 * Copyright 2002-2017 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.core.env;

import java.util.LinkedHashSet;
import java.util.Set;

/**
 * 缺少必须的属性异常
 */
@SuppressWarnings("serial")
public class MissingRequiredPropertiesException extends IllegalStateException {

	private final Set<String> missingRequiredProperties = new LinkedHashSet<String>();


	void addMissingRequiredProperty(String key) {
		this.missingRequiredProperties.add(key);
	}

	@Override
	public String getMessage() {
		return "The following properties were declared as required but could not be resolved: " +
				getMissingRequiredProperties();
	}

	public Set<String> getMissingRequiredProperties() {
		return this.missingRequiredProperties;
	}

}

```



## ReadOnlySystemAttributesMap.java

```java
package org.springframework.core.env;

import java.util.Collection;
import java.util.Collections;
import java.util.Map;
import java.util.Set;

/**
 * 只读Map实现，AbstractEnvironment内部使用
 */
abstract class ReadOnlySystemAttributesMap implements Map<String, String> {

	@Override
	public boolean containsKey(Object key) {
		return (get(key) != null);
	}

	@Override
	public String get(Object key) {
		if (!(key instanceof String)) {
			throw new IllegalArgumentException(
					"Type of key [" + key.getClass().getName() + "] must be java.lang.String");
		}
		return getSystemAttribute((String) key);
	}

	@Override
	public boolean isEmpty() {
		return false;
	}

	protected abstract String getSystemAttribute(String attributeName);


	@Override
	public int size() {
		throw new UnsupportedOperationException();
	}

	@Override
	public String put(String key, String value) {
		throw new UnsupportedOperationException();
	}

	@Override
	public boolean containsValue(Object value) {
		throw new UnsupportedOperationException();
	}

	@Override
	public String remove(Object key) {
		throw new UnsupportedOperationException();
	}

	@Override
	public void clear() {
		throw new UnsupportedOperationException();
	}

	@Override
	public Set<String> keySet() {
		return Collections.emptySet();
	}

	@Override
	public void putAll(Map<? extends String, ? extends String> map) {
		throw new UnsupportedOperationException();
	}

	@Override
	public Collection<String> values() {
		return Collections.emptySet();
	}

	@Override
	public Set<Entry<String, String>> entrySet() {
		return Collections.emptySet();
	}

}

```

## EnvironmentCapable.java

```java
package org.springframework.core.env;

public interface EnvironmentCapable {

	Environment getEnvironment();

}

```

# 系统环境

## System.getProperties()

```java
[java.runtime.name, sun.boot.library.path, java.vm.version, java.vm.vendor, java.vendor.url, path.separator, java.vm.name, file.encoding.pkg, user.country, user.script, sun.java.launcher, sun.os.patch.level, java.vm.specification.name, user.dir, java.runtime.version, java.awt.graphicsenv, java.endorsed.dirs, os.arch, java.io.tmpdir, line.separator, java.vm.specification.vendor, user.variant, os.name, sun.jnu.encoding, java.library.path, java.specification.name, java.class.version, sun.management.compiler, os.version, user.home, user.timezone, java.awt.printerjob, file.encoding, java.specification.version, java.class.path, user.name, java.vm.specification.version, sun.java.command, java.home, sun.arch.data.model, user.language, java.specification.vendor, awt.toolkit, java.vm.info, java.version, java.ext.dirs, sun.boot.class.path, java.vendor, file.separator, java.vendor.url.bug, sun.io.unicode.encoding, sun.cpu.endian, sun.desktop, sun.cpu.isalist]
```

```java
java.version Java ：运行时环境版本
java.vendor Java ：运行时环境供应商
java.vendor.url ：Java供应商的 URL
java.home &nbsp;&nbsp;：Java安装目录
java.vm.specification.version： Java虚拟机规范版本
java.vm.specification.vendor ：Java虚拟机规范供应商
java.vm.specification.name &nbsp; ：Java虚拟机规范名称
java.vm.version ：Java虚拟机实现版本
java.vm.vendor ：Java虚拟机实现供应商
java.vm.name&nbsp; ：Java虚拟机实现名称
java.specification.version：Java运行时环境规范版本
java.specification.vendor：Java运行时环境规范供应商
java.specification.name ：Java运行时环境规范名称
java.class.version ：Java类格式版本号
java.class.path ：Java类路径
java.library.path  ：加载库时搜索的路径列表
java.io.tmpdir  ：默认的临时文件路径
java.compiler  ：要使用的 JIT编译器的名称
java.ext.dirs ：一个或多个扩展目录的路径
os.name ：操作系统的名称
os.arch  ：操作系统的架构
os.version  ：操作系统的版本
file.separator ：文件分隔符
path.separator ：路径分隔符
line.separator ：行分隔符
user.name ：用户的账户名称
user.home ：用户的主目录
user.dir：用户的当前工作目录
```

## System.getenv()

```java
[LOCALAPPDATA, PROCESSOR_LEVEL, FP_NO_HOST_CHECK, USERDOMAIN, LOGONSERVER, JAVA_HOME, SESSIONNAME, ALLUSERSPROFILE, PROCESSOR_ARCHITECTURE, PSModulePath, SystemDrive, APPDATA, USERNAME, windows_tracing_logfile, ProgramFiles(x86), CommonProgramFiles, Path, PATHEXT, OS, windows_tracing_flags, COMPUTERNAME, GOPATH, PROCESSOR_REVISION, CommonProgramW6432, GOROOT, ComSpec, DEVMGR_SHOW_DETAILS, ProgramData, ProgramW6432, HOMEPATH, SystemRoot, TEMP, HOMEDRIVE, PROCESSOR_IDENTIFIER, USERPROFILE, TMP, CommonProgramFiles(x86), ProgramFiles, PUBLIC, NUMBER_OF_PROCESSORS, windir, =::]
```



```java
USERPROFILE        ：用户目录
USERDNSDOMAIN      ：用户域
PATHEXT            ：可执行后缀
JAVA_HOME          ：Java安装目录
TEMP               ：用户临时文件目录
SystemDrive        ：系统盘符
ProgramFiles       ：默认程序目录
USERDOMAIN         ：帐户的域的名称
ALLUSERSPROFILE    ：用户公共目录
SESSIONNAME        ：Session名称
TMP                ：临时目录
Path               ：path环境变量
CLASSPATH          ：classpath环境变量
PROCESSOR_ARCHITECTURE ：处理器体系结构
OS                     ：操作系统类型
PROCESSOR_LEVEL    ：处理级别
COMPUTERNAME       ：计算机名
Windir             ：系统安装目录
SystemRoot         ：系统启动目录
USERNAME           ：用户名
ComSpec            ：命令行解释器可执行程序的准确路径
APPDATA            ：应用程序数据目录
```


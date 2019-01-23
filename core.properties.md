# Properties

## PropertiesPersister.java

```java
package org.springframework.util;

public interface PropertiesPersister {

	void load(Properties props, InputStream is) throws IOException;

	void load(Properties props, Reader reader) throws IOException;

	void store(Properties props, OutputStream os, String header) throws IOException;

	void store(Properties props, Writer writer, String header) throws IOException;

	void loadFromXml(Properties props, InputStream is) throws IOException;

	void storeToXml(Properties props, OutputStream os, String header) throws IOException;

	void storeToXml(Properties props, OutputStream os, String header, String encoding) throws IOException;

}

```

### DefaultPropertiesPersister.java

```java
package org.springframework.util;

public class DefaultPropertiesPersister implements PropertiesPersister {

	@Override
	public void load(Properties props, InputStream is) throws IOException {
		props.load(is);
	}

	@Override
	public void load(Properties props, Reader reader) throws IOException {
		props.load(reader);
	}

	@Override
	public void store(Properties props, OutputStream os, String header) throws IOException {
		props.store(os, header);
	}

	@Override
	public void store(Properties props, Writer writer, String header) throws IOException {
		props.store(writer, header);
	}

	@Override
	public void loadFromXml(Properties props, InputStream is) throws IOException {
		props.loadFromXML(is);
	}

	@Override
	public void storeToXml(Properties props, OutputStream os, String header) throws IOException {
		props.storeToXML(os, header);
	}

	@Override
	public void storeToXml(Properties props, OutputStream os, String header, String encoding) throws IOException {
		props.storeToXML(os, header, encoding);
	}

}

```



## PropertiesLoaderSupport.java

```java
package org.springframework.core.io.support;

public abstract class PropertiesLoaderSupport {

	protected final Log logger = LogFactory.getLog(getClass());

	protected Properties[] localProperties;

	protected boolean localOverride = false;

	private Resource[] locations;

	private boolean ignoreResourceNotFound = false;

	private String fileEncoding;

	private PropertiesPersister propertiesPersister = new DefaultPropertiesPersister();

	public void setProperties(Properties properties) {
		this.localProperties = new Properties[] {properties};
	}

	public void setPropertiesArray(Properties... propertiesArray) {
		this.localProperties = propertiesArray;
	}

	public void setLocation(Resource location) {
		this.locations = new Resource[] {location};
	}

	public void setLocations(Resource... locations) {
		this.locations = locations;
	}

	public void setLocalOverride(boolean localOverride) {
		this.localOverride = localOverride;
	}

	public void setIgnoreResourceNotFound(boolean ignoreResourceNotFound) {
		this.ignoreResourceNotFound = ignoreResourceNotFound;
	}

	public void setFileEncoding(String encoding) {
		this.fileEncoding = encoding;
	}

	public void setPropertiesPersister(PropertiesPersister propertiesPersister) {
		this.propertiesPersister =
				(propertiesPersister != null ? propertiesPersister : new DefaultPropertiesPersister());
	}

	protected Properties mergeProperties() throws IOException {
		Properties result = new Properties();

		if (this.localOverride) {
			// Load properties from file upfront, to let local properties override.
			loadProperties(result);
		}

		if (this.localProperties != null) {
			for (Properties localProp : this.localProperties) {
				CollectionUtils.mergePropertiesIntoMap(localProp, result);
			}
		}

		if (!this.localOverride) {
			// Load properties from file afterwards, to let those properties override.
			loadProperties(result);
		}

		return result;
	}

	protected void loadProperties(Properties props) throws IOException {
		if (this.locations != null) {
			for (Resource location : this.locations) {
				if (logger.isDebugEnabled()) {
					logger.debug("Loading properties file from " + location);
				}
				try {
					PropertiesLoaderUtils.fillProperties(
							props, new EncodedResource(location, this.fileEncoding), this.propertiesPersister);
				}
				catch (IOException ex) {
					// Resource not found when trying to open it
					if (this.ignoreResourceNotFound &&
							(ex instanceof FileNotFoundException || ex instanceof UnknownHostException)) {
						if (logger.isInfoEnabled()) {
							logger.info("Properties resource not found: " + ex.getMessage());
						}
					}
					else {
						throw ex;
					}
				}
			}
		}
	}

}

```

### PropertiesFactoryBean.java

```java
package org.springframework.beans.factory.config;

public class PropertiesFactoryBean extends PropertiesLoaderSupport
		implements FactoryBean<Properties>, InitializingBean {

	private boolean singleton = true;

	private Properties singletonInstance;

	public final void setSingleton(boolean singleton) {
		this.singleton = singleton;
	}

	@Override
	public final boolean isSingleton() {
		return this.singleton;
	}


	@Override
	public final void afterPropertiesSet() throws IOException {
		if (this.singleton) {
			this.singletonInstance = createProperties();
		}
	}

	@Override
	public final Properties getObject() throws IOException {
		if (this.singleton) {
			return this.singletonInstance;
		}
		else {
			return createProperties();
		}
	}

	@Override
	public Class<Properties> getObjectType() {
		return Properties.class;
	}

	protected Properties createProperties() throws IOException {
		return mergeProperties();
	}

}

```

#### PropertyResourceConfigurer.java

```java
package org.springframework.beans.factory.config;

public abstract class PropertyResourceConfigurer extends PropertiesLoaderSupport
		implements BeanFactoryPostProcessor, PriorityOrdered {

	private int order = Ordered.LOWEST_PRECEDENCE;  // default: same as non-Ordered

	public void setOrder(int order) {
		this.order = order;
	}

	@Override
	public int getOrder() {
		return this.order;
	}

	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		try {
			Properties mergedProps = mergeProperties();

			// Convert the merged properties, if necessary.
			convertProperties(mergedProps);

			// Let the subclass process the properties.
			processProperties(beanFactory, mergedProps);
		}
		catch (IOException ex) {
			throw new BeanInitializationException("Could not load properties", ex);
		}
	}

	protected void convertProperties(Properties props) {
		Enumeration<?> propertyNames = props.propertyNames();
		while (propertyNames.hasMoreElements()) {
			String propertyName = (String) propertyNames.nextElement();
			String propertyValue = props.getProperty(propertyName);
			String convertedValue = convertProperty(propertyName, propertyValue);
			if (!ObjectUtils.nullSafeEquals(propertyValue, convertedValue)) {
				props.setProperty(propertyName, convertedValue);
			}
		}
	}

	protected String convertProperty(String propertyName, String propertyValue) {
		return convertPropertyValue(propertyValue);
	}

	protected String convertPropertyValue(String originalValue) {
		return originalValue;
	}

	protected abstract void processProperties(ConfigurableListableBeanFactory beanFactory, Properties props)
			throws BeansException;

}

```

##### PlaceholderConfigurerSupport.java

```java
package org.springframework.beans.factory.config;

public abstract class PlaceholderConfigurerSupport extends PropertyResourceConfigurer
		implements BeanNameAware, BeanFactoryAware {

	/** Default placeholder prefix: {@value} */
	public static final String DEFAULT_PLACEHOLDER_PREFIX = "${";

	/** Default placeholder suffix: {@value} */
	public static final String DEFAULT_PLACEHOLDER_SUFFIX = "}";

	/** Default value separator: {@value} */
	public static final String DEFAULT_VALUE_SEPARATOR = ":";


	/** Defaults to {@value #DEFAULT_PLACEHOLDER_PREFIX} */
	protected String placeholderPrefix = DEFAULT_PLACEHOLDER_PREFIX;

	/** Defaults to {@value #DEFAULT_PLACEHOLDER_SUFFIX} */
	protected String placeholderSuffix = DEFAULT_PLACEHOLDER_SUFFIX;

	/** Defaults to {@value #DEFAULT_VALUE_SEPARATOR} */
	protected String valueSeparator = DEFAULT_VALUE_SEPARATOR;

	protected boolean trimValues = false;

	protected String nullValue;

	protected boolean ignoreUnresolvablePlaceholders = false;

	private String beanName;

	private BeanFactory beanFactory;

	public void setPlaceholderPrefix(String placeholderPrefix) {
		this.placeholderPrefix = placeholderPrefix;
	}

	public void setPlaceholderSuffix(String placeholderSuffix) {
		this.placeholderSuffix = placeholderSuffix;
	}

	public void setValueSeparator(String valueSeparator) {
		this.valueSeparator = valueSeparator;
	}

	public void setTrimValues(boolean trimValues) {
		this.trimValues = trimValues;
	}

	public void setNullValue(String nullValue) {
		this.nullValue = nullValue;
	}

	public void setIgnoreUnresolvablePlaceholders(boolean ignoreUnresolvablePlaceholders) {
		this.ignoreUnresolvablePlaceholders = ignoreUnresolvablePlaceholders;
	}

	@Override
	public void setBeanName(String beanName) {
		this.beanName = beanName;
	}

	@Override
	public void setBeanFactory(BeanFactory beanFactory) {
		this.beanFactory = beanFactory;
	}


	protected void doProcessProperties(ConfigurableListableBeanFactory beanFactoryToProcess,
			StringValueResolver valueResolver) {

		BeanDefinitionVisitor visitor = new BeanDefinitionVisitor(valueResolver);

		String[] beanNames = beanFactoryToProcess.getBeanDefinitionNames();
		for (String curName : beanNames) {
			// Check that we're not parsing our own bean definition,
			// to avoid failing on unresolvable placeholders in properties file locations.
			if (!(curName.equals(this.beanName) && beanFactoryToProcess.equals(this.beanFactory))) {
				BeanDefinition bd = beanFactoryToProcess.getBeanDefinition(curName);
				try {
					visitor.visitBeanDefinition(bd);
				}
				catch (Exception ex) {
					throw new BeanDefinitionStoreException(bd.getResourceDescription(), curName, ex.getMessage(), ex);
				}
			}
		}

		// New in Spring 2.5: resolve placeholders in alias target names and aliases as well.
		beanFactoryToProcess.resolveAliases(valueResolver);

		// New in Spring 3.0: resolve placeholders in embedded values such as annotation attributes.
		beanFactoryToProcess.addEmbeddedValueResolver(valueResolver);
	}

}

```


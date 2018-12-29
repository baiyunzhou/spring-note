# BeanFactory
## AliasRegistry.java

```java
package org.springframework.core;

/**
 * 别名注册器。来自core包。
 */
public interface AliasRegistry {

	void registerAlias(String name, String alias);

	void removeAlias(String alias);

	boolean isAlias(String name);

	String[] getAliases(String name);

}
```

### BeanDefinitionRegistry.java

```java
package org.springframework.beans.factory.support;

import org.springframework.beans.factory.BeanDefinitionStoreException;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.core.AliasRegistry;

/**
 * Bean定义注册器
 */
public interface BeanDefinitionRegistry extends AliasRegistry {

	void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException;

	void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	boolean containsBeanDefinition(String beanName);

	String[] getBeanDefinitionNames();

	int getBeanDefinitionCount();

	boolean isBeanNameInUse(String beanName);

}
```

## SingletonBeanRegistry.java

```java
package org.springframework.beans.factory.config;

/**
 * 单例注册器
 */
public interface SingletonBeanRegistry {

	void registerSingleton(String beanName, Object singletonObject);

	Object getSingleton(String beanName);

	boolean containsSingleton(String beanName);

	String[] getSingletonNames();

	int getSingletonCount();

	Object getSingletonMutex();

}
```

## BeanFactory.java

```java
package org.springframework.beans.factory;

import org.springframework.beans.BeansException;
import org.springframework.core.ResolvableType;

/**
 * Bean工厂
 */
public interface BeanFactory {

	String FACTORY_BEAN_PREFIX = "&";

	Object getBean(String name) throws BeansException;

	<T> T getBean(String name, Class<T> requiredType) throws BeansException;

	Object getBean(String name, Object... args) throws BeansException;

	<T> T getBean(Class<T> requiredType) throws BeansException;

	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

	boolean containsBean(String name);

	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

	Class<?> getType(String name) throws NoSuchBeanDefinitionException;

	String[] getAliases(String name);

}
```

### ListableBeanFactory.java

```java
package org.springframework.beans.factory;

import java.lang.annotation.Annotation;
import java.util.Map;

import org.springframework.beans.BeansException;
import org.springframework.core.ResolvableType;

/**
 * 可列举的Bean工厂
 */
public interface ListableBeanFactory extends BeanFactory {

	boolean containsBeanDefinition(String beanName);

	int getBeanDefinitionCount();

	String[] getBeanDefinitionNames();

	String[] getBeanNamesForType(ResolvableType type);

	String[] getBeanNamesForType(Class<?> type);

	String[] getBeanNamesForType(Class<?> type, boolean includeNonSingletons, boolean allowEagerInit);

	<T> Map<String, T> getBeansOfType(Class<T> type) throws BeansException;

	<T> Map<String, T> getBeansOfType(Class<T> type, boolean includeNonSingletons, boolean allowEagerInit)
			throws BeansException;

	String[] getBeanNamesForAnnotation(Class<? extends Annotation> annotationType);

	Map<String, Object> getBeansWithAnnotation(Class<? extends Annotation> annotationType) throws BeansException;

	<A extends Annotation> A findAnnotationOnBean(String beanName, Class<A> annotationType)
			throws NoSuchBeanDefinitionException;

}

```

### AutowireCapableBeanFactory.java

```java
package org.springframework.beans.factory.config;

import java.util.Set;

import org.springframework.beans.BeansException;
import org.springframework.beans.TypeConverter;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;

/**
 * 可自动装配的Bean工厂
 */
public interface AutowireCapableBeanFactory extends BeanFactory {

	int AUTOWIRE_NO = 0;
	int AUTOWIRE_BY_NAME = 1;
	int AUTOWIRE_BY_TYPE = 2;
	int AUTOWIRE_CONSTRUCTOR = 3;
	@Deprecated
	int AUTOWIRE_AUTODETECT = 4;

	<T> T createBean(Class<T> beanClass) throws BeansException;

	void autowireBean(Object existingBean) throws BeansException;

	Object configureBean(Object existingBean, String beanName) throws BeansException;

	Object createBean(Class<?> beanClass, int autowireMode, boolean dependencyCheck) throws BeansException;

	Object autowire(Class<?> beanClass, int autowireMode, boolean dependencyCheck) throws BeansException;

	void autowireBeanProperties(Object existingBean, int autowireMode, boolean dependencyCheck)
			throws BeansException;

	void applyBeanPropertyValues(Object existingBean, String beanName) throws BeansException;

	Object initializeBean(Object existingBean, String beanName) throws BeansException;

	Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException;

	Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException;

	void destroyBean(Object existingBean);

	<T> NamedBeanHolder<T> resolveNamedBean(Class<T> requiredType) throws BeansException;

	Object resolveDependency(DependencyDescriptor descriptor, String requestingBeanName) throws BeansException;

	Object resolveDependency(DependencyDescriptor descriptor, String requestingBeanName,
			Set<String> autowiredBeanNames, TypeConverter typeConverter) throws BeansException;

}

```
### HierarchicalBeanFactory.java

```java
package org.springframework.beans.factory;

/**
 * 分层的Bean工厂
 */
public interface HierarchicalBeanFactory extends BeanFactory {

	BeanFactory getParentBeanFactory();

	boolean containsLocalBean(String name);

}
```

#### ConfigurableBeanFactory.java

```java
package org.springframework.beans.factory.config;

import java.beans.PropertyEditor;
import java.security.AccessControlContext;

import org.springframework.beans.PropertyEditorRegistrar;
import org.springframework.beans.PropertyEditorRegistry;
import org.springframework.beans.TypeConverter;
import org.springframework.beans.factory.BeanDefinitionStoreException;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.HierarchicalBeanFactory;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.core.convert.ConversionService;
import org.springframework.util.StringValueResolver;

/**
 * 可配置的Bean工厂
 */
public interface ConfigurableBeanFactory extends HierarchicalBeanFactory, SingletonBeanRegistry {

	String SCOPE_SINGLETON = "singleton";

	String SCOPE_PROTOTYPE = "prototype";

	void setParentBeanFactory(BeanFactory parentBeanFactory) throws IllegalStateException;

	void setBeanClassLoader(ClassLoader beanClassLoader);

	ClassLoader getBeanClassLoader();

	void setTempClassLoader(ClassLoader tempClassLoader);

	ClassLoader getTempClassLoader();

	void setCacheBeanMetadata(boolean cacheBeanMetadata);

	boolean isCacheBeanMetadata();

	void setBeanExpressionResolver(BeanExpressionResolver resolver);

	BeanExpressionResolver getBeanExpressionResolver();

	void setConversionService(ConversionService conversionService);

	ConversionService getConversionService();

	void addPropertyEditorRegistrar(PropertyEditorRegistrar registrar);

	void registerCustomEditor(Class<?> requiredType, Class<? extends PropertyEditor> propertyEditorClass);

	void copyRegisteredEditorsTo(PropertyEditorRegistry registry);

	void setTypeConverter(TypeConverter typeConverter);

	TypeConverter getTypeConverter();

	void addEmbeddedValueResolver(StringValueResolver valueResolver);

	boolean hasEmbeddedValueResolver();

	String resolveEmbeddedValue(String value);

	void addBeanPostProcessor(BeanPostProcessor beanPostProcessor);

	int getBeanPostProcessorCount();

	void registerScope(String scopeName, Scope scope);

	String[] getRegisteredScopeNames();

	Scope getRegisteredScope(String scopeName);

	AccessControlContext getAccessControlContext();

	void copyConfigurationFrom(ConfigurableBeanFactory otherFactory);

	void registerAlias(String beanName, String alias) throws BeanDefinitionStoreException;

	void resolveAliases(StringValueResolver valueResolver);

	BeanDefinition getMergedBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	boolean isFactoryBean(String name) throws NoSuchBeanDefinitionException;

	void setCurrentlyInCreation(String beanName, boolean inCreation);

	boolean isCurrentlyInCreation(String beanName);

	void registerDependentBean(String beanName, String dependentBeanName);

	String[] getDependentBeans(String beanName);

	String[] getDependenciesForBean(String beanName);

	void destroyBean(String beanName, Object beanInstance);

	void destroyScopedBean(String beanName);

	void destroySingletons();

}
```

##### ConfigurableListableBeanFactory.java

```java
package org.springframework.beans.factory.config;

import java.util.Iterator;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.ListableBeanFactory;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;

/**
 * 可配置、可列举的Bean工厂。同时也是可自动装配的，并添加了一些其他功能。
 */
public interface ConfigurableListableBeanFactory
		extends ListableBeanFactory, AutowireCapableBeanFactory, ConfigurableBeanFactory {

	void ignoreDependencyType(Class<?> type);

	void ignoreDependencyInterface(Class<?> ifc);

	void registerResolvableDependency(Class<?> dependencyType, Object autowiredValue);

	boolean isAutowireCandidate(String beanName, DependencyDescriptor descriptor)
			throws NoSuchBeanDefinitionException;

	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	Iterator<String> getBeanNamesIterator();

	void clearMetadataCache();

	void freezeConfiguration();

	boolean isConfigurationFrozen();

	void preInstantiateSingletons() throws BeansException;

}

```

# BeanDefinition

## BeanDefinition.java

```java
package org.springframework.beans.factory.config;

import org.springframework.beans.BeanMetadataElement;
import org.springframework.beans.MutablePropertyValues;
import org.springframework.core.AttributeAccessor;

/**
 * BeanDefinition
 */
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

	String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;

	String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;

	int ROLE_APPLICATION = 0;

	int ROLE_SUPPORT = 1;

	int ROLE_INFRASTRUCTURE = 2;

	void setParentName(String parentName);

	String getParentName();

	void setBeanClassName(String beanClassName);

	String getBeanClassName();

	void setScope(String scope);

	String getScope();

	void setLazyInit(boolean lazyInit);

	boolean isLazyInit();

	void setDependsOn(String... dependsOn);

	String[] getDependsOn();

	void setAutowireCandidate(boolean autowireCandidate);

	boolean isAutowireCandidate();

	void setPrimary(boolean primary);

	boolean isPrimary();

	void setFactoryBeanName(String factoryBeanName);

	String getFactoryBeanName();

	void setFactoryMethodName(String factoryMethodName);

	String getFactoryMethodName();

	ConstructorArgumentValues getConstructorArgumentValues();

	MutablePropertyValues getPropertyValues();

	boolean isSingleton();

	boolean isPrototype();

	boolean isAbstract();

	int getRole();

	String getDescription();

	String getResourceDescription();

	BeanDefinition getOriginatingBeanDefinition();

}

```

### AnnotatedBeanDefinition.java

```java
package org.springframework.beans.factory.annotation;

import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.MethodMetadata;

/**
 * 注解BeanDefinition
 */
public interface AnnotatedBeanDefinition extends BeanDefinition {

	AnnotationMetadata getMetadata();

	MethodMetadata getFactoryMethodMetadata();

}

```

### AbstractBeanDefinition

#### ChildBeanDefinition

#### GenericBeanDefinition

##### AnnotatedGenericBeanDefinition

##### ScannedGenericBeanDefinition

#### RootBeanDefinition

# Document

## EntityResolver

rt包下org.xml.sax.EntityResolver

### BeansDtdResolver.java

```java
package org.springframework.beans.factory.xml;

import java.io.IOException;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.xml.sax.EntityResolver;
import org.xml.sax.InputSource;

import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;

/**
 * DTD约束文件解析器。读取当前包下spring-beans-2.0.dtd
 */
public class BeansDtdResolver implements EntityResolver {

	private static final String DTD_EXTENSION = ".dtd";

	private static final String DTD_FILENAME = "spring-beans-2.0";

	private static final String DTD_NAME = "spring-beans";

	private static final Log logger = LogFactory.getLog(BeansDtdResolver.class);


	@Override
	public InputSource resolveEntity(String publicId, String systemId) throws IOException {
		if (logger.isTraceEnabled()) {
			logger.trace("Trying to resolve XML entity with public ID [" + publicId +
					"] and system ID [" + systemId + "]");
		}
		if (systemId != null && systemId.endsWith(DTD_EXTENSION)) {
			int lastPathSeparator = systemId.lastIndexOf('/');
			int dtdNameStart = systemId.indexOf(DTD_NAME, lastPathSeparator);
			if (dtdNameStart != -1) {
				String dtdFile = DTD_FILENAME + DTD_EXTENSION;
				if (logger.isTraceEnabled()) {
					logger.trace("Trying to locate [" + dtdFile + "] in Spring jar on classpath");
				}
				try {
					Resource resource = new ClassPathResource(dtdFile, getClass());
					InputSource source = new InputSource(resource.getInputStream());
					source.setPublicId(publicId);
					source.setSystemId(systemId);
					if (logger.isDebugEnabled()) {
						logger.debug("Found beans DTD [" + systemId + "] in classpath: " + dtdFile);
					}
					return source;
				}
				catch (IOException ex) {
					if (logger.isDebugEnabled()) {
						logger.debug("Could not resolve beans DTD [" + systemId + "]: not found in classpath", ex);
					}
				}

			}
		}
		return null;
	}


	@Override
	public String toString() {
		return "EntityResolver for spring-beans DTD";
	}

}
```

### PluggableSchemaResolver.java

```java

package org.springframework.beans.factory.xml;

import java.io.FileNotFoundException;
import java.io.IOException;
import java.util.Map;
import java.util.Properties;
import java.util.concurrent.ConcurrentHashMap;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.xml.sax.EntityResolver;
import org.xml.sax.InputSource;

import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.PropertiesLoaderUtils;
import org.springframework.util.Assert;
import org.springframework.util.CollectionUtils;

/**
 * 可插拔的实体解析器。通过解析META-INF/spring.schemas文件获取实体。内容格式为：systemId=schema-location。
 * schema-location应该在类路径下，而且应该转换:为\:
 * 如：http\://www.springframework.org/schema/beans/spring-beans-4.3.xsd=org/springframework/beans/factory/xml/spring-beans-4.3.xsd
 */
public class PluggableSchemaResolver implements EntityResolver {

	public static final String DEFAULT_SCHEMA_MAPPINGS_LOCATION = "META-INF/spring.schemas";


	private static final Log logger = LogFactory.getLog(PluggableSchemaResolver.class);

	private final ClassLoader classLoader;

	private final String schemaMappingsLocation;

	private volatile Map<String, String> schemaMappings;

	public PluggableSchemaResolver(ClassLoader classLoader) {
		this.classLoader = classLoader;
		this.schemaMappingsLocation = DEFAULT_SCHEMA_MAPPINGS_LOCATION;
	}

	public PluggableSchemaResolver(ClassLoader classLoader, String schemaMappingsLocation) {
		Assert.hasText(schemaMappingsLocation, "'schemaMappingsLocation' must not be empty");
		this.classLoader = classLoader;
		this.schemaMappingsLocation = schemaMappingsLocation;
	}

	@Override
	public InputSource resolveEntity(String publicId, String systemId) throws IOException {
		if (logger.isTraceEnabled()) {
			logger.trace("Trying to resolve XML entity with public id [" + publicId +
					"] and system id [" + systemId + "]");
		}

		if (systemId != null) {
			String resourceLocation = getSchemaMappings().get(systemId);
			if (resourceLocation != null) {
				Resource resource = new ClassPathResource(resourceLocation, this.classLoader);
				try {
					InputSource source = new InputSource(resource.getInputStream());
					source.setPublicId(publicId);
					source.setSystemId(systemId);
					if (logger.isDebugEnabled()) {
						logger.debug("Found XML schema [" + systemId + "] in classpath: " + resourceLocation);
					}
					return source;
				}
				catch (FileNotFoundException ex) {
					if (logger.isDebugEnabled()) {
						logger.debug("Could not find XML schema [" + systemId + "]: " + resource, ex);
					}
				}
			}
		}
		return null;
	}

	private Map<String, String> getSchemaMappings() {
		Map<String, String> schemaMappings = this.schemaMappings;
		if (schemaMappings == null) {
			synchronized (this) {
				schemaMappings = this.schemaMappings;
				if (schemaMappings == null) {
					if (logger.isDebugEnabled()) {
						logger.debug("Loading schema mappings from [" + this.schemaMappingsLocation + "]");
					}
					try {
						Properties mappings =
								PropertiesLoaderUtils.loadAllProperties(this.schemaMappingsLocation, this.classLoader);
						if (logger.isDebugEnabled()) {
							logger.debug("Loaded schema mappings: " + mappings);
						}
						schemaMappings = new ConcurrentHashMap<String, String>(mappings.size());
						CollectionUtils.mergePropertiesIntoMap(mappings, schemaMappings);
						this.schemaMappings = schemaMappings;
					}
					catch (IOException ex) {
						throw new IllegalStateException(
								"Unable to load schema mappings from location [" + this.schemaMappingsLocation + "]", ex);
					}
				}
			}
		}
		return schemaMappings;
	}


	@Override
	public String toString() {
		return "EntityResolver using mappings " + getSchemaMappings();
	}

}

```

### DelegatingEntityResolver.java

```java
package org.springframework.beans.factory.xml;

import java.io.IOException;

import org.xml.sax.EntityResolver;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;

import org.springframework.util.Assert;

/**
 * 代理实体解析器。dtd就给BeansDtdResolver处理，xsd就给PluggableSchemaResolver处理。
 */
public class DelegatingEntityResolver implements EntityResolver {

	public static final String DTD_SUFFIX = ".dtd";

	public static final String XSD_SUFFIX = ".xsd";


	private final EntityResolver dtdResolver;

	private final EntityResolver schemaResolver;

	public DelegatingEntityResolver(ClassLoader classLoader) {
		this.dtdResolver = new BeansDtdResolver();
		this.schemaResolver = new PluggableSchemaResolver(classLoader);
	}

	public DelegatingEntityResolver(EntityResolver dtdResolver, EntityResolver schemaResolver) {
		Assert.notNull(dtdResolver, "'dtdResolver' is required");
		Assert.notNull(schemaResolver, "'schemaResolver' is required");
		this.dtdResolver = dtdResolver;
		this.schemaResolver = schemaResolver;
	}


	@Override
	public InputSource resolveEntity(String publicId, String systemId) throws SAXException, IOException {
		if (systemId != null) {
			if (systemId.endsWith(DTD_SUFFIX)) {
				return this.dtdResolver.resolveEntity(publicId, systemId);
			}
			else if (systemId.endsWith(XSD_SUFFIX)) {
				return this.schemaResolver.resolveEntity(publicId, systemId);
			}
		}
		return null;
	}


	@Override
	public String toString() {
		return "EntityResolver delegating " + XSD_SUFFIX + " to " + this.schemaResolver +
				" and " + DTD_SUFFIX + " to " + this.dtdResolver;
	}

}

```

#### ResourceEntityResolver.java

```java
package org.springframework.beans.factory.xml;

import java.io.File;
import java.io.IOException;
import java.net.URL;
import java.net.URLDecoder;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;

import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;

/**
 * 资源实体解析器。如果DelegatingEntityResolver没解析到，尝试从资源路径解析。绝对路径或者相对路径。
 */
public class ResourceEntityResolver extends DelegatingEntityResolver {

	private static final Log logger = LogFactory.getLog(ResourceEntityResolver.class);

	private final ResourceLoader resourceLoader;

	public ResourceEntityResolver(ResourceLoader resourceLoader) {
		super(resourceLoader.getClassLoader());
		this.resourceLoader = resourceLoader;
	}


	@Override
	public InputSource resolveEntity(String publicId, String systemId) throws SAXException, IOException {
		InputSource source = super.resolveEntity(publicId, systemId);
		if (source == null && systemId != null) {
			String resourcePath = null;
			try {
				String decodedSystemId = URLDecoder.decode(systemId, "UTF-8");
				String givenUrl = new URL(decodedSystemId).toString();
				String systemRootUrl = new File("").toURI().toURL().toString();
				// Try relative to resource base if currently in system root.
				if (givenUrl.startsWith(systemRootUrl)) {
					resourcePath = givenUrl.substring(systemRootUrl.length());
				}
			}
			catch (Exception ex) {
				// Typically a MalformedURLException or AccessControlException.
				if (logger.isDebugEnabled()) {
					logger.debug("Could not resolve XML entity [" + systemId + "] against system root URL", ex);
				}
				// No URL (or no resolvable URL) -> try relative to resource base.
				resourcePath = systemId;
			}
			if (resourcePath != null) {
				if (logger.isTraceEnabled()) {
					logger.trace("Trying to locate XML entity [" + systemId + "] as resource [" + resourcePath + "]");
				}
				Resource resource = this.resourceLoader.getResource(resourcePath);
				source = new InputSource(resource.getInputStream());
				source.setPublicId(publicId);
				source.setSystemId(systemId);
				if (logger.isDebugEnabled()) {
					logger.debug("Found XML entity [" + systemId + "]: " + resource);
				}
			}
		}
		return source;
	}

}

```

## ErrorHandler.java

rt包下org.xml.sax.ErrorHandler

### SimpleSaxErrorHandler.java

```java
package org.springframework.util.xml;

import org.apache.commons.logging.Log;
import org.xml.sax.ErrorHandler;
import org.xml.sax.SAXException;
import org.xml.sax.SAXParseException;

/**
 * 简单错误处理器。不致命的就打下警告日志就行了。
 */
public class SimpleSaxErrorHandler implements ErrorHandler {

	private final Log logger;

	public SimpleSaxErrorHandler(Log logger) {
		this.logger = logger;
	}


	@Override
	public void warning(SAXParseException ex) throws SAXException {
		logger.warn("Ignored XML validation warning", ex);
	}

	@Override
	public void error(SAXParseException ex) throws SAXException {
		throw ex;
	}

	@Override
	public void fatalError(SAXParseException ex) throws SAXException {
		throw ex;
	}

}

```

## XmlValidationModeDetector.java

```java
package org.springframework.util.xml;

import java.io.BufferedReader;
import java.io.CharConversionException;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;

import org.springframework.util.StringUtils;

/**
 * 获取XML文件的验证模式
 */
public class XmlValidationModeDetector {

	public static final int VALIDATION_NONE = 0;

	public static final int VALIDATION_AUTO = 1;

	public static final int VALIDATION_DTD = 2;

	public static final int VALIDATION_XSD = 3;

	private static final String DOCTYPE = "DOCTYPE";

	private static final String START_COMMENT = "<!--";

	private static final String END_COMMENT = "-->";

	private boolean inComment;

	public int detectValidationMode(InputStream inputStream) throws IOException {
		// Peek into the file to look for DOCTYPE.
		BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
		try {
			boolean isDtdValidated = false;
			String content;
			while ((content = reader.readLine()) != null) {
				content = consumeCommentTokens(content);
				if (this.inComment || !StringUtils.hasText(content)) {
					continue;
				}
				if (hasDoctype(content)) {
					isDtdValidated = true;
					break;
				}
				if (hasOpeningTag(content)) {
					// End of meaningful data...
					break;
				}
			}
			return (isDtdValidated ? VALIDATION_DTD : VALIDATION_XSD);
		}
		catch (CharConversionException ex) {
			// Choked on some character encoding...
			// Leave the decision up to the caller.
			return VALIDATION_AUTO;
		}
		finally {
			reader.close();
		}
	}

	private boolean hasDoctype(String content) {
		return content.contains(DOCTYPE);
	}

	private boolean hasOpeningTag(String content) {
		if (this.inComment) {
			return false;
		}
		int openTagIndex = content.indexOf('<');
		return (openTagIndex > -1 && (content.length() > openTagIndex + 1) &&
				Character.isLetter(content.charAt(openTagIndex + 1)));
	}

	private String consumeCommentTokens(String line) {
		if (!line.contains(START_COMMENT) && !line.contains(END_COMMENT)) {
			return line;
		}
		while ((line = consume(line)) != null) {
			if (!this.inComment && !line.trim().startsWith(START_COMMENT)) {
				return line;
			}
		}
		return line;
	}

	private String consume(String line) {
		int index = (this.inComment ? endComment(line) : startComment(line));
		return (index == -1 ? null : line.substring(index));
	}

	private int startComment(String line) {
		return commentToken(line, START_COMMENT, true);
	}

	private int endComment(String line) {
		return commentToken(line, END_COMMENT, false);
	}

	private int commentToken(String line, String token, boolean inCommentIfPresent) {
		int index = line.indexOf(token);
		if (index > - 1) {
			this.inComment = inCommentIfPresent;
		}
		return (index == -1 ? index : index + token.length());
	}

}

```

## DocumentLoader.java

```java
package org.springframework.beans.factory.xml;

import org.w3c.dom.Document;
import org.xml.sax.EntityResolver;
import org.xml.sax.ErrorHandler;
import org.xml.sax.InputSource;

/**
 * Document加载器
 */
public interface DocumentLoader {
  
	Document loadDocument(
			InputSource inputSource, EntityResolver entityResolver,
			ErrorHandler errorHandler, int validationMode, boolean namespaceAware)
			throws Exception;

}

```

### DefaultDocumentLoader.java

```java
package org.springframework.beans.factory.xml;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.w3c.dom.Document;
import org.xml.sax.EntityResolver;
import org.xml.sax.ErrorHandler;
import org.xml.sax.InputSource;

import org.springframework.util.xml.XmlValidationModeDetector;

/**
 * Document加载器的默认实现
 */
public class DefaultDocumentLoader implements DocumentLoader {

	private static final String SCHEMA_LANGUAGE_ATTRIBUTE = "http://java.sun.com/xml/jaxp/properties/schemaLanguage";

	private static final String XSD_SCHEMA_LANGUAGE = "http://www.w3.org/2001/XMLSchema";


	private static final Log logger = LogFactory.getLog(DefaultDocumentLoader.class);

	@Override
	public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
			ErrorHandler errorHandler, int validationMode, boolean namespaceAware) throws Exception {

		DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);
		if (logger.isDebugEnabled()) {
			logger.debug("Using JAXP provider [" + factory.getClass().getName() + "]");
		}
		DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);
		return builder.parse(inputSource);
	}

	protected DocumentBuilderFactory createDocumentBuilderFactory(int validationMode, boolean namespaceAware)
			throws ParserConfigurationException {

		DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
		factory.setNamespaceAware(namespaceAware);

		if (validationMode != XmlValidationModeDetector.VALIDATION_NONE) {
			factory.setValidating(true);
			if (validationMode == XmlValidationModeDetector.VALIDATION_XSD) {
				// Enforce namespace aware for XSD...
				factory.setNamespaceAware(true);
				try {
					factory.setAttribute(SCHEMA_LANGUAGE_ATTRIBUTE, XSD_SCHEMA_LANGUAGE);
				}
				catch (IllegalArgumentException ex) {
					ParserConfigurationException pcex = new ParserConfigurationException(
							"Unable to validate using XSD: Your JAXP provider [" + factory +
							"] does not support XML Schema. Are you running on Java 1.4 with Apache Crimson? " +
							"Upgrade to Apache Xerces (or Java 1.5) for full XSD support.");
					pcex.initCause(ex);
					throw pcex;
				}
			}
		}

		return factory;
	}

	protected DocumentBuilder createDocumentBuilder(
			DocumentBuilderFactory factory, EntityResolver entityResolver, ErrorHandler errorHandler)
			throws ParserConfigurationException {

		DocumentBuilder docBuilder = factory.newDocumentBuilder();
		if (entityResolver != null) {
			docBuilder.setEntityResolver(entityResolver);
		}
		if (errorHandler != null) {
			docBuilder.setErrorHandler(errorHandler);
		}
		return docBuilder;
	}

}
```

# BeanDefinitionDocumentReader

## ProblemReporter.java

```java
package org.springframework.beans.factory.parsing;

public interface ProblemReporter {

	void fatal(Problem problem);

	void error(Problem problem);

	void warning(Problem problem);

}
```

### FailFastProblemReporter.java

```java
package org.springframework.beans.factory.parsing;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

public class FailFastProblemReporter implements ProblemReporter {

	private Log logger = LogFactory.getLog(getClass());

	public void setLogger(Log logger) {
		this.logger = (logger != null ? logger : LogFactory.getLog(getClass()));
	}

	@Override
	public void fatal(Problem problem) {
		throw new BeanDefinitionParsingException(problem);
	}

	@Override
	public void error(Problem problem) {
		throw new BeanDefinitionParsingException(problem);
	}

	@Override
	public void warning(Problem problem) {
		this.logger.warn(problem, problem.getRootCause());
	}

}

```

## EventListener.java

java.util.EventListener

### ReaderEventListener.java

```java
package org.springframework.beans.factory.parsing;

import java.util.EventListener;

public interface ReaderEventListener extends EventListener {

	void defaultsRegistered(DefaultsDefinition defaultsDefinition);

	void componentRegistered(ComponentDefinition componentDefinition);

	void aliasRegistered(AliasDefinition aliasDefinition);

	void importProcessed(ImportDefinition importDefinition);

}

```

#### EmptyReaderEventListener.java

```java
package org.springframework.beans.factory.parsing;

public class EmptyReaderEventListener implements ReaderEventListener {

	@Override
	public void defaultsRegistered(DefaultsDefinition defaultsDefinition) {
		// no-op
	}

	@Override
	public void componentRegistered(ComponentDefinition componentDefinition) {
		// no-op
	}

	@Override
	public void aliasRegistered(AliasDefinition aliasDefinition) {
		// no-op
	}

	@Override
	public void importProcessed(ImportDefinition importDefinition) {
		// no-op
	}

}
```



## SourceExtractor.java

```java
package org.springframework.beans.factory.parsing;

import org.springframework.core.io.Resource;

public interface SourceExtractor {

	Object extractSource(Object sourceCandidate, Resource definingResource);

}

```

### NullSourceExtractor.java

```java
package org.springframework.beans.factory.parsing;

import org.springframework.core.io.Resource;

public class NullSourceExtractor implements SourceExtractor {

	@Override
	public Object extractSource(Object sourceCandidate, Resource definitionResource) {
		return null;
	}

}

```

### PassThroughSourceExtractor.java

```java
package org.springframework.beans.factory.parsing;

import org.springframework.core.io.Resource;

public class PassThroughSourceExtractor implements SourceExtractor {

	@Override
	public Object extractSource(Object sourceCandidate, Resource definingResource) {
		return sourceCandidate;
	}

}

```


## BeanDefinitionParser.java

```java
package org.springframework.beans.factory.xml;

import org.w3c.dom.Element;

import org.springframework.beans.factory.config.BeanDefinition;

public interface BeanDefinitionParser {

	BeanDefinition parse(Element element, ParserContext parserContext);

}

```

## BeanDefinitionDecorator.java

```java
package org.springframework.beans.factory.xml;

import org.w3c.dom.Node;

import org.springframework.beans.factory.config.BeanDefinitionHolder;

public interface BeanDefinitionDecorator {

	BeanDefinitionHolder decorate(Node node, BeanDefinitionHolder definition, ParserContext parserContext);

}

```
## NamespaceHandler.java

```java
package org.springframework.beans.factory.xml;

import org.w3c.dom.Element;
import org.w3c.dom.Node;

import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanDefinitionHolder;
/**
  * 名称空间处理器
  */
public interface NamespaceHandler {

	void init();

	BeanDefinition parse(Element element, ParserContext parserContext);

	BeanDefinitionHolder decorate(Node source, BeanDefinitionHolder definition, ParserContext parserContext);

}
```
### NamespaceHandlerSupport.java

```java
package org.springframework.beans.factory.xml;

import java.util.HashMap;
import java.util.Map;

import org.w3c.dom.Attr;
import org.w3c.dom.Element;
import org.w3c.dom.Node;

import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanDefinitionHolder;

/**
 * 名称处理器默认实现。
 */
public abstract class NamespaceHandlerSupport implements NamespaceHandler {

	private final Map<String, BeanDefinitionParser> parsers =
			new HashMap<String, BeanDefinitionParser>();

	private final Map<String, BeanDefinitionDecorator> decorators =
			new HashMap<String, BeanDefinitionDecorator>();

	private final Map<String, BeanDefinitionDecorator> attributeDecorators =
			new HashMap<String, BeanDefinitionDecorator>();

	@Override
	public BeanDefinition parse(Element element, ParserContext parserContext) {
		return findParserForElement(element, parserContext).parse(element, parserContext);
	}

	private BeanDefinitionParser findParserForElement(Element element, ParserContext parserContext) {
		String localName = parserContext.getDelegate().getLocalName(element);
		BeanDefinitionParser parser = this.parsers.get(localName);
		if (parser == null) {
			parserContext.getReaderContext().fatal(
					"Cannot locate BeanDefinitionParser for element [" + localName + "]", element);
		}
		return parser;
	}

	@Override
	public BeanDefinitionHolder decorate(
			Node node, BeanDefinitionHolder definition, ParserContext parserContext) {

		return findDecoratorForNode(node, parserContext).decorate(node, definition, parserContext);
	}

	private BeanDefinitionDecorator findDecoratorForNode(Node node, ParserContext parserContext) {
		BeanDefinitionDecorator decorator = null;
		String localName = parserContext.getDelegate().getLocalName(node);
		if (node instanceof Element) {
			decorator = this.decorators.get(localName);
		}
		else if (node instanceof Attr) {
			decorator = this.attributeDecorators.get(localName);
		}
		else {
			parserContext.getReaderContext().fatal(
					"Cannot decorate based on Nodes of type [" + node.getClass().getName() + "]", node);
		}
		if (decorator == null) {
			parserContext.getReaderContext().fatal("Cannot locate BeanDefinitionDecorator for " +
					(node instanceof Element ? "element" : "attribute") + " [" + localName + "]", node);
		}
		return decorator;
	}

	protected final void registerBeanDefinitionParser(String elementName, BeanDefinitionParser parser) {
		this.parsers.put(elementName, parser);
	}

	protected final void registerBeanDefinitionDecorator(String elementName, BeanDefinitionDecorator dec) {
		this.decorators.put(elementName, dec);
	}

	protected final void registerBeanDefinitionDecoratorForAttribute(String attrName, BeanDefinitionDecorator dec) {
		this.attributeDecorators.put(attrName, dec);
	}

}

```

### SimpleConstructorNamespaceHandler.java

```java
package org.springframework.beans.factory.xml;

import java.util.Collection;

import org.w3c.dom.Attr;
import org.w3c.dom.Element;
import org.w3c.dom.Node;

import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanDefinitionHolder;
import org.springframework.beans.factory.config.ConstructorArgumentValues;
import org.springframework.beans.factory.config.ConstructorArgumentValues.ValueHolder;
import org.springframework.beans.factory.config.RuntimeBeanReference;
import org.springframework.core.Conventions;
import org.springframework.util.StringUtils;

/**
 * C名称空间解析器
 */
public class SimpleConstructorNamespaceHandler implements NamespaceHandler {

	private static final String REF_SUFFIX = "-ref";

	private static final String DELIMITER_PREFIX = "_";


	@Override
	public void init() {
	}

	@Override
	public BeanDefinition parse(Element element, ParserContext parserContext) {
		parserContext.getReaderContext().error(
				"Class [" + getClass().getName() + "] does not support custom elements.", element);
		return null;
	}

	@Override
	public BeanDefinitionHolder decorate(Node node, BeanDefinitionHolder definition, ParserContext parserContext) {
		if (node instanceof Attr) {
			Attr attr = (Attr) node;
			String argName = StringUtils.trimWhitespace(parserContext.getDelegate().getLocalName(attr));
			String argValue = StringUtils.trimWhitespace(attr.getValue());

			ConstructorArgumentValues cvs = definition.getBeanDefinition().getConstructorArgumentValues();
			boolean ref = false;

			// handle -ref arguments
			if (argName.endsWith(REF_SUFFIX)) {
				ref = true;
				argName = argName.substring(0, argName.length() - REF_SUFFIX.length());
			}

			ValueHolder valueHolder = new ValueHolder(ref ? new RuntimeBeanReference(argValue) : argValue);
			valueHolder.setSource(parserContext.getReaderContext().extractSource(attr));

			// handle "escaped"/"_" arguments
			if (argName.startsWith(DELIMITER_PREFIX)) {
				String arg = argName.substring(1).trim();

				// fast default check
				if (!StringUtils.hasText(arg)) {
					cvs.addGenericArgumentValue(valueHolder);
				}
				// assume an index otherwise
				else {
					int index = -1;
					try {
						index = Integer.parseInt(arg);
					}
					catch (NumberFormatException ex) {
						parserContext.getReaderContext().error(
								"Constructor argument '" + argName + "' specifies an invalid integer", attr);
					}
					if (index < 0) {
						parserContext.getReaderContext().error(
								"Constructor argument '" + argName + "' specifies a negative index", attr);
					}

					if (cvs.hasIndexedArgumentValue(index)){
						parserContext.getReaderContext().error(
								"Constructor argument '" + argName + "' with index "+ index+" already defined using <constructor-arg>." +
								" Only one approach may be used per argument.", attr);
					}

					cvs.addIndexedArgumentValue(index, valueHolder);
				}
			}
			// no escaping -> ctr name
			else {
				String name = Conventions.attributeNameToPropertyName(argName);
				if (containsArgWithName(name, cvs)){
					parserContext.getReaderContext().error(
							"Constructor argument '" + argName + "' already defined using <constructor-arg>." +
							" Only one approach may be used per argument.", attr);
				}
				valueHolder.setName(Conventions.attributeNameToPropertyName(argName));
				cvs.addGenericArgumentValue(valueHolder);
			}
		}
		return definition;
	}

	private boolean containsArgWithName(String name, ConstructorArgumentValues cvs) {
		return (checkName(name, cvs.getGenericArgumentValues()) ||
				checkName(name, cvs.getIndexedArgumentValues().values()));
	}

	private boolean checkName(String name, Collection<ValueHolder> values) {
		for (ValueHolder holder : values) {
			if (name.equals(holder.getName())) {
				return true;
			}
		}
		return false;
	}

}

```

### SimplePropertyNamespaceHandler.java

```java
package org.springframework.beans.factory.xml;

import org.w3c.dom.Attr;
import org.w3c.dom.Element;
import org.w3c.dom.Node;

import org.springframework.beans.MutablePropertyValues;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanDefinitionHolder;
import org.springframework.beans.factory.config.RuntimeBeanReference;
import org.springframework.core.Conventions;

/**
 * P名称空间解析器
 */
public class SimplePropertyNamespaceHandler implements NamespaceHandler {

	private static final String REF_SUFFIX = "-ref";


	@Override
	public void init() {
	}

	@Override
	public BeanDefinition parse(Element element, ParserContext parserContext) {
		parserContext.getReaderContext().error(
				"Class [" + getClass().getName() + "] does not support custom elements.", element);
		return null;
	}

	@Override
	public BeanDefinitionHolder decorate(Node node, BeanDefinitionHolder definition, ParserContext parserContext) {
		if (node instanceof Attr) {
			Attr attr = (Attr) node;
			String propertyName = parserContext.getDelegate().getLocalName(attr);
			String propertyValue = attr.getValue();
			MutablePropertyValues pvs = definition.getBeanDefinition().getPropertyValues();
			if (pvs.contains(propertyName)) {
				parserContext.getReaderContext().error("Property '" + propertyName + "' is already defined using " +
						"both <property> and inline syntax. Only one approach may be used per property.", attr);
			}
			if (propertyName.endsWith(REF_SUFFIX)) {
				propertyName = propertyName.substring(0, propertyName.length() - REF_SUFFIX.length());
				pvs.add(Conventions.attributeNameToPropertyName(propertyName), new RuntimeBeanReference(propertyValue));
			}
			else {
				pvs.add(Conventions.attributeNameToPropertyName(propertyName), propertyValue);
			}
		}
		return definition;
	}

}

```

## NamespaceHandlerResolver.java

```java
package org.springframework.beans.factory.xml;

/**
 * 名称空间处理器解析器
 */
public interface NamespaceHandlerResolver {

	NamespaceHandler resolve(String namespaceUri);

}

```

### DefaultNamespaceHandlerResolver.java

```java
package org.springframework.beans.factory.xml;

import java.io.IOException;
import java.util.Map;
import java.util.Properties;
import java.util.concurrent.ConcurrentHashMap;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import org.springframework.beans.BeanUtils;
import org.springframework.beans.FatalBeanException;
import org.springframework.core.io.support.PropertiesLoaderUtils;
import org.springframework.util.Assert;
import org.springframework.util.ClassUtils;
import org.springframework.util.CollectionUtils;

/**
 * 默认的名称空间处理器解析器。读取META-INF/spring.handlers文件获取映射信息。注意：转义
 * 如：http\://www.springframework.org/schema/c=org.springframework.beans.factory.xml.SimpleConstructorNamespaceHandler
 */
public class DefaultNamespaceHandlerResolver implements NamespaceHandlerResolver {

	public static final String DEFAULT_HANDLER_MAPPINGS_LOCATION = "META-INF/spring.handlers";

	protected final Log logger = LogFactory.getLog(getClass());

	private final ClassLoader classLoader;

	private final String handlerMappingsLocation;

	private volatile Map<String, Object> handlerMappings;

	public DefaultNamespaceHandlerResolver() {
		this(null, DEFAULT_HANDLER_MAPPINGS_LOCATION);

	public DefaultNamespaceHandlerResolver(ClassLoader classLoader) {
		this(classLoader, DEFAULT_HANDLER_MAPPINGS_LOCATION);
	}

	public DefaultNamespaceHandlerResolver(ClassLoader classLoader, String handlerMappingsLocation) {
		Assert.notNull(handlerMappingsLocation, "Handler mappings location must not be null");
		this.classLoader = (classLoader != null ? classLoader : ClassUtils.getDefaultClassLoader());
		this.handlerMappingsLocation = handlerMappingsLocation;
	}

	@Override
	public NamespaceHandler resolve(String namespaceUri) {
		Map<String, Object> handlerMappings = getHandlerMappings();
		Object handlerOrClassName = handlerMappings.get(namespaceUri);
		if (handlerOrClassName == null) {
			return null;
		}
		else if (handlerOrClassName instanceof NamespaceHandler) {
			return (NamespaceHandler) handlerOrClassName;
		}
		else {
			String className = (String) handlerOrClassName;
			try {
				Class<?> handlerClass = ClassUtils.forName(className, this.classLoader);
				if (!NamespaceHandler.class.isAssignableFrom(handlerClass)) {
					throw new FatalBeanException("Class [" + className + "] for namespace [" + namespaceUri +
							"] does not implement the [" + NamespaceHandler.class.getName() + "] interface");
				}
              	//使用无参构造函数初始化，自动调用init初始化
				NamespaceHandler namespaceHandler = (NamespaceHandler) BeanUtils.instantiateClass(handlerClass);
				namespaceHandler.init();
				handlerMappings.put(namespaceUri, namespaceHandler);
				return namespaceHandler;
			}
			catch (ClassNotFoundException ex) {
				throw new FatalBeanException("NamespaceHandler class [" + className + "] for namespace [" +
						namespaceUri + "] not found", ex);
			}
			catch (LinkageError err) {
				throw new FatalBeanException("Invalid NamespaceHandler class [" + className + "] for namespace [" +
						namespaceUri + "]: problem with handler class file or dependent class", err);
			}
		}
	}

	private Map<String, Object> getHandlerMappings() {
		Map<String, Object> handlerMappings = this.handlerMappings;
		if (handlerMappings == null) {
			synchronized (this) {
				handlerMappings = this.handlerMappings;
				if (handlerMappings == null) {
					if (logger.isDebugEnabled()) {
						logger.debug("Loading NamespaceHandler mappings from [" + this.handlerMappingsLocation + "]");
					}
					try {
						Properties mappings =
								PropertiesLoaderUtils.loadAllProperties(this.handlerMappingsLocation, this.classLoader);
						if (logger.isDebugEnabled()) {
							logger.debug("Loaded NamespaceHandler mappings: " + mappings);
						}
						handlerMappings = new ConcurrentHashMap<String, Object>(mappings.size());
						CollectionUtils.mergePropertiesIntoMap(mappings, handlerMappings);
						this.handlerMappings = handlerMappings;
					}
					catch (IOException ex) {
						throw new IllegalStateException(
								"Unable to load NamespaceHandler mappings from location [" + this.handlerMappingsLocation + "]", ex);
					}
				}
			}
		}
		return handlerMappings;
	}


	@Override
	public String toString() {
		return "NamespaceHandlerResolver using mappings " + getHandlerMappings();
	}

}

```



## ReaderContext.java

```java
package org.springframework.beans.factory.parsing;

import org.springframework.core.io.Resource;

/**
 * 读取器上下文
 */
public class ReaderContext {

	private final Resource resource;

	private final ProblemReporter problemReporter;

	private final ReaderEventListener eventListener;

	private final SourceExtractor sourceExtractor;

	public ReaderContext(Resource resource, ProblemReporter problemReporter,
			ReaderEventListener eventListener, SourceExtractor sourceExtractor) {

		this.resource = resource;
		this.problemReporter = problemReporter;
		this.eventListener = eventListener;
		this.sourceExtractor = sourceExtractor;
	}

	public final Resource getResource() {
		return this.resource;
	}

	public void fatal(String message, Object source) {
		fatal(message, source, null, null);
	}

	public void fatal(String message, Object source, Throwable ex) {
		fatal(message, source, null, ex);
	}

	public void fatal(String message, Object source, ParseState parseState) {
		fatal(message, source, parseState, null);
	}

	public void fatal(String message, Object source, ParseState parseState, Throwable cause) {
		Location location = new Location(getResource(), source);
		this.problemReporter.fatal(new Problem(message, location, parseState, cause));
	}

	public void error(String message, Object source) {
		error(message, source, null, null);
	}

	public void error(String message, Object source, Throwable ex) {
		error(message, source, null, ex);
	}

	public void error(String message, Object source, ParseState parseState) {
		error(message, source, parseState, null);
	}

	public void error(String message, Object source, ParseState parseState, Throwable cause) {
		Location location = new Location(getResource(), source);
		this.problemReporter.error(new Problem(message, location, parseState, cause));
	}

	public void warning(String message, Object source) {
		warning(message, source, null, null);
	}

	public void warning(String message, Object source, Throwable ex) {
		warning(message, source, null, ex);
	}

	public void warning(String message, Object source, ParseState parseState) {
		warning(message, source, parseState, null);
	}

	public void warning(String message, Object source, ParseState parseState, Throwable cause) {
		Location location = new Location(getResource(), source);
		this.problemReporter.warning(new Problem(message, location, parseState, cause));
	}

	public void fireDefaultsRegistered(DefaultsDefinition defaultsDefinition) {
		this.eventListener.defaultsRegistered(defaultsDefinition);
	}

	public void fireComponentRegistered(ComponentDefinition componentDefinition) {
		this.eventListener.componentRegistered(componentDefinition);
	}

	public void fireAliasRegistered(String beanName, String alias, Object source) {
		this.eventListener.aliasRegistered(new AliasDefinition(beanName, alias, source));
	}

	public void fireImportProcessed(String importedResource, Object source) {
		this.eventListener.importProcessed(new ImportDefinition(importedResource, source));
	}

	public void fireImportProcessed(String importedResource, Resource[] actualResources, Object source) {
		this.eventListener.importProcessed(new ImportDefinition(importedResource, actualResources, source));
	}

	public SourceExtractor getSourceExtractor() {
		return this.sourceExtractor;
	}

	public Object extractSource(Object sourceCandidate) {
		return this.sourceExtractor.extractSource(sourceCandidate, this.resource);
	}

}
```

### XmlReaderContext.java

```java
package org.springframework.beans.factory.xml;

import java.io.StringReader;

import org.w3c.dom.Document;
import org.xml.sax.InputSource;

import org.springframework.beans.factory.BeanDefinitionStoreException;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.parsing.ProblemReporter;
import org.springframework.beans.factory.parsing.ReaderContext;
import org.springframework.beans.factory.parsing.ReaderEventListener;
import org.springframework.beans.factory.parsing.SourceExtractor;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.core.env.Environment;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;

/**
 * XML读取器上下文
 */
public class XmlReaderContext extends ReaderContext {

	private final XmlBeanDefinitionReader reader;

	private final NamespaceHandlerResolver namespaceHandlerResolver;


	public XmlReaderContext(
			Resource resource, ProblemReporter problemReporter,
			ReaderEventListener eventListener, SourceExtractor sourceExtractor,
			XmlBeanDefinitionReader reader, NamespaceHandlerResolver namespaceHandlerResolver) {

		super(resource, problemReporter, eventListener, sourceExtractor);
		this.reader = reader;
		this.namespaceHandlerResolver = namespaceHandlerResolver;
	}


	public final XmlBeanDefinitionReader getReader() {
		return this.reader;
	}

	public final BeanDefinitionRegistry getRegistry() {
		return this.reader.getRegistry();
	}

	public final ResourceLoader getResourceLoader() {
		return this.reader.getResourceLoader();
	}

	public final ClassLoader getBeanClassLoader() {
		return this.reader.getBeanClassLoader();
	}

	public final Environment getEnvironment() {
		return this.reader.getEnvironment();
	}

	public final NamespaceHandlerResolver getNamespaceHandlerResolver() {
		return this.namespaceHandlerResolver;
	}


	public String generateBeanName(BeanDefinition beanDefinition) {
		return this.reader.getBeanNameGenerator().generateBeanName(beanDefinition, getRegistry());
	}

	public String registerWithGeneratedName(BeanDefinition beanDefinition) {
		String generatedName = generateBeanName(beanDefinition);
		getRegistry().registerBeanDefinition(generatedName, beanDefinition);
		return generatedName;
	}

	public Document readDocumentFromString(String documentContent) {
		InputSource is = new InputSource(new StringReader(documentContent));
		try {
			return this.reader.doLoadDocument(is, getResource());
		}
		catch (Exception ex) {
			throw new BeanDefinitionStoreException("Failed to read XML document", ex);
		}
	}

}
```

## BeanDefinitionReader.java

```java
package org.springframework.beans.factory.support;

import org.springframework.beans.factory.BeanDefinitionStoreException;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;

/**
 * BeanDefinition读取器。定义了四个组件获取方法，四个加载BeanDefinition的方法。
 */
public interface BeanDefinitionReader {

	BeanDefinitionRegistry getRegistry();

	ResourceLoader getResourceLoader();

	ClassLoader getBeanClassLoader();

	BeanNameGenerator getBeanNameGenerator();

	int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException;

	int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException;

	int loadBeanDefinitions(String location) throws BeanDefinitionStoreException;

	int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException;

}

```

### AbstractBeanDefinitionReader.java

```java
package org.springframework.beans.factory.support;

import java.io.IOException;
import java.util.Set;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import org.springframework.beans.factory.BeanDefinitionStoreException;
import org.springframework.core.env.Environment;
import org.springframework.core.env.EnvironmentCapable;
import org.springframework.core.env.StandardEnvironment;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;
import org.springframework.util.Assert;

/**
 * BeanDefinitionReader的抽象实现。添加了对环境的支持。子类只需要实现从Resource读取BeanDefinition。
 */
public abstract class AbstractBeanDefinitionReader implements EnvironmentCapable, BeanDefinitionReader {

	protected final Log logger = LogFactory.getLog(getClass());

	private final BeanDefinitionRegistry registry;

	private ResourceLoader resourceLoader;

	private ClassLoader beanClassLoader;

	private Environment environment;

	private BeanNameGenerator beanNameGenerator = new DefaultBeanNameGenerator();

	protected AbstractBeanDefinitionReader(BeanDefinitionRegistry registry) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		this.registry = registry;

		// Determine ResourceLoader to use.
		if (this.registry instanceof ResourceLoader) {
			this.resourceLoader = (ResourceLoader) this.registry;
		}
		else {
			this.resourceLoader = new PathMatchingResourcePatternResolver();
		}

		// Inherit Environment if possible
		if (this.registry instanceof EnvironmentCapable) {
			this.environment = ((EnvironmentCapable) this.registry).getEnvironment();
		}
		else {
			this.environment = new StandardEnvironment();
		}
	}


	public final BeanDefinitionRegistry getBeanFactory() {
		return this.registry;
	}

	@Override
	public final BeanDefinitionRegistry getRegistry() {
		return this.registry;
	}

	public void setResourceLoader(ResourceLoader resourceLoader) {
		this.resourceLoader = resourceLoader;
	}

	@Override
	public ResourceLoader getResourceLoader() {
		return this.resourceLoader;
	}

	public void setBeanClassLoader(ClassLoader beanClassLoader) {
		this.beanClassLoader = beanClassLoader;
	}

	@Override
	public ClassLoader getBeanClassLoader() {
		return this.beanClassLoader;
	}

	public void setEnvironment(Environment environment) {
		this.environment = environment;
	}

	@Override
	public Environment getEnvironment() {
		return this.environment;
	}

	public void setBeanNameGenerator(BeanNameGenerator beanNameGenerator) {
		this.beanNameGenerator = (beanNameGenerator != null ? beanNameGenerator : new DefaultBeanNameGenerator());
	}

	@Override
	public BeanNameGenerator getBeanNameGenerator() {
		return this.beanNameGenerator;
	}


	@Override
	public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
		Assert.notNull(resources, "Resource array must not be null");
		int counter = 0;
		for (Resource resource : resources) {
			counter += loadBeanDefinitions(resource);
		}
		return counter;
	}

	@Override
	public int loadBeanDefinitions(String location) throws BeanDefinitionStoreException {
		return loadBeanDefinitions(location, null);
	}

	public int loadBeanDefinitions(String location, Set<Resource> actualResources) throws BeanDefinitionStoreException {
		ResourceLoader resourceLoader = getResourceLoader();
		if (resourceLoader == null) {
			throw new BeanDefinitionStoreException(
					"Cannot import bean definitions from location [" + location + "]: no ResourceLoader available");
		}

		if (resourceLoader instanceof ResourcePatternResolver) {
			// Resource pattern matching available.
			try {
				Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
				int loadCount = loadBeanDefinitions(resources);
				if (actualResources != null) {
					for (Resource resource : resources) {
						actualResources.add(resource);
					}
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Loaded " + loadCount + " bean definitions from location pattern [" + location + "]");
				}
				return loadCount;
			}
			catch (IOException ex) {
				throw new BeanDefinitionStoreException(
						"Could not resolve bean definition resource pattern [" + location + "]", ex);
			}
		}
		else {
			// Can only load single resources by absolute URL.
			Resource resource = resourceLoader.getResource(location);
			int loadCount = loadBeanDefinitions(resource);
			if (actualResources != null) {
				actualResources.add(resource);
			}
			if (logger.isDebugEnabled()) {
				logger.debug("Loaded " + loadCount + " bean definitions from location [" + location + "]");
			}
			return loadCount;
		}
	}

	@Override
	public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
		Assert.notNull(locations, "Location array must not be null");
		int counter = 0;
		for (String location : locations) {
			counter += loadBeanDefinitions(location);
		}
		return counter;
	}

}
```

#### XmlBeanDefinitionReader.java

```java
package org.springframework.beans.factory.xml;
/**
 * 从XML中读取BeanDefinition的读取器
 */
public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader {

	public static final int VALIDATION_NONE = XmlValidationModeDetector.VALIDATION_NONE;

	public static final int VALIDATION_AUTO = XmlValidationModeDetector.VALIDATION_AUTO;

	public static final int VALIDATION_DTD = XmlValidationModeDetector.VALIDATION_DTD;

	public static final int VALIDATION_XSD = XmlValidationModeDetector.VALIDATION_XSD;

	private static final Constants constants = new Constants(XmlBeanDefinitionReader.class);

	private int validationMode = VALIDATION_AUTO;

	private boolean namespaceAware = false;

	private Class<?> documentReaderClass = DefaultBeanDefinitionDocumentReader.class;

	private ProblemReporter problemReporter = new FailFastProblemReporter();

	private ReaderEventListener eventListener = new EmptyReaderEventListener();

	private SourceExtractor sourceExtractor = new NullSourceExtractor();

	private NamespaceHandlerResolver namespaceHandlerResolver;

	private DocumentLoader documentLoader = new DefaultDocumentLoader();

	private EntityResolver entityResolver;

	private ErrorHandler errorHandler = new SimpleSaxErrorHandler(logger);

	private final XmlValidationModeDetector validationModeDetector = new XmlValidationModeDetector();

	private final ThreadLocal<Set<EncodedResource>> resourcesCurrentlyBeingLoaded =
			new NamedThreadLocal<Set<EncodedResource>>("XML bean definition resources currently being loaded");

	public XmlBeanDefinitionReader(BeanDefinitionRegistry registry) {
		super(registry);
	}

	public void setValidating(boolean validating) {
		this.validationMode = (validating ? VALIDATION_AUTO : VALIDATION_NONE);
		this.namespaceAware = !validating;
	}

	public void setValidationModeName(String validationModeName) {
		setValidationMode(constants.asNumber(validationModeName).intValue());
	}

	public void setValidationMode(int validationMode) {
		this.validationMode = validationMode;
	}

	public int getValidationMode() {
		return this.validationMode;
	}

	public void setNamespaceAware(boolean namespaceAware) {
		this.namespaceAware = namespaceAware;
	}

	public boolean isNamespaceAware() {
		return this.namespaceAware;
	}

	public void setProblemReporter(ProblemReporter problemReporter) {
		this.problemReporter = (problemReporter != null ? problemReporter : new FailFastProblemReporter());
	}

	public void setEventListener(ReaderEventListener eventListener) {
		this.eventListener = (eventListener != null ? eventListener : new EmptyReaderEventListener());
	}

	public void setSourceExtractor(SourceExtractor sourceExtractor) {
		this.sourceExtractor = (sourceExtractor != null ? sourceExtractor : new NullSourceExtractor());
	}

	public void setNamespaceHandlerResolver(NamespaceHandlerResolver namespaceHandlerResolver) {
		this.namespaceHandlerResolver = namespaceHandlerResolver;
	}

	public void setDocumentLoader(DocumentLoader documentLoader) {
		this.documentLoader = (documentLoader != null ? documentLoader : new DefaultDocumentLoader());
	}

	public void setEntityResolver(EntityResolver entityResolver) {
		this.entityResolver = entityResolver;
	}

	protected EntityResolver getEntityResolver() {
		if (this.entityResolver == null) {
			// Determine default EntityResolver to use.
			ResourceLoader resourceLoader = getResourceLoader();
			if (resourceLoader != null) {
				this.entityResolver = new ResourceEntityResolver(resourceLoader);
			}
			else {
				this.entityResolver = new DelegatingEntityResolver(getBeanClassLoader());
			}
		}
		return this.entityResolver;
	}

	public void setErrorHandler(ErrorHandler errorHandler) {
		this.errorHandler = errorHandler;
	}

	public void setDocumentReaderClass(Class<?> documentReaderClass) {
		if (documentReaderClass == null || !BeanDefinitionDocumentReader.class.isAssignableFrom(documentReaderClass)) {
			throw new IllegalArgumentException(
					"documentReaderClass must be an implementation of the BeanDefinitionDocumentReader interface");
		}
		this.documentReaderClass = documentReaderClass;
	}

	@Override
	public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
		return loadBeanDefinitions(new EncodedResource(resource));
	}

	public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
		Assert.notNull(encodedResource, "EncodedResource must not be null");
		if (logger.isInfoEnabled()) {
			logger.info("Loading XML bean definitions from " + encodedResource);
		}

		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
		if (currentResources == null) {
			currentResources = new HashSet<EncodedResource>(4);
			this.resourcesCurrentlyBeingLoaded.set(currentResources);
		}
		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
		}
		try {
			InputStream inputStream = encodedResource.getResource().getInputStream();
			try {
				InputSource inputSource = new InputSource(inputStream);
				if (encodedResource.getEncoding() != null) {
					inputSource.setEncoding(encodedResource.getEncoding());
				}
				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
			}
			finally {
				inputStream.close();
			}
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"IOException parsing XML document from " + encodedResource.getResource(), ex);
		}
		finally {
			currentResources.remove(encodedResource);
			if (currentResources.isEmpty()) {
				this.resourcesCurrentlyBeingLoaded.remove();
			}
		}
	}

	public int loadBeanDefinitions(InputSource inputSource) throws BeanDefinitionStoreException {
		return loadBeanDefinitions(inputSource, "resource loaded through SAX InputSource");
	}

	public int loadBeanDefinitions(InputSource inputSource, String resourceDescription)
			throws BeanDefinitionStoreException {

		return doLoadBeanDefinitions(inputSource, new DescriptiveResource(resourceDescription));
	}

	protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {
		try {
			Document doc = doLoadDocument(inputSource, resource);
			return registerBeanDefinitions(doc, resource);
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (SAXParseException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
		}
		catch (SAXException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"XML document from " + resource + " is invalid", ex);
		}
		catch (ParserConfigurationException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Parser configuration exception parsing XML from " + resource, ex);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"IOException parsing XML document from " + resource, ex);
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Unexpected exception parsing XML document from " + resource, ex);
		}
	}

	protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
		return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
				getValidationModeForResource(resource), isNamespaceAware());
	}

	protected int getValidationModeForResource(Resource resource) {
		int validationModeToUse = getValidationMode();
		if (validationModeToUse != VALIDATION_AUTO) {
			return validationModeToUse;
		}
		int detectedMode = detectValidationMode(resource);
		if (detectedMode != VALIDATION_AUTO) {
			return detectedMode;
		}
		// Hmm, we didn't get a clear indication... Let's assume XSD,
		// since apparently no DTD declaration has been found up until
		// detection stopped (before finding the document's root tag).
		return VALIDATION_XSD;
	}

	protected int detectValidationMode(Resource resource) {
		if (resource.isOpen()) {
			throw new BeanDefinitionStoreException(
					"Passed-in Resource [" + resource + "] contains an open stream: " +
					"cannot determine validation mode automatically. Either pass in a Resource " +
					"that is able to create fresh streams, or explicitly specify the validationMode " +
					"on your XmlBeanDefinitionReader instance.");
		}

		InputStream inputStream;
		try {
			inputStream = resource.getInputStream();
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"Unable to determine validation mode for [" + resource + "]: cannot open InputStream. " +
					"Did you attempt to load directly from a SAX InputSource without specifying the " +
					"validationMode on your XmlBeanDefinitionReader instance?", ex);
		}

		try {
			return this.validationModeDetector.detectValidationMode(inputStream);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException("Unable to determine validation mode for [" +
					resource + "]: an error occurred whilst reading from the InputStream.", ex);
		}
	}

	public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		int countBefore = getRegistry().getBeanDefinitionCount();
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}


	protected BeanDefinitionDocumentReader createBeanDefinitionDocumentReader() {
		return BeanDefinitionDocumentReader.class.cast(BeanUtils.instantiateClass(this.documentReaderClass));
	}

	public XmlReaderContext createReaderContext(Resource resource) {
		return new XmlReaderContext(resource, this.problemReporter, this.eventListener,
				this.sourceExtractor, this, getNamespaceHandlerResolver());
	}

	public NamespaceHandlerResolver getNamespaceHandlerResolver() {
		if (this.namespaceHandlerResolver == null) {
			this.namespaceHandlerResolver = createDefaultNamespaceHandlerResolver();
		}
		return this.namespaceHandlerResolver;
	}

	protected NamespaceHandlerResolver createDefaultNamespaceHandlerResolver() {
		return new DefaultNamespaceHandlerResolver(getResourceLoader().getClassLoader());
	}

}
```
## DefaultsDefinition.java

```java
package org.springframework.beans.factory.parsing;

import org.springframework.beans.BeanMetadataElement;

/**
 * 默认定义的标记接口
 */
public interface DefaultsDefinition extends BeanMetadataElement {

}

```

### DocumentDefaultsDefinition.java

```java
package org.springframework.beans.factory.xml;

import org.springframework.beans.factory.parsing.DefaultsDefinition;

/**
 * Document的默认定义
 */
public class DocumentDefaultsDefinition implements DefaultsDefinition {

	private String lazyInit;

	private String merge;

	private String autowire;

	private String dependencyCheck;

	private String autowireCandidates;

	private String initMethod;

	private String destroyMethod;

	private Object source;
//省略getter和setter

}

```

## ParseState.java

```java
package org.springframework.beans.factory.parsing;

import java.util.Stack;

/**
 * 解析状态
 */
public final class ParseState {

	private static final char TAB = '\t';

	private final Stack<Entry> state;

	public ParseState() {
		this.state = new Stack<Entry>();
	}

	@SuppressWarnings("unchecked")
	private ParseState(ParseState other) {
		this.state = (Stack<Entry>) other.state.clone();
	}

	public void push(Entry entry) {
		this.state.push(entry);
	}

	public void pop() {
		this.state.pop();
	}

	public Entry peek() {
		return this.state.empty() ? null : this.state.peek();
	}

	public ParseState snapshot() {
		return new ParseState(this);
	}

	@Override
	public String toString() {
		StringBuilder sb = new StringBuilder();
		for (int x = 0; x < this.state.size(); x++) {
			if (x > 0) {
				sb.append('\n');
				for (int y = 0; y < x; y++) {
					sb.append(TAB);
				}
				sb.append("-> ");
			}
			sb.append(this.state.get(x));
		}
		return sb.toString();
	}

	public interface Entry {

	}

}

```



## BeanDefinitionParserDelegate.java

```java
package org.springframework.beans.factory.xml;
/**
 * BeanDefinition解析器代理。自身是有状态的。
 * 真正解析Element为BeanDefinition的代理。
 */
public class BeanDefinitionParserDelegate {

	public static final String BEANS_NAMESPACE_URI = "http://www.springframework.org/schema/beans";

	public static final String MULTI_VALUE_ATTRIBUTE_DELIMITERS = ",; ";

	public static final String TRUE_VALUE = "true";

	public static final String FALSE_VALUE = "false";

	public static final String DEFAULT_VALUE = "default";

	public static final String DESCRIPTION_ELEMENT = "description";

	public static final String AUTOWIRE_NO_VALUE = "no";

	public static final String AUTOWIRE_BY_NAME_VALUE = "byName";

	public static final String AUTOWIRE_BY_TYPE_VALUE = "byType";

	public static final String AUTOWIRE_CONSTRUCTOR_VALUE = "constructor";

	public static final String AUTOWIRE_AUTODETECT_VALUE = "autodetect";

	public static final String DEPENDENCY_CHECK_ALL_ATTRIBUTE_VALUE = "all";

	public static final String DEPENDENCY_CHECK_SIMPLE_ATTRIBUTE_VALUE = "simple";

	public static final String DEPENDENCY_CHECK_OBJECTS_ATTRIBUTE_VALUE = "objects";

	public static final String NAME_ATTRIBUTE = "name";

	public static final String BEAN_ELEMENT = "bean";

	public static final String META_ELEMENT = "meta";

	public static final String ID_ATTRIBUTE = "id";

	public static final String PARENT_ATTRIBUTE = "parent";

	public static final String CLASS_ATTRIBUTE = "class";

	public static final String ABSTRACT_ATTRIBUTE = "abstract";

	public static final String SCOPE_ATTRIBUTE = "scope";

	private static final String SINGLETON_ATTRIBUTE = "singleton";

	public static final String LAZY_INIT_ATTRIBUTE = "lazy-init";

	public static final String AUTOWIRE_ATTRIBUTE = "autowire";

	public static final String AUTOWIRE_CANDIDATE_ATTRIBUTE = "autowire-candidate";

	public static final String PRIMARY_ATTRIBUTE = "primary";

	public static final String DEPENDENCY_CHECK_ATTRIBUTE = "dependency-check";

	public static final String DEPENDS_ON_ATTRIBUTE = "depends-on";

	public static final String INIT_METHOD_ATTRIBUTE = "init-method";

	public static final String DESTROY_METHOD_ATTRIBUTE = "destroy-method";

	public static final String FACTORY_METHOD_ATTRIBUTE = "factory-method";

	public static final String FACTORY_BEAN_ATTRIBUTE = "factory-bean";

	public static final String CONSTRUCTOR_ARG_ELEMENT = "constructor-arg";

	public static final String INDEX_ATTRIBUTE = "index";

	public static final String TYPE_ATTRIBUTE = "type";

	public static final String VALUE_TYPE_ATTRIBUTE = "value-type";

	public static final String KEY_TYPE_ATTRIBUTE = "key-type";

	public static final String PROPERTY_ELEMENT = "property";

	public static final String REF_ATTRIBUTE = "ref";

	public static final String VALUE_ATTRIBUTE = "value";

	public static final String LOOKUP_METHOD_ELEMENT = "lookup-method";

	public static final String REPLACED_METHOD_ELEMENT = "replaced-method";

	public static final String REPLACER_ATTRIBUTE = "replacer";

	public static final String ARG_TYPE_ELEMENT = "arg-type";

	public static final String ARG_TYPE_MATCH_ATTRIBUTE = "match";

	public static final String REF_ELEMENT = "ref";

	public static final String IDREF_ELEMENT = "idref";

	public static final String BEAN_REF_ATTRIBUTE = "bean";

	public static final String LOCAL_REF_ATTRIBUTE = "local";

	public static final String PARENT_REF_ATTRIBUTE = "parent";

	public static final String VALUE_ELEMENT = "value";

	public static final String NULL_ELEMENT = "null";

	public static final String ARRAY_ELEMENT = "array";

	public static final String LIST_ELEMENT = "list";

	public static final String SET_ELEMENT = "set";

	public static final String MAP_ELEMENT = "map";

	public static final String ENTRY_ELEMENT = "entry";

	public static final String KEY_ELEMENT = "key";

	public static final String KEY_ATTRIBUTE = "key";

	public static final String KEY_REF_ATTRIBUTE = "key-ref";

	public static final String VALUE_REF_ATTRIBUTE = "value-ref";

	public static final String PROPS_ELEMENT = "props";

	public static final String PROP_ELEMENT = "prop";

	public static final String MERGE_ATTRIBUTE = "merge";

	public static final String QUALIFIER_ELEMENT = "qualifier";

	public static final String QUALIFIER_ATTRIBUTE_ELEMENT = "attribute";

	public static final String DEFAULT_LAZY_INIT_ATTRIBUTE = "default-lazy-init";

	public static final String DEFAULT_MERGE_ATTRIBUTE = "default-merge";

	public static final String DEFAULT_AUTOWIRE_ATTRIBUTE = "default-autowire";

	public static final String DEFAULT_DEPENDENCY_CHECK_ATTRIBUTE = "default-dependency-check";

	public static final String DEFAULT_AUTOWIRE_CANDIDATES_ATTRIBUTE = "default-autowire-candidates";

	public static final String DEFAULT_INIT_METHOD_ATTRIBUTE = "default-init-method";

	public static final String DEFAULT_DESTROY_METHOD_ATTRIBUTE = "default-destroy-method";


	protected final Log logger = LogFactory.getLog(getClass());

	private final XmlReaderContext readerContext;

	private final DocumentDefaultsDefinition defaults = new DocumentDefaultsDefinition();

	private final ParseState parseState = new ParseState();

	/**
	 * Stores all used bean names so we can enforce uniqueness on a per
	 * beans-element basis. Duplicate bean ids/names may not exist within the
	 * same level of beans element nesting, but may be duplicated across levels.
	 */
	private final Set<String> usedNames = new HashSet<String>();


	/**
	 * Create a new BeanDefinitionParserDelegate associated with the supplied
	 * {@link XmlReaderContext}.
	 */
	public BeanDefinitionParserDelegate(XmlReaderContext readerContext) {
		Assert.notNull(readerContext, "XmlReaderContext must not be null");
		this.readerContext = readerContext;
	}


	/**
	 * Get the {@link XmlReaderContext} associated with this helper instance.
	 */
	public final XmlReaderContext getReaderContext() {
		return this.readerContext;
	}

	/**
	 * Get the {@link Environment} associated with this helper instance.
	 * @deprecated in favor of {@link XmlReaderContext#getEnvironment()}
	 */
	@Deprecated
	public final Environment getEnvironment() {
		return this.readerContext.getEnvironment();
	}

	/**
	 * Invoke the {@link org.springframework.beans.factory.parsing.SourceExtractor} to pull the
	 * source metadata from the supplied {@link Element}.
	 */
	protected Object extractSource(Element ele) {
		return this.readerContext.extractSource(ele);
	}

	/**
	 * Report an error with the given message for the given source element.
	 */
	protected void error(String message, Node source) {
		this.readerContext.error(message, source, this.parseState.snapshot());
	}

	/**
	 * Report an error with the given message for the given source element.
	 */
	protected void error(String message, Element source) {
		this.readerContext.error(message, source, this.parseState.snapshot());
	}

	/**
	 * Report an error with the given message for the given source element.
	 */
	protected void error(String message, Element source, Throwable cause) {
		this.readerContext.error(message, source, this.parseState.snapshot(), cause);
	}


	/**
	 * Initialize the default settings assuming a {@code null} parent delegate.
	 */
	public void initDefaults(Element root) {
		initDefaults(root, null);
	}

	/**
	 * Initialize the default lazy-init, autowire, dependency check settings,
	 * init-method, destroy-method and merge settings. Support nested 'beans'
	 * element use cases by falling back to the given parent in case the
	 * defaults are not explicitly set locally.
	 * @see #populateDefaults(DocumentDefaultsDefinition, DocumentDefaultsDefinition, org.w3c.dom.Element)
	 * @see #getDefaults()
	 */
	public void initDefaults(Element root, BeanDefinitionParserDelegate parent) {
		populateDefaults(this.defaults, (parent != null ? parent.defaults : null), root);
		this.readerContext.fireDefaultsRegistered(this.defaults);
	}

	/**
	 * Populate the given DocumentDefaultsDefinition instance with the default lazy-init,
	 * autowire, dependency check settings, init-method, destroy-method and merge settings.
	 * Support nested 'beans' element use cases by falling back to <literal>parentDefaults</literal>
	 * in case the defaults are not explicitly set locally.
	 * @param defaults the defaults to populate
	 * @param parentDefaults the parent BeanDefinitionParserDelegate (if any) defaults to fall back to
	 * @param root the root element of the current bean definition document (or nested beans element)
	 */
	protected void populateDefaults(DocumentDefaultsDefinition defaults, DocumentDefaultsDefinition parentDefaults, Element root) {
		String lazyInit = root.getAttribute(DEFAULT_LAZY_INIT_ATTRIBUTE);
		if (DEFAULT_VALUE.equals(lazyInit)) {
			// Potentially inherited from outer <beans> sections, otherwise falling back to false.
			lazyInit = (parentDefaults != null ? parentDefaults.getLazyInit() : FALSE_VALUE);
		}
		defaults.setLazyInit(lazyInit);

		String merge = root.getAttribute(DEFAULT_MERGE_ATTRIBUTE);
		if (DEFAULT_VALUE.equals(merge)) {
			// Potentially inherited from outer <beans> sections, otherwise falling back to false.
			merge = (parentDefaults != null ? parentDefaults.getMerge() : FALSE_VALUE);
		}
		defaults.setMerge(merge);

		String autowire = root.getAttribute(DEFAULT_AUTOWIRE_ATTRIBUTE);
		if (DEFAULT_VALUE.equals(autowire)) {
			// Potentially inherited from outer <beans> sections, otherwise falling back to 'no'.
			autowire = (parentDefaults != null ? parentDefaults.getAutowire() : AUTOWIRE_NO_VALUE);
		}
		defaults.setAutowire(autowire);

		// Don't fall back to parentDefaults for dependency-check as it's no longer supported in
		// <beans> as of 3.0. Therefore, no nested <beans> would ever need to fall back to it.
		defaults.setDependencyCheck(root.getAttribute(DEFAULT_DEPENDENCY_CHECK_ATTRIBUTE));

		if (root.hasAttribute(DEFAULT_AUTOWIRE_CANDIDATES_ATTRIBUTE)) {
			defaults.setAutowireCandidates(root.getAttribute(DEFAULT_AUTOWIRE_CANDIDATES_ATTRIBUTE));
		}
		else if (parentDefaults != null) {
			defaults.setAutowireCandidates(parentDefaults.getAutowireCandidates());
		}

		if (root.hasAttribute(DEFAULT_INIT_METHOD_ATTRIBUTE)) {
			defaults.setInitMethod(root.getAttribute(DEFAULT_INIT_METHOD_ATTRIBUTE));
		}
		else if (parentDefaults != null) {
			defaults.setInitMethod(parentDefaults.getInitMethod());
		}

		if (root.hasAttribute(DEFAULT_DESTROY_METHOD_ATTRIBUTE)) {
			defaults.setDestroyMethod(root.getAttribute(DEFAULT_DESTROY_METHOD_ATTRIBUTE));
		}
		else if (parentDefaults != null) {
			defaults.setDestroyMethod(parentDefaults.getDestroyMethod());
		}

		defaults.setSource(this.readerContext.extractSource(root));
	}

	/**
	 * Return the defaults definition object.
	 */
	public DocumentDefaultsDefinition getDefaults() {
		return this.defaults;
	}

	/**
	 * Return the default settings for bean definitions as indicated within
	 * the attributes of the top-level {@code <beans/>} element.
	 */
	public BeanDefinitionDefaults getBeanDefinitionDefaults() {
		BeanDefinitionDefaults bdd = new BeanDefinitionDefaults();
		bdd.setLazyInit("TRUE".equalsIgnoreCase(this.defaults.getLazyInit()));
		bdd.setDependencyCheck(getDependencyCheck(DEFAULT_VALUE));
		bdd.setAutowireMode(getAutowireMode(DEFAULT_VALUE));
		bdd.setInitMethodName(this.defaults.getInitMethod());
		bdd.setDestroyMethodName(this.defaults.getDestroyMethod());
		return bdd;
	}

	/**
	 * Return any patterns provided in the 'default-autowire-candidates'
	 * attribute of the top-level {@code <beans/>} element.
	 */
	public String[] getAutowireCandidatePatterns() {
		String candidatePattern = this.defaults.getAutowireCandidates();
		return (candidatePattern != null ? StringUtils.commaDelimitedListToStringArray(candidatePattern) : null);
	}


	/**
	 * Parses the supplied {@code <bean>} element. May return {@code null}
	 * if there were errors during parse. Errors are reported to the
	 * {@link org.springframework.beans.factory.parsing.ProblemReporter}.
	 */
	public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {
		return parseBeanDefinitionElement(ele, null);
	}

	/**
	 * Parses the supplied {@code <bean>} element. May return {@code null}
	 * if there were errors during parse. Errors are reported to the
	 * {@link org.springframework.beans.factory.parsing.ProblemReporter}.
	 */
	public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
		String id = ele.getAttribute(ID_ATTRIBUTE);
		String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);

		List<String> aliases = new ArrayList<String>();
		if (StringUtils.hasLength(nameAttr)) {
			String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
			aliases.addAll(Arrays.asList(nameArr));
		}

		String beanName = id;
		if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
			beanName = aliases.remove(0);
			if (logger.isDebugEnabled()) {
				logger.debug("No XML 'id' specified - using '" + beanName +
						"' as bean name and " + aliases + " as aliases");
			}
		}

		if (containingBean == null) {
			checkNameUniqueness(beanName, aliases, ele);
		}

		AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
		if (beanDefinition != null) {
			if (!StringUtils.hasText(beanName)) {
				try {
					if (containingBean != null) {
						beanName = BeanDefinitionReaderUtils.generateBeanName(
								beanDefinition, this.readerContext.getRegistry(), true);
					}
					else {
						beanName = this.readerContext.generateBeanName(beanDefinition);
						// Register an alias for the plain bean class name, if still possible,
						// if the generator returned the class name plus a suffix.
						// This is expected for Spring 1.2/2.0 backwards compatibility.
						String beanClassName = beanDefinition.getBeanClassName();
						if (beanClassName != null &&
								beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&
								!this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
							aliases.add(beanClassName);
						}
					}
					if (logger.isDebugEnabled()) {
						logger.debug("Neither XML 'id' nor 'name' specified - " +
								"using generated bean name [" + beanName + "]");
					}
				}
				catch (Exception ex) {
					error(ex.getMessage(), ele);
					return null;
				}
			}
			String[] aliasesArray = StringUtils.toStringArray(aliases);
			return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
		}

		return null;
	}

	/**
	 * Validate that the specified bean name and aliases have not been used already
	 * within the current level of beans element nesting.
	 */
	protected void checkNameUniqueness(String beanName, List<String> aliases, Element beanElement) {
		String foundName = null;

		if (StringUtils.hasText(beanName) && this.usedNames.contains(beanName)) {
			foundName = beanName;
		}
		if (foundName == null) {
			foundName = CollectionUtils.findFirstMatch(this.usedNames, aliases);
		}
		if (foundName != null) {
			error("Bean name '" + foundName + "' is already used in this <beans> element", beanElement);
		}

		this.usedNames.add(beanName);
		this.usedNames.addAll(aliases);
	}

	/**
	 * Parse the bean definition itself, without regard to name or aliases. May return
	 * {@code null} if problems occurred during the parsing of the bean definition.
	 */
	public AbstractBeanDefinition parseBeanDefinitionElement(
			Element ele, String beanName, BeanDefinition containingBean) {

		this.parseState.push(new BeanEntry(beanName));

		String className = null;
		if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
			className = ele.getAttribute(CLASS_ATTRIBUTE).trim();
		}

		try {
			String parent = null;
			if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
				parent = ele.getAttribute(PARENT_ATTRIBUTE);
			}
			AbstractBeanDefinition bd = createBeanDefinition(className, parent);

			parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
			bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));

			parseMetaElements(ele, bd);
			parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
			parseReplacedMethodSubElements(ele, bd.getMethodOverrides());

			parseConstructorArgElements(ele, bd);
			parsePropertyElements(ele, bd);
			parseQualifierElements(ele, bd);

			bd.setResource(this.readerContext.getResource());
			bd.setSource(extractSource(ele));

			return bd;
		}
		catch (ClassNotFoundException ex) {
			error("Bean class [" + className + "] not found", ele, ex);
		}
		catch (NoClassDefFoundError err) {
			error("Class that bean class [" + className + "] depends on not found", ele, err);
		}
		catch (Throwable ex) {
			error("Unexpected failure during bean definition parsing", ele, ex);
		}
		finally {
			this.parseState.pop();
		}

		return null;
	}

	/**
	 * Apply the attributes of the given bean element to the given bean * definition.
	 * @param ele bean declaration element
	 * @param beanName bean name
	 * @param containingBean containing bean definition
	 * @return a bean definition initialized according to the bean element attributes
	 */
	public AbstractBeanDefinition parseBeanDefinitionAttributes(Element ele, String beanName,
			BeanDefinition containingBean, AbstractBeanDefinition bd) {

		if (ele.hasAttribute(SINGLETON_ATTRIBUTE)) {
			error("Old 1.x 'singleton' attribute in use - upgrade to 'scope' declaration", ele);
		}
		else if (ele.hasAttribute(SCOPE_ATTRIBUTE)) {
			bd.setScope(ele.getAttribute(SCOPE_ATTRIBUTE));
		}
		else if (containingBean != null) {
			// Take default from containing bean in case of an inner bean definition.
			bd.setScope(containingBean.getScope());
		}

		if (ele.hasAttribute(ABSTRACT_ATTRIBUTE)) {
			bd.setAbstract(TRUE_VALUE.equals(ele.getAttribute(ABSTRACT_ATTRIBUTE)));
		}

		String lazyInit = ele.getAttribute(LAZY_INIT_ATTRIBUTE);
		if (DEFAULT_VALUE.equals(lazyInit)) {
			lazyInit = this.defaults.getLazyInit();
		}
		bd.setLazyInit(TRUE_VALUE.equals(lazyInit));

		String autowire = ele.getAttribute(AUTOWIRE_ATTRIBUTE);
		bd.setAutowireMode(getAutowireMode(autowire));

		String dependencyCheck = ele.getAttribute(DEPENDENCY_CHECK_ATTRIBUTE);
		bd.setDependencyCheck(getDependencyCheck(dependencyCheck));

		if (ele.hasAttribute(DEPENDS_ON_ATTRIBUTE)) {
			String dependsOn = ele.getAttribute(DEPENDS_ON_ATTRIBUTE);
			bd.setDependsOn(StringUtils.tokenizeToStringArray(dependsOn, MULTI_VALUE_ATTRIBUTE_DELIMITERS));
		}

		String autowireCandidate = ele.getAttribute(AUTOWIRE_CANDIDATE_ATTRIBUTE);
		if ("".equals(autowireCandidate) || DEFAULT_VALUE.equals(autowireCandidate)) {
			String candidatePattern = this.defaults.getAutowireCandidates();
			if (candidatePattern != null) {
				String[] patterns = StringUtils.commaDelimitedListToStringArray(candidatePattern);
				bd.setAutowireCandidate(PatternMatchUtils.simpleMatch(patterns, beanName));
			}
		}
		else {
			bd.setAutowireCandidate(TRUE_VALUE.equals(autowireCandidate));
		}

		if (ele.hasAttribute(PRIMARY_ATTRIBUTE)) {
			bd.setPrimary(TRUE_VALUE.equals(ele.getAttribute(PRIMARY_ATTRIBUTE)));
		}

		if (ele.hasAttribute(INIT_METHOD_ATTRIBUTE)) {
			String initMethodName = ele.getAttribute(INIT_METHOD_ATTRIBUTE);
			if (!"".equals(initMethodName)) {
				bd.setInitMethodName(initMethodName);
			}
		}
		else {
			if (this.defaults.getInitMethod() != null) {
				bd.setInitMethodName(this.defaults.getInitMethod());
				bd.setEnforceInitMethod(false);
			}
		}

		if (ele.hasAttribute(DESTROY_METHOD_ATTRIBUTE)) {
			String destroyMethodName = ele.getAttribute(DESTROY_METHOD_ATTRIBUTE);
			bd.setDestroyMethodName(destroyMethodName);
		}
		else {
			if (this.defaults.getDestroyMethod() != null) {
				bd.setDestroyMethodName(this.defaults.getDestroyMethod());
				bd.setEnforceDestroyMethod(false);
			}
		}

		if (ele.hasAttribute(FACTORY_METHOD_ATTRIBUTE)) {
			bd.setFactoryMethodName(ele.getAttribute(FACTORY_METHOD_ATTRIBUTE));
		}
		if (ele.hasAttribute(FACTORY_BEAN_ATTRIBUTE)) {
			bd.setFactoryBeanName(ele.getAttribute(FACTORY_BEAN_ATTRIBUTE));
		}

		return bd;
	}

	/**
	 * Create a bean definition for the given class name and parent name.
	 * @param className the name of the bean class
	 * @param parentName the name of the bean's parent bean
	 * @return the newly created bean definition
	 * @throws ClassNotFoundException if bean class resolution was attempted but failed
	 */
	protected AbstractBeanDefinition createBeanDefinition(String className, String parentName)
			throws ClassNotFoundException {

		return BeanDefinitionReaderUtils.createBeanDefinition(
				parentName, className, this.readerContext.getBeanClassLoader());
	}

	public void parseMetaElements(Element ele, BeanMetadataAttributeAccessor attributeAccessor) {
		NodeList nl = ele.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (isCandidateElement(node) && nodeNameEquals(node, META_ELEMENT)) {
				Element metaElement = (Element) node;
				String key = metaElement.getAttribute(KEY_ATTRIBUTE);
				String value = metaElement.getAttribute(VALUE_ATTRIBUTE);
				BeanMetadataAttribute attribute = new BeanMetadataAttribute(key, value);
				attribute.setSource(extractSource(metaElement));
				attributeAccessor.addMetadataAttribute(attribute);
			}
		}
	}

	@SuppressWarnings("deprecation")
	public int getAutowireMode(String attValue) {
		String att = attValue;
		if (DEFAULT_VALUE.equals(att)) {
			att = this.defaults.getAutowire();
		}
		int autowire = AbstractBeanDefinition.AUTOWIRE_NO;
		if (AUTOWIRE_BY_NAME_VALUE.equals(att)) {
			autowire = AbstractBeanDefinition.AUTOWIRE_BY_NAME;
		}
		else if (AUTOWIRE_BY_TYPE_VALUE.equals(att)) {
			autowire = AbstractBeanDefinition.AUTOWIRE_BY_TYPE;
		}
		else if (AUTOWIRE_CONSTRUCTOR_VALUE.equals(att)) {
			autowire = AbstractBeanDefinition.AUTOWIRE_CONSTRUCTOR;
		}
		else if (AUTOWIRE_AUTODETECT_VALUE.equals(att)) {
			autowire = AbstractBeanDefinition.AUTOWIRE_AUTODETECT;
		}
		// Else leave default value.
		return autowire;
	}

	public int getDependencyCheck(String attValue) {
		String att = attValue;
		if (DEFAULT_VALUE.equals(att)) {
			att = this.defaults.getDependencyCheck();
		}
		if (DEPENDENCY_CHECK_ALL_ATTRIBUTE_VALUE.equals(att)) {
			return AbstractBeanDefinition.DEPENDENCY_CHECK_ALL;
		}
		else if (DEPENDENCY_CHECK_OBJECTS_ATTRIBUTE_VALUE.equals(att)) {
			return AbstractBeanDefinition.DEPENDENCY_CHECK_OBJECTS;
		}
		else if (DEPENDENCY_CHECK_SIMPLE_ATTRIBUTE_VALUE.equals(att)) {
			return AbstractBeanDefinition.DEPENDENCY_CHECK_SIMPLE;
		}
		else {
			return AbstractBeanDefinition.DEPENDENCY_CHECK_NONE;
		}
	}

	/**
	 * Parse constructor-arg sub-elements of the given bean element.
	 */
	public void parseConstructorArgElements(Element beanEle, BeanDefinition bd) {
		NodeList nl = beanEle.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (isCandidateElement(node) && nodeNameEquals(node, CONSTRUCTOR_ARG_ELEMENT)) {
				parseConstructorArgElement((Element) node, bd);
			}
		}
	}

	/**
	 * Parse property sub-elements of the given bean element.
	 */
	public void parsePropertyElements(Element beanEle, BeanDefinition bd) {
		NodeList nl = beanEle.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (isCandidateElement(node) && nodeNameEquals(node, PROPERTY_ELEMENT)) {
				parsePropertyElement((Element) node, bd);
			}
		}
	}

	/**
	 * Parse qualifier sub-elements of the given bean element.
	 */
	public void parseQualifierElements(Element beanEle, AbstractBeanDefinition bd) {
		NodeList nl = beanEle.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (isCandidateElement(node) && nodeNameEquals(node, QUALIFIER_ELEMENT)) {
				parseQualifierElement((Element) node, bd);
			}
		}
	}

	/**
	 * Parse lookup-override sub-elements of the given bean element.
	 */
	public void parseLookupOverrideSubElements(Element beanEle, MethodOverrides overrides) {
		NodeList nl = beanEle.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (isCandidateElement(node) && nodeNameEquals(node, LOOKUP_METHOD_ELEMENT)) {
				Element ele = (Element) node;
				String methodName = ele.getAttribute(NAME_ATTRIBUTE);
				String beanRef = ele.getAttribute(BEAN_ELEMENT);
				LookupOverride override = new LookupOverride(methodName, beanRef);
				override.setSource(extractSource(ele));
				overrides.addOverride(override);
			}
		}
	}

	/**
	 * Parse replaced-method sub-elements of the given bean element.
	 */
	public void parseReplacedMethodSubElements(Element beanEle, MethodOverrides overrides) {
		NodeList nl = beanEle.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (isCandidateElement(node) && nodeNameEquals(node, REPLACED_METHOD_ELEMENT)) {
				Element replacedMethodEle = (Element) node;
				String name = replacedMethodEle.getAttribute(NAME_ATTRIBUTE);
				String callback = replacedMethodEle.getAttribute(REPLACER_ATTRIBUTE);
				ReplaceOverride replaceOverride = new ReplaceOverride(name, callback);
				// Look for arg-type match elements.
				List<Element> argTypeEles = DomUtils.getChildElementsByTagName(replacedMethodEle, ARG_TYPE_ELEMENT);
				for (Element argTypeEle : argTypeEles) {
					String match = argTypeEle.getAttribute(ARG_TYPE_MATCH_ATTRIBUTE);
					match = (StringUtils.hasText(match) ? match : DomUtils.getTextValue(argTypeEle));
					if (StringUtils.hasText(match)) {
						replaceOverride.addTypeIdentifier(match);
					}
				}
				replaceOverride.setSource(extractSource(replacedMethodEle));
				overrides.addOverride(replaceOverride);
			}
		}
	}

	/**
	 * Parse a constructor-arg element.
	 */
	public void parseConstructorArgElement(Element ele, BeanDefinition bd) {
		String indexAttr = ele.getAttribute(INDEX_ATTRIBUTE);
		String typeAttr = ele.getAttribute(TYPE_ATTRIBUTE);
		String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);
		if (StringUtils.hasLength(indexAttr)) {
			try {
				int index = Integer.parseInt(indexAttr);
				if (index < 0) {
					error("'index' cannot be lower than 0", ele);
				}
				else {
					try {
						this.parseState.push(new ConstructorArgumentEntry(index));
						Object value = parsePropertyValue(ele, bd, null);
						ConstructorArgumentValues.ValueHolder valueHolder = new ConstructorArgumentValues.ValueHolder(value);
						if (StringUtils.hasLength(typeAttr)) {
							valueHolder.setType(typeAttr);
						}
						if (StringUtils.hasLength(nameAttr)) {
							valueHolder.setName(nameAttr);
						}
						valueHolder.setSource(extractSource(ele));
						if (bd.getConstructorArgumentValues().hasIndexedArgumentValue(index)) {
							error("Ambiguous constructor-arg entries for index " + index, ele);
						}
						else {
							bd.getConstructorArgumentValues().addIndexedArgumentValue(index, valueHolder);
						}
					}
					finally {
						this.parseState.pop();
					}
				}
			}
			catch (NumberFormatException ex) {
				error("Attribute 'index' of tag 'constructor-arg' must be an integer", ele);
			}
		}
		else {
			try {
				this.parseState.push(new ConstructorArgumentEntry());
				Object value = parsePropertyValue(ele, bd, null);
				ConstructorArgumentValues.ValueHolder valueHolder = new ConstructorArgumentValues.ValueHolder(value);
				if (StringUtils.hasLength(typeAttr)) {
					valueHolder.setType(typeAttr);
				}
				if (StringUtils.hasLength(nameAttr)) {
					valueHolder.setName(nameAttr);
				}
				valueHolder.setSource(extractSource(ele));
				bd.getConstructorArgumentValues().addGenericArgumentValue(valueHolder);
			}
			finally {
				this.parseState.pop();
			}
		}
	}

	/**
	 * Parse a property element.
	 */
	public void parsePropertyElement(Element ele, BeanDefinition bd) {
		String propertyName = ele.getAttribute(NAME_ATTRIBUTE);
		if (!StringUtils.hasLength(propertyName)) {
			error("Tag 'property' must have a 'name' attribute", ele);
			return;
		}
		this.parseState.push(new PropertyEntry(propertyName));
		try {
			if (bd.getPropertyValues().contains(propertyName)) {
				error("Multiple 'property' definitions for property '" + propertyName + "'", ele);
				return;
			}
			Object val = parsePropertyValue(ele, bd, propertyName);
			PropertyValue pv = new PropertyValue(propertyName, val);
			parseMetaElements(ele, pv);
			pv.setSource(extractSource(ele));
			bd.getPropertyValues().addPropertyValue(pv);
		}
		finally {
			this.parseState.pop();
		}
	}

	/**
	 * Parse a qualifier element.
	 */
	public void parseQualifierElement(Element ele, AbstractBeanDefinition bd) {
		String typeName = ele.getAttribute(TYPE_ATTRIBUTE);
		if (!StringUtils.hasLength(typeName)) {
			error("Tag 'qualifier' must have a 'type' attribute", ele);
			return;
		}
		this.parseState.push(new QualifierEntry(typeName));
		try {
			AutowireCandidateQualifier qualifier = new AutowireCandidateQualifier(typeName);
			qualifier.setSource(extractSource(ele));
			String value = ele.getAttribute(VALUE_ATTRIBUTE);
			if (StringUtils.hasLength(value)) {
				qualifier.setAttribute(AutowireCandidateQualifier.VALUE_KEY, value);
			}
			NodeList nl = ele.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (isCandidateElement(node) && nodeNameEquals(node, QUALIFIER_ATTRIBUTE_ELEMENT)) {
					Element attributeEle = (Element) node;
					String attributeName = attributeEle.getAttribute(KEY_ATTRIBUTE);
					String attributeValue = attributeEle.getAttribute(VALUE_ATTRIBUTE);
					if (StringUtils.hasLength(attributeName) && StringUtils.hasLength(attributeValue)) {
						BeanMetadataAttribute attribute = new BeanMetadataAttribute(attributeName, attributeValue);
						attribute.setSource(extractSource(attributeEle));
						qualifier.addMetadataAttribute(attribute);
					}
					else {
						error("Qualifier 'attribute' tag must have a 'name' and 'value'", attributeEle);
						return;
					}
				}
			}
			bd.addQualifier(qualifier);
		}
		finally {
			this.parseState.pop();
		}
	}

	/**
	 * Get the value of a property element. May be a list etc.
	 * Also used for constructor arguments, "propertyName" being null in this case.
	 */
	public Object parsePropertyValue(Element ele, BeanDefinition bd, String propertyName) {
		String elementName = (propertyName != null) ?
						"<property> element for property '" + propertyName + "'" :
						"<constructor-arg> element";

		// Should only have one child element: ref, value, list, etc.
		NodeList nl = ele.getChildNodes();
		Element subElement = null;
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (node instanceof Element && !nodeNameEquals(node, DESCRIPTION_ELEMENT) &&
					!nodeNameEquals(node, META_ELEMENT)) {
				// Child element is what we're looking for.
				if (subElement != null) {
					error(elementName + " must not contain more than one sub-element", ele);
				}
				else {
					subElement = (Element) node;
				}
			}
		}

		boolean hasRefAttribute = ele.hasAttribute(REF_ATTRIBUTE);
		boolean hasValueAttribute = ele.hasAttribute(VALUE_ATTRIBUTE);
		if ((hasRefAttribute && hasValueAttribute) ||
				((hasRefAttribute || hasValueAttribute) && subElement != null)) {
			error(elementName +
					" is only allowed to contain either 'ref' attribute OR 'value' attribute OR sub-element", ele);
		}

		if (hasRefAttribute) {
			String refName = ele.getAttribute(REF_ATTRIBUTE);
			if (!StringUtils.hasText(refName)) {
				error(elementName + " contains empty 'ref' attribute", ele);
			}
			RuntimeBeanReference ref = new RuntimeBeanReference(refName);
			ref.setSource(extractSource(ele));
			return ref;
		}
		else if (hasValueAttribute) {
			TypedStringValue valueHolder = new TypedStringValue(ele.getAttribute(VALUE_ATTRIBUTE));
			valueHolder.setSource(extractSource(ele));
			return valueHolder;
		}
		else if (subElement != null) {
			return parsePropertySubElement(subElement, bd);
		}
		else {
			// Neither child element nor "ref" or "value" attribute found.
			error(elementName + " must specify a ref or value", ele);
			return null;
		}
	}

	public Object parsePropertySubElement(Element ele, BeanDefinition bd) {
		return parsePropertySubElement(ele, bd, null);
	}

	/**
	 * Parse a value, ref or collection sub-element of a property or
	 * constructor-arg element.
	 * @param ele subelement of property element; we don't know which yet
	 * @param defaultValueType the default type (class name) for any
	 * {@code <value>} tag that might be created
	 */
	public Object parsePropertySubElement(Element ele, BeanDefinition bd, String defaultValueType) {
		if (!isDefaultNamespace(ele)) {
			return parseNestedCustomElement(ele, bd);
		}
		else if (nodeNameEquals(ele, BEAN_ELEMENT)) {
			BeanDefinitionHolder nestedBd = parseBeanDefinitionElement(ele, bd);
			if (nestedBd != null) {
				nestedBd = decorateBeanDefinitionIfRequired(ele, nestedBd, bd);
			}
			return nestedBd;
		}
		else if (nodeNameEquals(ele, REF_ELEMENT)) {
			// A generic reference to any name of any bean.
			String refName = ele.getAttribute(BEAN_REF_ATTRIBUTE);
			boolean toParent = false;
			if (!StringUtils.hasLength(refName)) {
				// A reference to the id of another bean in the same XML file.
				refName = ele.getAttribute(LOCAL_REF_ATTRIBUTE);
				if (!StringUtils.hasLength(refName)) {
					// A reference to the id of another bean in a parent context.
					refName = ele.getAttribute(PARENT_REF_ATTRIBUTE);
					toParent = true;
					if (!StringUtils.hasLength(refName)) {
						error("'bean', 'local' or 'parent' is required for <ref> element", ele);
						return null;
					}
				}
			}
			if (!StringUtils.hasText(refName)) {
				error("<ref> element contains empty target attribute", ele);
				return null;
			}
			RuntimeBeanReference ref = new RuntimeBeanReference(refName, toParent);
			ref.setSource(extractSource(ele));
			return ref;
		}
		else if (nodeNameEquals(ele, IDREF_ELEMENT)) {
			return parseIdRefElement(ele);
		}
		else if (nodeNameEquals(ele, VALUE_ELEMENT)) {
			return parseValueElement(ele, defaultValueType);
		}
		else if (nodeNameEquals(ele, NULL_ELEMENT)) {
			// It's a distinguished null value. Let's wrap it in a TypedStringValue
			// object in order to preserve the source location.
			TypedStringValue nullHolder = new TypedStringValue(null);
			nullHolder.setSource(extractSource(ele));
			return nullHolder;
		}
		else if (nodeNameEquals(ele, ARRAY_ELEMENT)) {
			return parseArrayElement(ele, bd);
		}
		else if (nodeNameEquals(ele, LIST_ELEMENT)) {
			return parseListElement(ele, bd);
		}
		else if (nodeNameEquals(ele, SET_ELEMENT)) {
			return parseSetElement(ele, bd);
		}
		else if (nodeNameEquals(ele, MAP_ELEMENT)) {
			return parseMapElement(ele, bd);
		}
		else if (nodeNameEquals(ele, PROPS_ELEMENT)) {
			return parsePropsElement(ele);
		}
		else {
			error("Unknown property sub-element: [" + ele.getNodeName() + "]", ele);
			return null;
		}
	}

	/**
	 * Return a typed String value Object for the given 'idref' element.
	 */
	public Object parseIdRefElement(Element ele) {
		// A generic reference to any name of any bean.
		String refName = ele.getAttribute(BEAN_REF_ATTRIBUTE);
		if (!StringUtils.hasLength(refName)) {
			// A reference to the id of another bean in the same XML file.
			refName = ele.getAttribute(LOCAL_REF_ATTRIBUTE);
			if (!StringUtils.hasLength(refName)) {
				error("Either 'bean' or 'local' is required for <idref> element", ele);
				return null;
			}
		}
		if (!StringUtils.hasText(refName)) {
			error("<idref> element contains empty target attribute", ele);
			return null;
		}
		RuntimeBeanNameReference ref = new RuntimeBeanNameReference(refName);
		ref.setSource(extractSource(ele));
		return ref;
	}

	/**
	 * Return a typed String value Object for the given value element.
	 */
	public Object parseValueElement(Element ele, String defaultTypeName) {
		// It's a literal value.
		String value = DomUtils.getTextValue(ele);
		String specifiedTypeName = ele.getAttribute(TYPE_ATTRIBUTE);
		String typeName = specifiedTypeName;
		if (!StringUtils.hasText(typeName)) {
			typeName = defaultTypeName;
		}
		try {
			TypedStringValue typedValue = buildTypedStringValue(value, typeName);
			typedValue.setSource(extractSource(ele));
			typedValue.setSpecifiedTypeName(specifiedTypeName);
			return typedValue;
		}
		catch (ClassNotFoundException ex) {
			error("Type class [" + typeName + "] not found for <value> element", ele, ex);
			return value;
		}
	}

	/**
	 * Build a typed String value Object for the given raw value.
	 * @see org.springframework.beans.factory.config.TypedStringValue
	 */
	protected TypedStringValue buildTypedStringValue(String value, String targetTypeName)
			throws ClassNotFoundException {

		ClassLoader classLoader = this.readerContext.getBeanClassLoader();
		TypedStringValue typedValue;
		if (!StringUtils.hasText(targetTypeName)) {
			typedValue = new TypedStringValue(value);
		}
		else if (classLoader != null) {
			Class<?> targetType = ClassUtils.forName(targetTypeName, classLoader);
			typedValue = new TypedStringValue(value, targetType);
		}
		else {
			typedValue = new TypedStringValue(value, targetTypeName);
		}
		return typedValue;
	}

	/**
	 * Parse an array element.
	 */
	public Object parseArrayElement(Element arrayEle, BeanDefinition bd) {
		String elementType = arrayEle.getAttribute(VALUE_TYPE_ATTRIBUTE);
		NodeList nl = arrayEle.getChildNodes();
		ManagedArray target = new ManagedArray(elementType, nl.getLength());
		target.setSource(extractSource(arrayEle));
		target.setElementTypeName(elementType);
		target.setMergeEnabled(parseMergeAttribute(arrayEle));
		parseCollectionElements(nl, target, bd, elementType);
		return target;
	}

	/**
	 * Parse a list element.
	 */
	public List<Object> parseListElement(Element collectionEle, BeanDefinition bd) {
		String defaultElementType = collectionEle.getAttribute(VALUE_TYPE_ATTRIBUTE);
		NodeList nl = collectionEle.getChildNodes();
		ManagedList<Object> target = new ManagedList<Object>(nl.getLength());
		target.setSource(extractSource(collectionEle));
		target.setElementTypeName(defaultElementType);
		target.setMergeEnabled(parseMergeAttribute(collectionEle));
		parseCollectionElements(nl, target, bd, defaultElementType);
		return target;
	}

	/**
	 * Parse a set element.
	 */
	public Set<Object> parseSetElement(Element collectionEle, BeanDefinition bd) {
		String defaultElementType = collectionEle.getAttribute(VALUE_TYPE_ATTRIBUTE);
		NodeList nl = collectionEle.getChildNodes();
		ManagedSet<Object> target = new ManagedSet<Object>(nl.getLength());
		target.setSource(extractSource(collectionEle));
		target.setElementTypeName(defaultElementType);
		target.setMergeEnabled(parseMergeAttribute(collectionEle));
		parseCollectionElements(nl, target, bd, defaultElementType);
		return target;
	}

	protected void parseCollectionElements(
			NodeList elementNodes, Collection<Object> target, BeanDefinition bd, String defaultElementType) {

		for (int i = 0; i < elementNodes.getLength(); i++) {
			Node node = elementNodes.item(i);
			if (node instanceof Element && !nodeNameEquals(node, DESCRIPTION_ELEMENT)) {
				target.add(parsePropertySubElement((Element) node, bd, defaultElementType));
			}
		}
	}

	/**
	 * Parse a map element.
	 */
	public Map<Object, Object> parseMapElement(Element mapEle, BeanDefinition bd) {
		String defaultKeyType = mapEle.getAttribute(KEY_TYPE_ATTRIBUTE);
		String defaultValueType = mapEle.getAttribute(VALUE_TYPE_ATTRIBUTE);

		List<Element> entryEles = DomUtils.getChildElementsByTagName(mapEle, ENTRY_ELEMENT);
		ManagedMap<Object, Object> map = new ManagedMap<Object, Object>(entryEles.size());
		map.setSource(extractSource(mapEle));
		map.setKeyTypeName(defaultKeyType);
		map.setValueTypeName(defaultValueType);
		map.setMergeEnabled(parseMergeAttribute(mapEle));

		for (Element entryEle : entryEles) {
			// Should only have one value child element: ref, value, list, etc.
			// Optionally, there might be a key child element.
			NodeList entrySubNodes = entryEle.getChildNodes();
			Element keyEle = null;
			Element valueEle = null;
			for (int j = 0; j < entrySubNodes.getLength(); j++) {
				Node node = entrySubNodes.item(j);
				if (node instanceof Element) {
					Element candidateEle = (Element) node;
					if (nodeNameEquals(candidateEle, KEY_ELEMENT)) {
						if (keyEle != null) {
							error("<entry> element is only allowed to contain one <key> sub-element", entryEle);
						}
						else {
							keyEle = candidateEle;
						}
					}
					else {
						// Child element is what we're looking for.
						if (nodeNameEquals(candidateEle, DESCRIPTION_ELEMENT)) {
							// the element is a <description> -> ignore it
						}
						else if (valueEle != null) {
							error("<entry> element must not contain more than one value sub-element", entryEle);
						}
						else {
							valueEle = candidateEle;
						}
					}
				}
			}

			// Extract key from attribute or sub-element.
			Object key = null;
			boolean hasKeyAttribute = entryEle.hasAttribute(KEY_ATTRIBUTE);
			boolean hasKeyRefAttribute = entryEle.hasAttribute(KEY_REF_ATTRIBUTE);
			if ((hasKeyAttribute && hasKeyRefAttribute) ||
					(hasKeyAttribute || hasKeyRefAttribute) && keyEle != null) {
				error("<entry> element is only allowed to contain either " +
						"a 'key' attribute OR a 'key-ref' attribute OR a <key> sub-element", entryEle);
			}
			if (hasKeyAttribute) {
				key = buildTypedStringValueForMap(entryEle.getAttribute(KEY_ATTRIBUTE), defaultKeyType, entryEle);
			}
			else if (hasKeyRefAttribute) {
				String refName = entryEle.getAttribute(KEY_REF_ATTRIBUTE);
				if (!StringUtils.hasText(refName)) {
					error("<entry> element contains empty 'key-ref' attribute", entryEle);
				}
				RuntimeBeanReference ref = new RuntimeBeanReference(refName);
				ref.setSource(extractSource(entryEle));
				key = ref;
			}
			else if (keyEle != null) {
				key = parseKeyElement(keyEle, bd, defaultKeyType);
			}
			else {
				error("<entry> element must specify a key", entryEle);
			}

			// Extract value from attribute or sub-element.
			Object value = null;
			boolean hasValueAttribute = entryEle.hasAttribute(VALUE_ATTRIBUTE);
			boolean hasValueRefAttribute = entryEle.hasAttribute(VALUE_REF_ATTRIBUTE);
			boolean hasValueTypeAttribute = entryEle.hasAttribute(VALUE_TYPE_ATTRIBUTE);
			if ((hasValueAttribute && hasValueRefAttribute) ||
					(hasValueAttribute || hasValueRefAttribute) && valueEle != null) {
				error("<entry> element is only allowed to contain either " +
						"'value' attribute OR 'value-ref' attribute OR <value> sub-element", entryEle);
			}
			if ((hasValueTypeAttribute && hasValueRefAttribute) ||
				(hasValueTypeAttribute && !hasValueAttribute) ||
					(hasValueTypeAttribute && valueEle != null)) {
				error("<entry> element is only allowed to contain a 'value-type' " +
						"attribute when it has a 'value' attribute", entryEle);
			}
			if (hasValueAttribute) {
				String valueType = entryEle.getAttribute(VALUE_TYPE_ATTRIBUTE);
				if (!StringUtils.hasText(valueType)) {
					valueType = defaultValueType;
				}
				value = buildTypedStringValueForMap(entryEle.getAttribute(VALUE_ATTRIBUTE), valueType, entryEle);
			}
			else if (hasValueRefAttribute) {
				String refName = entryEle.getAttribute(VALUE_REF_ATTRIBUTE);
				if (!StringUtils.hasText(refName)) {
					error("<entry> element contains empty 'value-ref' attribute", entryEle);
				}
				RuntimeBeanReference ref = new RuntimeBeanReference(refName);
				ref.setSource(extractSource(entryEle));
				value = ref;
			}
			else if (valueEle != null) {
				value = parsePropertySubElement(valueEle, bd, defaultValueType);
			}
			else {
				error("<entry> element must specify a value", entryEle);
			}

			// Add final key and value to the Map.
			map.put(key, value);
		}

		return map;
	}

	/**
	 * Build a typed String value Object for the given raw value.
	 * @see org.springframework.beans.factory.config.TypedStringValue
	 */
	protected final Object buildTypedStringValueForMap(String value, String defaultTypeName, Element entryEle) {
		try {
			TypedStringValue typedValue = buildTypedStringValue(value, defaultTypeName);
			typedValue.setSource(extractSource(entryEle));
			return typedValue;
		}
		catch (ClassNotFoundException ex) {
			error("Type class [" + defaultTypeName + "] not found for Map key/value type", entryEle, ex);
			return value;
		}
	}

	/**
	 * Parse a key sub-element of a map element.
	 */
	protected Object parseKeyElement(Element keyEle, BeanDefinition bd, String defaultKeyTypeName) {
		NodeList nl = keyEle.getChildNodes();
		Element subElement = null;
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (node instanceof Element) {
				// Child element is what we're looking for.
				if (subElement != null) {
					error("<key> element must not contain more than one value sub-element", keyEle);
				}
				else {
					subElement = (Element) node;
				}
			}
		}
		return parsePropertySubElement(subElement, bd, defaultKeyTypeName);
	}

	/**
	 * Parse a props element.
	 */
	public Properties parsePropsElement(Element propsEle) {
		ManagedProperties props = new ManagedProperties();
		props.setSource(extractSource(propsEle));
		props.setMergeEnabled(parseMergeAttribute(propsEle));

		List<Element> propEles = DomUtils.getChildElementsByTagName(propsEle, PROP_ELEMENT);
		for (Element propEle : propEles) {
			String key = propEle.getAttribute(KEY_ATTRIBUTE);
			// Trim the text value to avoid unwanted whitespace
			// caused by typical XML formatting.
			String value = DomUtils.getTextValue(propEle).trim();
			TypedStringValue keyHolder = new TypedStringValue(key);
			keyHolder.setSource(extractSource(propEle));
			TypedStringValue valueHolder = new TypedStringValue(value);
			valueHolder.setSource(extractSource(propEle));
			props.put(keyHolder, valueHolder);
		}

		return props;
	}

	/**
	 * Parse the merge attribute of a collection element, if any.
	 */
	public boolean parseMergeAttribute(Element collectionElement) {
		String value = collectionElement.getAttribute(MERGE_ATTRIBUTE);
		if (DEFAULT_VALUE.equals(value)) {
			value = this.defaults.getMerge();
		}
		return TRUE_VALUE.equals(value);
	}

	public BeanDefinition parseCustomElement(Element ele) {
		return parseCustomElement(ele, null);
	}

	public BeanDefinition parseCustomElement(Element ele, BeanDefinition containingBd) {
		String namespaceUri = getNamespaceURI(ele);
		NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
		if (handler == null) {
			error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
			return null;
		}
		return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
	}

	public BeanDefinitionHolder decorateBeanDefinitionIfRequired(Element ele, BeanDefinitionHolder definitionHolder) {
		return decorateBeanDefinitionIfRequired(ele, definitionHolder, null);
	}

	public BeanDefinitionHolder decorateBeanDefinitionIfRequired(
			Element ele, BeanDefinitionHolder definitionHolder, BeanDefinition containingBd) {

		BeanDefinitionHolder finalDefinition = definitionHolder;

		// Decorate based on custom attributes first.
		NamedNodeMap attributes = ele.getAttributes();
		for (int i = 0; i < attributes.getLength(); i++) {
			Node node = attributes.item(i);
			finalDefinition = decorateIfRequired(node, finalDefinition, containingBd);
		}

		// Decorate based on custom nested elements.
		NodeList children = ele.getChildNodes();
		for (int i = 0; i < children.getLength(); i++) {
			Node node = children.item(i);
			if (node.getNodeType() == Node.ELEMENT_NODE) {
				finalDefinition = decorateIfRequired(node, finalDefinition, containingBd);
			}
		}
		return finalDefinition;
	}

	public BeanDefinitionHolder decorateIfRequired(
			Node node, BeanDefinitionHolder originalDef, BeanDefinition containingBd) {

		String namespaceUri = getNamespaceURI(node);
		if (!isDefaultNamespace(namespaceUri)) {
			NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
			if (handler != null) {
				return handler.decorate(node, originalDef, new ParserContext(this.readerContext, this, containingBd));
			}
			else if (namespaceUri != null && namespaceUri.startsWith("http://www.springframework.org/")) {
				error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", node);
			}
			else {
				// A custom namespace, not to be handled by Spring - maybe "xml:...".
				if (logger.isDebugEnabled()) {
					logger.debug("No Spring NamespaceHandler found for XML schema namespace [" + namespaceUri + "]");
				}
			}
		}
		return originalDef;
	}

	private BeanDefinitionHolder parseNestedCustomElement(Element ele, BeanDefinition containingBd) {
		BeanDefinition innerDefinition = parseCustomElement(ele, containingBd);
		if (innerDefinition == null) {
			error("Incorrect usage of element '" + ele.getNodeName() + "' in a nested manner. " +
					"This tag cannot be used nested inside <property>.", ele);
			return null;
		}
		String id = ele.getNodeName() + BeanDefinitionReaderUtils.GENERATED_BEAN_NAME_SEPARATOR +
				ObjectUtils.getIdentityHexString(innerDefinition);
		if (logger.isDebugEnabled()) {
			logger.debug("Using generated bean name [" + id +
					"] for nested custom element '" + ele.getNodeName() + "'");
		}
		return new BeanDefinitionHolder(innerDefinition, id);
	}

	public String getNamespaceURI(Node node) {
		return node.getNamespaceURI();
	}

	public String getLocalName(Node node) {
		return node.getLocalName();
	}

	public boolean nodeNameEquals(Node node, String desiredName) {
		return desiredName.equals(node.getNodeName()) || desiredName.equals(getLocalName(node));
	}

	public boolean isDefaultNamespace(String namespaceUri) {
		return (!StringUtils.hasLength(namespaceUri) || BEANS_NAMESPACE_URI.equals(namespaceUri));
	}

	public boolean isDefaultNamespace(Node node) {
		return isDefaultNamespace(getNamespaceURI(node));
	}

	private boolean isCandidateElement(Node node) {
		return (node instanceof Element && (isDefaultNamespace(node) || !isDefaultNamespace(node.getParentNode())));
	}

}

```

## ParserContext.java

```java
package org.springframework.beans.factory.xml;

import java.util.Stack;

import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.parsing.BeanComponentDefinition;
import org.springframework.beans.factory.parsing.ComponentDefinition;
import org.springframework.beans.factory.parsing.CompositeComponentDefinition;
import org.springframework.beans.factory.support.BeanDefinitionReaderUtils;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;

/**
 * 解析器上下文。用于节约多个bean注册时通知问题。
 */
public final class ParserContext {

	private final XmlReaderContext readerContext;

	private final BeanDefinitionParserDelegate delegate;

	private BeanDefinition containingBeanDefinition;

	private final Stack<CompositeComponentDefinition> containingComponents = new Stack<CompositeComponentDefinition>();


	public ParserContext(XmlReaderContext readerContext, BeanDefinitionParserDelegate delegate) {
		this.readerContext = readerContext;
		this.delegate = delegate;
	}

	public ParserContext(XmlReaderContext readerContext, BeanDefinitionParserDelegate delegate,
			BeanDefinition containingBeanDefinition) {

		this.readerContext = readerContext;
		this.delegate = delegate;
		this.containingBeanDefinition = containingBeanDefinition;
	}


	public final XmlReaderContext getReaderContext() {
		return this.readerContext;
	}

	public final BeanDefinitionRegistry getRegistry() {
		return this.readerContext.getRegistry();
	}

	public final BeanDefinitionParserDelegate getDelegate() {
		return this.delegate;
	}

	public final BeanDefinition getContainingBeanDefinition() {
		return this.containingBeanDefinition;
	}

	public final boolean isNested() {
		return (this.containingBeanDefinition != null);
	}

	public boolean isDefaultLazyInit() {
		return BeanDefinitionParserDelegate.TRUE_VALUE.equals(this.delegate.getDefaults().getLazyInit());
	}

	public Object extractSource(Object sourceCandidate) {
		return this.readerContext.extractSource(sourceCandidate);
	}

	public CompositeComponentDefinition getContainingComponent() {
		return (!this.containingComponents.isEmpty() ? this.containingComponents.lastElement() : null);
	}

	public void pushContainingComponent(CompositeComponentDefinition containingComponent) {
		this.containingComponents.push(containingComponent);
	}

	public CompositeComponentDefinition popContainingComponent() {
		return this.containingComponents.pop();
	}

	public void popAndRegisterContainingComponent() {
		registerComponent(popContainingComponent());
	}

	public void registerComponent(ComponentDefinition component) {
		CompositeComponentDefinition containingComponent = getContainingComponent();
		if (containingComponent != null) {
			containingComponent.addNestedComponent(component);
		}
		else {
			this.readerContext.fireComponentRegistered(component);
		}
	}

	public void registerBeanComponent(BeanComponentDefinition component) {
		BeanDefinitionReaderUtils.registerBeanDefinition(component, getRegistry());
		registerComponent(component);
	}

}
```

## BeanDefinitionDocumentReader.java

```java
package org.springframework.beans.factory.xml;

import org.w3c.dom.Document;

import org.springframework.beans.factory.BeanDefinitionStoreException;

/**
 * 解析Document
 */
public interface BeanDefinitionDocumentReader {

	void registerBeanDefinitions(Document doc, XmlReaderContext readerContext)
			throws BeanDefinitionStoreException;

}
```

### DefaultBeanDefinitionDocumentReader.java

```java
package org.springframework.beans.factory.xml;


/**
 * 默认的BeanDefinitionDocument读取器
 */
public class DefaultBeanDefinitionDocumentReader implements BeanDefinitionDocumentReader {

	public static final String BEAN_ELEMENT = BeanDefinitionParserDelegate.BEAN_ELEMENT;

	public static final String NESTED_BEANS_ELEMENT = "beans";

	public static final String ALIAS_ELEMENT = "alias";

	public static final String NAME_ATTRIBUTE = "name";

	public static final String ALIAS_ATTRIBUTE = "alias";

	public static final String IMPORT_ELEMENT = "import";

	public static final String RESOURCE_ATTRIBUTE = "resource";

	public static final String PROFILE_ATTRIBUTE = "profile";


	protected final Log logger = LogFactory.getLog(getClass());

	private XmlReaderContext readerContext;

	private BeanDefinitionParserDelegate delegate;

	@Override
	public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		this.readerContext = readerContext;
		logger.debug("Loading bean definitions");
		Element root = doc.getDocumentElement();
		doRegisterBeanDefinitions(root);
	}

	protected final XmlReaderContext getReaderContext() {
		return this.readerContext;
	}

	protected Object extractSource(Element ele) {
		return getReaderContext().extractSource(ele);
	}

	protected void doRegisterBeanDefinitions(Element root) {
		BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createDelegate(getReaderContext(), root, parent);

		if (this.delegate.isDefaultNamespace(root)) {
			String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
			if (StringUtils.hasText(profileSpec)) {
				String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
						profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
				if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
					if (logger.isInfoEnabled()) {
						logger.info("Skipped XML bean definition file due to specified profiles [" + profileSpec +
								"] not matching: " + getReaderContext().getResource());
					}
					return;
				}
			}
		}

		preProcessXml(root);
		parseBeanDefinitions(root, this.delegate);
		postProcessXml(root);

		this.delegate = parent;
	}

	protected BeanDefinitionParserDelegate createDelegate(
			XmlReaderContext readerContext, Element root, BeanDefinitionParserDelegate parentDelegate) {

		BeanDefinitionParserDelegate delegate = new BeanDefinitionParserDelegate(readerContext);
		delegate.initDefaults(root, parentDelegate);
		return delegate;
	}

	protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					if (delegate.isDefaultNamespace(ele)) {
						parseDefaultElement(ele, delegate);
					}
					else {
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			delegate.parseCustomElement(root);
		}
	}

	private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			importBeanDefinitionResource(ele);
		}
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			processAliasRegistration(ele);
		}
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// recurse
			doRegisterBeanDefinitions(ele);
		}
	}

	protected void importBeanDefinitionResource(Element ele) {
		String location = ele.getAttribute(RESOURCE_ATTRIBUTE);
		if (!StringUtils.hasText(location)) {
			getReaderContext().error("Resource location must not be empty", ele);
			return;
		}

		// Resolve system properties: e.g. "${user.dir}"
		location = getReaderContext().getEnvironment().resolveRequiredPlaceholders(location);

		Set<Resource> actualResources = new LinkedHashSet<Resource>(4);

		// Discover whether the location is an absolute or relative URI
		boolean absoluteLocation = false;
		try {
			absoluteLocation = ResourcePatternUtils.isUrl(location) || ResourceUtils.toURI(location).isAbsolute();
		}
		catch (URISyntaxException ex) {
			// cannot convert to an URI, considering the location relative
			// unless it is the well-known Spring prefix "classpath*:"
		}

		// Absolute or relative?
		if (absoluteLocation) {
			try {
				int importCount = getReaderContext().getReader().loadBeanDefinitions(location, actualResources);
				if (logger.isDebugEnabled()) {
					logger.debug("Imported " + importCount + " bean definitions from URL location [" + location + "]");
				}
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error(
						"Failed to import bean definitions from URL location [" + location + "]", ele, ex);
			}
		}
		else {
			// No URL -> considering resource location as relative to the current file.
			try {
				int importCount;
				Resource relativeResource = getReaderContext().getResource().createRelative(location);
				if (relativeResource.exists()) {
					importCount = getReaderContext().getReader().loadBeanDefinitions(relativeResource);
					actualResources.add(relativeResource);
				}
				else {
					String baseLocation = getReaderContext().getResource().getURL().toString();
					importCount = getReaderContext().getReader().loadBeanDefinitions(
							StringUtils.applyRelativePath(baseLocation, location), actualResources);
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Imported " + importCount + " bean definitions from relative location [" + location + "]");
				}
			}
			catch (IOException ex) {
				getReaderContext().error("Failed to resolve current resource location", ele, ex);
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error("Failed to import bean definitions from relative location [" + location + "]",
						ele, ex);
			}
		}
		Resource[] actResArray = actualResources.toArray(new Resource[actualResources.size()]);
		getReaderContext().fireImportProcessed(location, actResArray, extractSource(ele));
	}

	protected void processAliasRegistration(Element ele) {
		String name = ele.getAttribute(NAME_ATTRIBUTE);
		String alias = ele.getAttribute(ALIAS_ATTRIBUTE);
		boolean valid = true;
		if (!StringUtils.hasText(name)) {
			getReaderContext().error("Name must not be empty", ele);
			valid = false;
		}
		if (!StringUtils.hasText(alias)) {
			getReaderContext().error("Alias must not be empty", ele);
			valid = false;
		}
		if (valid) {
			try {
				getReaderContext().getRegistry().registerAlias(name, alias);
			}
			catch (Exception ex) {
				getReaderContext().error("Failed to register alias '" + alias +
						"' for bean with name '" + name + "'", ele, ex);
			}
			getReaderContext().fireAliasRegistered(name, alias, extractSource(ele));
		}
	}

	protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
		if (bdHolder != null) {
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
			try {
				// Register the final decorated instance.
				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error("Failed to register bean definition with name '" +
						bdHolder.getBeanName() + "'", ele, ex);
			}
			// Send registration event.
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
		}
	}

	protected void preProcessXml(Element root) {
	}

	protected void postProcessXml(Element root) {
	}

}
```

# Demo

```java
main:BeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("applicationContext.xml"), null);
XmlBeanFactory:super(parentBeanFactory);
AbstractAutowireCapableBeanFactory:	ignoreDependencyInterface(BeanNameAware.class);
									ignoreDependencyInterface(BeanFactoryAware.class);
									ignoreDependencyInterface(BeanClassLoaderAware.class);
XmlBeanFactory:this.reader.loadBeanDefinitions(resource);
XmlBeanDefinitionReader:loadBeanDefinitions(new EncodedResource(resource));
XmlBeanDefinitionReader:doLoadBeanDefinitions(inputSource, encodedResource.getResource());
XmlBeanDefinitionReader:doLoadDocument(inputSource, resource);
DefaultDocumentLoader:this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,getValidationModeForResource(resource), isNamespaceAware());
XmlBeanDefinitionReader:registerBeanDefinitions(doc, resource);
XmlBeanDefinitionReader:documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
DefaultBeanDefinitionDocumentReader:doRegisterBeanDefinitions(root);
DefaultBeanDefinitionDocumentReader:this.delegate = createDelegate(getReaderContext(), root, parent);
DefaultBeanDefinitionDocumentReader:parseBeanDefinitions(root, this.delegate);
DefaultBeanDefinitionDocumentReader:parseDefaultElement(ele, delegate);
BeanDefinitionParserDelegate:delegate.parseBeanDefinitionElement(ele);
BeanDefinitionParserDelegate:delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
BeanDefinitionParserDelegate:getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
```

## 简图

```java
@startuml
XmlBeanFactory -> XmlBeanDefinitionReader: 传入自身初始化读取器
XmlBeanDefinitionReader -> DefaultDocumentLoader : 使用Document加载器解析XML为Document
XmlBeanDefinitionReader -> DefaultBeanDefinitionDocumentReader : 解析Document的Element为BeanDefinition
DefaultBeanDefinitionDocumentReader -> BeanDefinitionParserDelegate: 委托给它解析Element
DefaultBeanDefinitionDocumentReader -> BeanDefinitionParserDelegate: 委托给它解析自定义标签和属性
XmlBeanDefinitionReader -> BeanDefinitionReaderUtils: 注册BeanDefinition到BeanDefinitionRegistry
@enduml
```

# BeanPostProcesser

## BeanPostProcessor.java

```java
package org.springframework.beans.factory.config;

import org.springframework.beans.BeansException;

/**
 * 后置处理器
 */
public interface BeanPostProcessor {
	//初始化前的处理
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
	//初始化后的处理
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}

```

### MergedBeanDefinitionPostProcessor.java

```java
package org.springframework.beans.factory.support;

import org.springframework.beans.factory.config.BeanPostProcessor;

/**
 * 已合并的BeanDefinition后处理器
 */
public interface MergedBeanDefinitionPostProcessor extends BeanPostProcessor {

	//对合并后的BeanDefinition进行处理
	void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName);

}

```

### DestructionAwareBeanPostProcessor.java

```java
package org.springframework.beans.factory.config;

import org.springframework.beans.BeansException;

/**
 * 销毁后处理器
 */
public interface DestructionAwareBeanPostProcessor extends BeanPostProcessor {
	//销毁bean之前处理
	void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException;
	//bean是否需要被当前处理器处理
	boolean requiresDestruction(Object bean);

}

```

### InstantiationAwareBeanPostProcessor.java

```java
package org.springframework.beans.factory.config;

import java.beans.PropertyDescriptor;

import org.springframework.beans.BeansException;
import org.springframework.beans.PropertyValues;

/**
 * 实例化后处理器。
 */
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {
	//实例化之前进行处理
	Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException;
	//实例化后进行处理
	boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException;
	//属性被引用前进行处理
	PropertyValues postProcessPropertyValues(
			PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException;

}

```

#### SmartInstantiationAwareBeanPostProcessor.java

```java
package org.springframework.beans.factory.config;

import java.lang.reflect.Constructor;

import org.springframework.beans.BeansException;

/**
 * 只能的初始化后处理器。
 */
public interface SmartInstantiationAwareBeanPostProcessor extends InstantiationAwareBeanPostProcessor {

	//预言实际类型
	Class<?> predictBeanType(Class<?> beanClass, String beanName) throws BeansException;

	//决定待用的构造函数
	Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName) throws BeansException;
	//获取早期引用
	Object getEarlyBeanReference(Object bean, String beanName) throws BeansException;

}

```



# end
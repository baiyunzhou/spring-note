# DataBinder

## DataBinder.java

```java
package org.springframework.validation;


/**
 * 数据绑定器。用于绑定，转换，验证数据。
 */
public class DataBinder implements PropertyEditorRegistry, TypeConverter {

	public static final String DEFAULT_OBJECT_NAME = "target";

	public static final int DEFAULT_AUTO_GROW_COLLECTION_LIMIT = 256;

	private final Object target;

	private final String objectName;

	private AbstractPropertyBindingResult bindingResult;

	private SimpleTypeConverter typeConverter;

	private boolean ignoreUnknownFields = true;

	private boolean ignoreInvalidFields = false;

	private boolean autoGrowNestedPaths = true;

	private int autoGrowCollectionLimit = DEFAULT_AUTO_GROW_COLLECTION_LIMIT;

	private String[] allowedFields;

	private String[] disallowedFields;

	private String[] requiredFields;

	private ConversionService conversionService;

	private MessageCodesResolver messageCodesResolver;

	private BindingErrorProcessor bindingErrorProcessor = new DefaultBindingErrorProcessor();

	private final List<Validator> validators = new ArrayList<Validator>();

	public DataBinder(Object target, String objectName) {
		if (target != null && target.getClass() == javaUtilOptionalClass) {
			this.target = OptionalUnwrapper.unwrap(target);
		}
		else {
			this.target = target;
		}
		this.objectName = objectName;
	}

	public void bind(PropertyValues pvs) {
		MutablePropertyValues mpvs = (pvs instanceof MutablePropertyValues) ?
				(MutablePropertyValues) pvs : new MutablePropertyValues(pvs);
		doBind(mpvs);
	}

	protected void doBind(MutablePropertyValues mpvs) {
		checkAllowedFields(mpvs);
		checkRequiredFields(mpvs);
		applyPropertyValues(mpvs);
	}

	public void validate() {
		for (Validator validator : this.validators) {
			validator.validate(getTarget(), getBindingResult());
		}
	}

	public void validate(Object... validationHints) {
		for (Validator validator : getValidators()) {
			if (!ObjectUtils.isEmpty(validationHints) && validator instanceof SmartValidator) {
				((SmartValidator) validator).validate(getTarget(), getBindingResult(), validationHints);
			}
			else if (validator != null) {
				validator.validate(getTarget(), getBindingResult());
			}
		}
	}
}

```

### WebDataBinder.java

```java
package org.springframework.web.bind;

import java.lang.reflect.Array;
import java.util.Collection;
import java.util.List;
import java.util.Map;

import org.springframework.beans.MutablePropertyValues;
import org.springframework.beans.PropertyValue;
import org.springframework.core.CollectionFactory;
import org.springframework.validation.DataBinder;
import org.springframework.web.multipart.MultipartFile;

/**
 * 用于将数据从web请求参数绑定到JavaBean对象的特殊DataBinder。设计用于web环境，但不依赖于Servlet API;用作更特定的DataBinder变体的基类，例如		 
 * org.springframework.web.bind.ServletRequestDataBinder。
 * 不是很懂!前缀
 */
public class WebDataBinder extends DataBinder {

	public static final String DEFAULT_FIELD_MARKER_PREFIX = "_";

	public static final String DEFAULT_FIELD_DEFAULT_PREFIX = "!";

	private String fieldMarkerPrefix = DEFAULT_FIELD_MARKER_PREFIX;

	private String fieldDefaultPrefix = DEFAULT_FIELD_DEFAULT_PREFIX;

	private boolean bindEmptyMultipartFiles = true;

	public WebDataBinder(Object target) {
		super(target);
	}

	public WebDataBinder(Object target, String objectName) {
		super(target, objectName);
	}

	public void setFieldMarkerPrefix(String fieldMarkerPrefix) {
		this.fieldMarkerPrefix = fieldMarkerPrefix;
	}

	public String getFieldMarkerPrefix() {
		return this.fieldMarkerPrefix;
	}

	public void setFieldDefaultPrefix(String fieldDefaultPrefix) {
		this.fieldDefaultPrefix = fieldDefaultPrefix;
	}

	public String getFieldDefaultPrefix() {
		return this.fieldDefaultPrefix;
	}

	public void setBindEmptyMultipartFiles(boolean bindEmptyMultipartFiles) {
		this.bindEmptyMultipartFiles = bindEmptyMultipartFiles;
	}

	public boolean isBindEmptyMultipartFiles() {
		return this.bindEmptyMultipartFiles;
	}

	@Override
	protected void doBind(MutablePropertyValues mpvs) {
		checkFieldDefaults(mpvs);
		checkFieldMarkers(mpvs);
		super.doBind(mpvs);
	}

	protected void checkFieldDefaults(MutablePropertyValues mpvs) {
		String fieldDefaultPrefix = getFieldDefaultPrefix();
		if (fieldDefaultPrefix != null) {
			PropertyValue[] pvArray = mpvs.getPropertyValues();
			for (PropertyValue pv : pvArray) {
				if (pv.getName().startsWith(fieldDefaultPrefix)) {
					String field = pv.getName().substring(fieldDefaultPrefix.length());
					if (getPropertyAccessor().isWritableProperty(field) && !mpvs.contains(field)) {
						mpvs.add(field, pv.getValue());
					}
					mpvs.removePropertyValue(pv);
				}
			}
		}
	}

	protected void checkFieldMarkers(MutablePropertyValues mpvs) {
		String fieldMarkerPrefix = getFieldMarkerPrefix();
		if (fieldMarkerPrefix != null) {
			PropertyValue[] pvArray = mpvs.getPropertyValues();
			for (PropertyValue pv : pvArray) {
				if (pv.getName().startsWith(fieldMarkerPrefix)) {
					String field = pv.getName().substring(fieldMarkerPrefix.length());
					if (getPropertyAccessor().isWritableProperty(field) && !mpvs.contains(field)) {
						Class<?> fieldType = getPropertyAccessor().getPropertyType(field);
						mpvs.add(field, getEmptyValue(field, fieldType));
					}
					mpvs.removePropertyValue(pv);
				}
			}
		}
	}

	protected Object getEmptyValue(String field, Class<?> fieldType) {
		if (fieldType != null) {
			try {
				if (boolean.class == fieldType || Boolean.class == fieldType) {
					// Special handling of boolean property.
					return Boolean.FALSE;
				}
				else if (fieldType.isArray()) {
					// Special handling of array property.
					return Array.newInstance(fieldType.getComponentType(), 0);
				}
				else if (Collection.class.isAssignableFrom(fieldType)) {
					return CollectionFactory.createCollection(fieldType, 0);
				}
				else if (Map.class.isAssignableFrom(fieldType)) {
					return CollectionFactory.createMap(fieldType, 0);
				}
			}
			catch (IllegalArgumentException ex) {
				if (logger.isDebugEnabled()) {
					logger.debug("Failed to create default value - falling back to null: " + ex.getMessage());
				}
			}
		}
		// Default value: null.
		return null;
	}

	protected void bindMultipart(Map<String, List<MultipartFile>> multipartFiles, MutablePropertyValues mpvs) {
		for (Map.Entry<String, List<MultipartFile>> entry : multipartFiles.entrySet()) {
			String key = entry.getKey();
			List<MultipartFile> values = entry.getValue();
			if (values.size() == 1) {
				MultipartFile value = values.get(0);
				if (isBindEmptyMultipartFiles() || !value.isEmpty()) {
					mpvs.add(key, value);
				}
			}
			else {
				mpvs.add(key, values);
			}
		}
	}

}

```

#### ServletRequestDataBinder.java

```java
package org.springframework.web.bind;

import javax.servlet.ServletRequest;

import org.springframework.beans.MutablePropertyValues;
import org.springframework.validation.BindException;
import org.springframework.web.multipart.MultipartRequest;
import org.springframework.web.util.WebUtils;

/**
 * Servlet请求的数据绑定器
 */
public class ServletRequestDataBinder extends WebDataBinder {

	public ServletRequestDataBinder(Object target) {
		super(target);
	}
ame the name of the target object
	 */
	public ServletRequestDataBinder(Object target, String objectName) {
		super(target, objectName);
	}

	public void bind(ServletRequest request) {
		MutablePropertyValues mpvs = new ServletRequestParameterPropertyValues(request);
		MultipartRequest multipartRequest = WebUtils.getNativeRequest(request, MultipartRequest.class);
		if (multipartRequest != null) {
			bindMultipart(multipartRequest.getMultiFileMap(), mpvs);
		}
		addBindValues(mpvs, request);
		doBind(mpvs);
	}

	protected void addBindValues(MutablePropertyValues mpvs, ServletRequest request) {
	}

	public void closeNoCatch() throws ServletRequestBindingException {
		if (getBindingResult().hasErrors()) {
			throw new ServletRequestBindingException(
					"Errors binding onto object '" + getBindingResult().getObjectName() + "'",
					new BindException(getBindingResult()));
		}
	}

}

```

##### ExtendedServletRequestDataBinder.java

```java
package org.springframework.web.servlet.mvc.method.annotation;

import java.util.Map;
import java.util.Map.Entry;
import javax.servlet.ServletRequest;

import org.springframework.beans.MutablePropertyValues;
import org.springframework.web.bind.ServletRequestDataBinder;
import org.springframework.web.servlet.HandlerMapping;

public class ExtendedServletRequestDataBinder extends ServletRequestDataBinder {

	public ExtendedServletRequestDataBinder(Object target) {
		super(target);
	}

	public ExtendedServletRequestDataBinder(Object target, String objectName) {
		super(target, objectName);
	}

	@Override
	@SuppressWarnings("unchecked")
	protected void addBindValues(MutablePropertyValues mpvs, ServletRequest request) {
		String attr = HandlerMapping.URI_TEMPLATE_VARIABLES_ATTRIBUTE;
		Map<String, String> uriVars = (Map<String, String>) request.getAttribute(attr);
		if (uriVars != null) {
			for (Entry<String, String> entry : uriVars.entrySet()) {
				if (mpvs.contains(entry.getKey())) {
					if (logger.isWarnEnabled()) {
						logger.warn("Skipping URI variable '" + entry.getKey() +
								"' since the request contains a bind value with the same name.");
					}
				}
				else {
					mpvs.addPropertyValue(entry.getKey(), entry.getValue());
				}
			}
		}
	}

}

```

#### WebRequestDataBinder.java

```java
package org.springframework.web.bind.support;

import java.util.List;
import java.util.Map;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.Part;

import org.springframework.beans.MutablePropertyValues;
import org.springframework.util.ClassUtils;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.util.StringUtils;
import org.springframework.validation.BindException;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.multipart.MultipartException;
import org.springframework.web.multipart.MultipartRequest;

public class WebRequestDataBinder extends WebDataBinder {

	private static final boolean servlet3Parts = ClassUtils.hasMethod(HttpServletRequest.class, "getParts");

	public WebRequestDataBinder(Object target) {
		super(target);
	}

	public WebRequestDataBinder(Object target, String objectName) {
		super(target, objectName);
	}

	public void bind(WebRequest request) {
		MutablePropertyValues mpvs = new MutablePropertyValues(request.getParameterMap());
		if (isMultipartRequest(request) && request instanceof NativeWebRequest) {
			MultipartRequest multipartRequest = ((NativeWebRequest) request).getNativeRequest(MultipartRequest.class);
			if (multipartRequest != null) {
				bindMultipart(multipartRequest.getMultiFileMap(), mpvs);
			}
			else if (servlet3Parts) {
				HttpServletRequest serlvetRequest = ((NativeWebRequest) request).getNativeRequest(HttpServletRequest.class);
				new Servlet3MultipartHelper(isBindEmptyMultipartFiles()).bindParts(serlvetRequest, mpvs);
			}
		}
		doBind(mpvs);
	}

	private boolean isMultipartRequest(WebRequest request) {
		String contentType = request.getHeader("Content-Type");
		return (contentType != null && StringUtils.startsWithIgnoreCase(contentType, "multipart"));
	}

	public void closeNoCatch() throws BindException {
		if (getBindingResult().hasErrors()) {
			throw new BindException(getBindingResult());
		}
	}

	private static class Servlet3MultipartHelper {

		private final boolean bindEmptyMultipartFiles;

		public Servlet3MultipartHelper(boolean bindEmptyMultipartFiles) {
			this.bindEmptyMultipartFiles = bindEmptyMultipartFiles;
		}

		public void bindParts(HttpServletRequest request, MutablePropertyValues mpvs) {
			try {
				MultiValueMap<String, Part> map = new LinkedMultiValueMap<String, Part>();
				for (Part part : request.getParts()) {
					map.add(part.getName(), part);
				}
				for (Map.Entry<String, List<Part>> entry: map.entrySet()) {
					if (entry.getValue().size() == 1) {
						Part part = entry.getValue().get(0);
						if (this.bindEmptyMultipartFiles || part.getSize() > 0) {
							mpvs.add(entry.getKey(), part);
						}
					}
					else {
						mpvs.add(entry.getKey(), entry.getValue());
					}
				}
			}
			catch (Exception ex) {
				throw new MultipartException("Failed to get request parts", ex);
			}
		}
	}

}

```

## WebBindingInitializer.java

```java
package org.springframework.web.bind.support;

import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.context.request.WebRequest;

/**
 * 数据绑定初始化器
 */
public interface WebBindingInitializer {

	void initBinder(WebDataBinder binder, WebRequest request);

}

```

### ConfigurableWebBindingInitializer.java

```java
package org.springframework.web.bind.support;

import org.springframework.beans.PropertyEditorRegistrar;
import org.springframework.core.convert.ConversionService;
import org.springframework.validation.BindingErrorProcessor;
import org.springframework.validation.MessageCodesResolver;
import org.springframework.validation.Validator;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.context.request.WebRequest;

/**
 * 可配置的web数据绑定初始化器
 */
public class ConfigurableWebBindingInitializer implements WebBindingInitializer {

	private boolean autoGrowNestedPaths = true;

	private boolean directFieldAccess = false;

	private MessageCodesResolver messageCodesResolver;

	private BindingErrorProcessor bindingErrorProcessor;

	private Validator validator;

	private ConversionService conversionService;

	private PropertyEditorRegistrar[] propertyEditorRegistrars;

	@Override
	public void initBinder(WebDataBinder binder, WebRequest request) {
		binder.setAutoGrowNestedPaths(this.autoGrowNestedPaths);
		if (this.directFieldAccess) {
			binder.initDirectFieldAccess();
		}
		if (this.messageCodesResolver != null) {
			binder.setMessageCodesResolver(this.messageCodesResolver);
		}
		if (this.bindingErrorProcessor != null) {
			binder.setBindingErrorProcessor(this.bindingErrorProcessor);
		}
		if (this.validator != null && binder.getTarget() != null &&
				this.validator.supports(binder.getTarget().getClass())) {
			binder.setValidator(this.validator);
		}
		if (this.conversionService != null) {
			binder.setConversionService(this.conversionService);
		}
		if (this.propertyEditorRegistrars != null) {
			for (PropertyEditorRegistrar propertyEditorRegistrar : this.propertyEditorRegistrars) {
				propertyEditorRegistrar.registerCustomEditors(binder);
			}
		}
	}

}

```



## WebDataBinderFactory.java

```java
package org.springframework.web.bind.support;

import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.context.request.NativeWebRequest;

public interface WebDataBinderFactory {

	WebDataBinder createBinder(NativeWebRequest webRequest, Object target, String objectName) throws Exception;

}

```

### DefaultDataBinderFactory.java

```java
package org.springframework.web.bind.support;

import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.context.request.NativeWebRequest;

/**
 * 创建WebRequestDataBinder}并使用WebBindingInitializer初始化
 */
public class DefaultDataBinderFactory implements WebDataBinderFactory {

	private final WebBindingInitializer initializer;

	public DefaultDataBinderFactory(WebBindingInitializer initializer) {
		this.initializer = initializer;
	}

	@Override
	public final WebDataBinder createBinder(NativeWebRequest webRequest, Object target, String objectName)
			throws Exception {

		WebDataBinder dataBinder = createBinderInstance(target, objectName, webRequest);
		if (this.initializer != null) {
			this.initializer.initBinder(dataBinder, webRequest);
		}
		initBinder(dataBinder, webRequest);
		return dataBinder;
	}

	protected WebDataBinder createBinderInstance(Object target, String objectName, NativeWebRequest webRequest)
			throws Exception {

		return new WebRequestDataBinder(target, objectName);
	}

	protected void initBinder(WebDataBinder dataBinder, NativeWebRequest webRequest) throws Exception {
	}

}

```

#### InitBinderDataBinderFactory.java

```java
package org.springframework.web.method.annotation;

import java.util.Collections;
import java.util.List;

import org.springframework.util.ObjectUtils;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.support.DefaultDataBinderFactory;
import org.springframework.web.bind.support.WebBindingInitializer;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.method.support.InvocableHandlerMethod;

/**
 * 添加使用@InitBinder标记的方法初始化
 */
public class InitBinderDataBinderFactory extends DefaultDataBinderFactory {

	private final List<InvocableHandlerMethod> binderMethods;

	public InitBinderDataBinderFactory(List<InvocableHandlerMethod> binderMethods, WebBindingInitializer initializer) {
		super(initializer);
		this.binderMethods = (binderMethods != null ? binderMethods : Collections.<InvocableHandlerMethod>emptyList());
	}

	@Override
	public void initBinder(WebDataBinder dataBinder, NativeWebRequest request) throws Exception {
		for (InvocableHandlerMethod binderMethod : this.binderMethods) {
			if (isBinderMethodApplicable(binderMethod, dataBinder)) {
				Object returnValue = binderMethod.invokeForRequest(request, null, dataBinder);
				if (returnValue != null) {
					throw new IllegalStateException(
							"@InitBinder methods must not return a value (should be void): " + binderMethod);
				}
			}
		}
	}

	protected boolean isBinderMethodApplicable(HandlerMethod initBinderMethod, WebDataBinder dataBinder) {
		InitBinder ann = initBinderMethod.getMethodAnnotation(InitBinder.class);
		String[] names = ann.value();
		return (ObjectUtils.isEmpty(names) || ObjectUtils.containsElement(names, dataBinder.getObjectName()));
	}

}

```

##### ServletRequestDataBinderFactory.java

```java
package org.springframework.web.servlet.mvc.method.annotation;

import java.util.List;

import org.springframework.web.bind.ServletRequestDataBinder;
import org.springframework.web.bind.support.WebBindingInitializer;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.annotation.InitBinderDataBinderFactory;
import org.springframework.web.method.support.InvocableHandlerMethod;

/**
 * 创建ServletRequestDataBinder数据绑定器
 */
public class ServletRequestDataBinderFactory extends InitBinderDataBinderFactory {

	public ServletRequestDataBinderFactory(List<InvocableHandlerMethod> binderMethods, WebBindingInitializer initializer) {
		super(binderMethods, initializer);
	}

	@Override
	protected ServletRequestDataBinder createBinderInstance(Object target, String objectName, NativeWebRequest request) {
		return new ExtendedServletRequestDataBinder(target, objectName);
	}

}
```

# ModelAndView

## ModelMap.java

```java
package org.springframework.ui;

import java.util.Collection;
import java.util.LinkedHashMap;
import java.util.Map;

import org.springframework.core.Conventions;
import org.springframework.util.Assert;

/**
 * 保存model的map扩展
 */
@SuppressWarnings("serial")
public class ModelMap extends LinkedHashMap<String, Object> {

	public ModelMap() {
	}

	public ModelMap(String attributeName, Object attributeValue) {
		addAttribute(attributeName, attributeValue);
	}

	public ModelMap(Object attributeValue) {
		addAttribute(attributeValue);
	}

	public ModelMap addAttribute(String attributeName, Object attributeValue) {
		Assert.notNull(attributeName, "Model attribute name must not be null");
		put(attributeName, attributeValue);
		return this;
	}

	public ModelMap addAttribute(Object attributeValue) {
		Assert.notNull(attributeValue, "Model object must not be null");
		if (attributeValue instanceof Collection && ((Collection<?>) attributeValue).isEmpty()) {
			return this;
		}
		return addAttribute(Conventions.getVariableName(attributeValue), attributeValue);
	}

	public ModelMap addAllAttributes(Collection<?> attributeValues) {
		if (attributeValues != null) {
			for (Object attributeValue : attributeValues) {
				addAttribute(attributeValue);
			}
		}
		return this;
	}

	public ModelMap addAllAttributes(Map<String, ?> attributes) {
		if (attributes != null) {
			putAll(attributes);
		}
		return this;
	}

	public ModelMap mergeAttributes(Map<String, ?> attributes) {
		if (attributes != null) {
			for (Map.Entry<String, ?> entry : attributes.entrySet()) {
				String key = entry.getKey();
				if (!containsKey(key)) {
					put(key, entry.getValue());
				}
			}
		}
		return this;
	}

	public boolean containsAttribute(String attributeName) {
		return containsKey(attributeName);
	}

}

```

## Model.java

```java
package org.springframework.ui;

import java.util.Collection;
import java.util.Map;

public interface Model {

	Model addAttribute(String attributeName, Object attributeValue);

	Model addAttribute(Object attributeValue);

	Model addAllAttributes(Collection<?> attributeValues);

	Model addAllAttributes(Map<String, ?> attributes);

	Model mergeAttributes(Map<String, ?> attributes);

	boolean containsAttribute(String attributeName);

	Map<String, Object> asMap();

}

```

### RedirectAttributes.java

```java
package org.springframework.web.servlet.mvc.support;

import java.util.Collection;
import java.util.Map;

import org.springframework.ui.Model;
import org.springframework.web.servlet.FlashMap;

public interface RedirectAttributes extends Model {

	@Override
	RedirectAttributes addAttribute(String attributeName, Object attributeValue);

	@Override
	RedirectAttributes addAttribute(Object attributeValue);

	@Override
	RedirectAttributes addAllAttributes(Collection<?> attributeValues);

	@Override
	RedirectAttributes mergeAttributes(Map<String, ?> attributes);

	RedirectAttributes addFlashAttribute(String attributeName, Object attributeValue);

	RedirectAttributes addFlashAttribute(Object attributeValue);

	Map<String, ?> getFlashAttributes();
}

```

#### RedirectAttributesModelMap.java

```java
package org.springframework.web.servlet.mvc.support;

import java.util.Collection;
import java.util.Map;

import org.springframework.ui.ModelMap;
import org.springframework.validation.DataBinder;

@SuppressWarnings("serial")
public class RedirectAttributesModelMap extends ModelMap implements RedirectAttributes {

	private final DataBinder dataBinder;

	private final ModelMap flashAttributes = new ModelMap();

	public RedirectAttributesModelMap() {
		this(null);
	}

	public RedirectAttributesModelMap(DataBinder dataBinder) {
		this.dataBinder = dataBinder;
	}

	@Override
	public Map<String, ?> getFlashAttributes() {
		return this.flashAttributes;
	}

	@Override
	public RedirectAttributesModelMap addAttribute(String attributeName, Object attributeValue) {
		super.addAttribute(attributeName, formatValue(attributeValue));
		return this;
	}

	private String formatValue(Object value) {
		if (value == null) {
			return null;
		}
		return (this.dataBinder != null ? this.dataBinder.convertIfNecessary(value, String.class) : value.toString());
	}

	@Override
	public RedirectAttributesModelMap addAttribute(Object attributeValue) {
		super.addAttribute(attributeValue);
		return this;
	}

	@Override
	public RedirectAttributesModelMap addAllAttributes(Collection<?> attributeValues) {
		super.addAllAttributes(attributeValues);
		return this;
	}

	@Override
	public RedirectAttributesModelMap addAllAttributes(Map<String, ?> attributes) {
		if (attributes != null) {
			for (String key : attributes.keySet()) {
				addAttribute(key, attributes.get(key));
			}
		}
		return this;
	}

	@Override
	public RedirectAttributesModelMap mergeAttributes(Map<String, ?> attributes) {
		if (attributes != null) {
			for (String key : attributes.keySet()) {
				if (!containsKey(key)) {
					addAttribute(key, attributes.get(key));
				}
			}
		}
		return this;
	}

	@Override
	public Map<String, Object> asMap() {
		return this;
	}

	@Override
	public Object put(String key, Object value) {
		return super.put(key, formatValue(value));
	}

	@Override
	public void putAll(Map<? extends String, ? extends Object> map) {
		if (map != null) {
			for (String key : map.keySet()) {
				put(key, formatValue(map.get(key)));
			}
		}
	}

	@Override
	public RedirectAttributes addFlashAttribute(String attributeName, Object attributeValue) {
		this.flashAttributes.addAttribute(attributeName, attributeValue);
		return this;
	}

	@Override
	public RedirectAttributes addFlashAttribute(Object attributeValue) {
		this.flashAttributes.addAttribute(attributeValue);
		return this;
	}

}

```



### ExtendedModelMap.java

```java
package org.springframework.ui;

import java.util.Collection;
import java.util.Map;

@SuppressWarnings("serial")
public class ExtendedModelMap extends ModelMap implements Model {

	@Override
	public ExtendedModelMap addAttribute(String attributeName, Object attributeValue) {
		super.addAttribute(attributeName, attributeValue);
		return this;
	}

	@Override
	public ExtendedModelMap addAttribute(Object attributeValue) {
		super.addAttribute(attributeValue);
		return this;
	}

	@Override
	public ExtendedModelMap addAllAttributes(Collection<?> attributeValues) {
		super.addAllAttributes(attributeValues);
		return this;
	}

	@Override
	public ExtendedModelMap addAllAttributes(Map<String, ?> attributes) {
		super.addAllAttributes(attributes);
		return this;
	}

	@Override
	public ExtendedModelMap mergeAttributes(Map<String, ?> attributes) {
		super.mergeAttributes(attributes);
		return this;
	}

	@Override
	public Map<String, Object> asMap() {
		return this;
	}

}

```

#### BindingAwareModelMap.java

```java
package org.springframework.validation.support;

import java.util.Map;

import org.springframework.ui.ExtendedModelMap;
import org.springframework.validation.BindingResult;

/**
 * 添加数据绑定拓展
 */
@SuppressWarnings("serial")
public class BindingAwareModelMap extends ExtendedModelMap {

	@Override
	public Object put(String key, Object value) {
		removeBindingResultIfNecessary(key, value);
		return super.put(key, value);
	}

	@Override
	public void putAll(Map<? extends String, ?> map) {
		for (Map.Entry<? extends String, ?> entry : map.entrySet()) {
			removeBindingResultIfNecessary(entry.getKey(), entry.getValue());
		}
		super.putAll(map);
	}

	private void removeBindingResultIfNecessary(Object key, Object value) {
		if (key instanceof String) {
			String attributeName = (String) key;
			if (!attributeName.startsWith(BindingResult.MODEL_KEY_PREFIX)) {
				String bindingResultKey = BindingResult.MODEL_KEY_PREFIX + attributeName;
				BindingResult bindingResult = (BindingResult) get(bindingResultKey);
				if (bindingResult != null && bindingResult.getTarget() != value) {
					remove(bindingResultKey);
				}
			}
		}
	}

}

```

## SessionStatus.java

```java
package org.springframework.web.bind.support;

public interface SessionStatus {

	void setComplete();

	boolean isComplete();

}

```

### SimpleSessionStatus.java

```java
package org.springframework.web.bind.support;

public class SimpleSessionStatus implements SessionStatus {

	private boolean complete = false;


	@Override
	public void setComplete() {
		this.complete = true;
	}

	@Override
	public boolean isComplete() {
		return this.complete;
	}

}

```

## HttpStatus.java

```java
package org.springframework.http;

public enum HttpStatus {
	
  	// 1xx Informational
	CONTINUE(100, "Continue"),
	SWITCHING_PROTOCOLS(101, "Switching Protocols"),
	PROCESSING(102, "Processing"),
	CHECKPOINT(103, "Checkpoint"),
    // 2xx Success
	OK(200, "OK"),
	CREATED(201, "Created"),
	ACCEPTED(202, "Accepted"),
	NON_AUTHORITATIVE_INFORMATION(203, "Non-Authoritative Information"),
	NO_CONTENT(204, "No Content"),
	RESET_CONTENT(205, "Reset Content"),
	PARTIAL_CONTENT(206, "Partial Content"),
	MULTI_STATUS(207, "Multi-Status"),
	ALREADY_REPORTED(208, "Already Reported"),
	IM_USED(226, "IM Used"),
	// 3xx Redirection
	MULTIPLE_CHOICES(300, "Multiple Choices"),
	MOVED_PERMANENTLY(301, "Moved Permanently"),
	FOUND(302, "Found"),
	@Deprecated
	MOVED_TEMPORARILY(302, "Moved Temporarily"),
	SEE_OTHER(303, "See Other"),
	NOT_MODIFIED(304, "Not Modified"),
	@Deprecated
	USE_PROXY(305, "Use Proxy"),
	TEMPORARY_REDIRECT(307, "Temporary Redirect"),
	PERMANENT_REDIRECT(308, "Permanent Redirect"),
	// --- 4xx Client Error ---
	BAD_REQUEST(400, "Bad Request"),
	UNAUTHORIZED(401, "Unauthorized"),
	PAYMENT_REQUIRED(402, "Payment Required"),
	FORBIDDEN(403, "Forbidden"),
	NOT_FOUND(404, "Not Found"),
	METHOD_NOT_ALLOWED(405, "Method Not Allowed"),
	NOT_ACCEPTABLE(406, "Not Acceptable"),
	PROXY_AUTHENTICATION_REQUIRED(407, "Proxy Authentication Required"),
	REQUEST_TIMEOUT(408, "Request Timeout"),
	CONFLICT(409, "Conflict"),
	GONE(410, "Gone"),
	LENGTH_REQUIRED(411, "Length Required"),
	PRECONDITION_FAILED(412, "Precondition Failed"),
	PAYLOAD_TOO_LARGE(413, "Payload Too Large"),
	@Deprecated
	REQUEST_ENTITY_TOO_LARGE(413, "Request Entity Too Large"),
	URI_TOO_LONG(414, "URI Too Long"),
	@Deprecated
	REQUEST_URI_TOO_LONG(414, "Request-URI Too Long"),
	UNSUPPORTED_MEDIA_TYPE(415, "Unsupported Media Type"),
	REQUESTED_RANGE_NOT_SATISFIABLE(416, "Requested range not satisfiable"),
	EXPECTATION_FAILED(417, "Expectation Failed"),
	I_AM_A_TEAPOT(418, "I'm a teapot"),
	@Deprecated
	INSUFFICIENT_SPACE_ON_RESOURCE(419, "Insufficient Space On Resource"),
	@Deprecated
	METHOD_FAILURE(420, "Method Failure"),
	@Deprecated
	DESTINATION_LOCKED(421, "Destination Locked"),
	UNPROCESSABLE_ENTITY(422, "Unprocessable Entity"),
	LOCKED(423, "Locked"),
	FAILED_DEPENDENCY(424, "Failed Dependency"),
	UPGRADE_REQUIRED(426, "Upgrade Required"),
	PRECONDITION_REQUIRED(428, "Precondition Required"),
	TOO_MANY_REQUESTS(429, "Too Many Requests"),
	REQUEST_HEADER_FIELDS_TOO_LARGE(431, "Request Header Fields Too Large"),
	UNAVAILABLE_FOR_LEGAL_REASONS(451, "Unavailable For Legal Reasons"),

	// --- 5xx Server Error ---
	INTERNAL_SERVER_ERROR(500, "Internal Server Error"),
	NOT_IMPLEMENTED(501, "Not Implemented"),
	BAD_GATEWAY(502, "Bad Gateway"),
	SERVICE_UNAVAILABLE(503, "Service Unavailable"),
	GATEWAY_TIMEOUT(504, "Gateway Timeout"),
	HTTP_VERSION_NOT_SUPPORTED(505, "HTTP Version not supported"),
	VARIANT_ALSO_NEGOTIATES(506, "Variant Also Negotiates"),
	INSUFFICIENT_STORAGE(507, "Insufficient Storage"),
	LOOP_DETECTED(508, "Loop Detected"),
	BANDWIDTH_LIMIT_EXCEEDED(509, "Bandwidth Limit Exceeded"),
	NOT_EXTENDED(510, "Not Extended"),
	NETWORK_AUTHENTICATION_REQUIRED(511, "Network Authentication Required");


	private final int value;

	private final String reasonPhrase;


	HttpStatus(int value, String reasonPhrase) {
		this.value = value;
		this.reasonPhrase = reasonPhrase;
	}
	public int value() {
		return this.value;
	}
	public String getReasonPhrase() {
		return this.reasonPhrase;
	}

	public boolean is1xxInformational() {
		return Series.INFORMATIONAL.equals(series());
	}

	public boolean is2xxSuccessful() {
		return Series.SUCCESSFUL.equals(series());
	}

	public boolean is3xxRedirection() {
		return Series.REDIRECTION.equals(series());
	}

	public boolean is4xxClientError() {
		return Series.CLIENT_ERROR.equals(series());
	}

	public boolean is5xxServerError() {
		return Series.SERVER_ERROR.equals(series());
	}

	public Series series() {
		return Series.valueOf(this);
	}

	@Override
	public String toString() {
		return Integer.toString(this.value);
	}

	public static HttpStatus valueOf(int statusCode) {
		for (HttpStatus status : values()) {
			if (status.value == statusCode) {
				return status;
			}
		}
		throw new IllegalArgumentException("No matching constant for [" + statusCode + "]");
	}

	public enum Series {

		INFORMATIONAL(1),
		SUCCESSFUL(2),
		REDIRECTION(3),
		CLIENT_ERROR(4),
		SERVER_ERROR(5);

		private final int value;

		Series(int value) {
			this.value = value;
		}

		public int value() {
			return this.value;
		}

		public static Series valueOf(int status) {
			int seriesCode = status / 100;
			for (Series series : values()) {
				if (series.value == seriesCode) {
					return series;
				}
			}
			throw new IllegalArgumentException("No matching constant for [" + status + "]");
		}

		public static Series valueOf(HttpStatus status) {
			return valueOf(status.value);
		}
	}

}

```

## ModelAndViewContainer.java

```java
package org.springframework.web.method.support;

import java.util.HashSet;
import java.util.Map;
import java.util.Set;

import org.springframework.http.HttpStatus;
import org.springframework.ui.Model;
import org.springframework.ui.ModelMap;
import org.springframework.validation.support.BindingAwareModelMap;
import org.springframework.web.bind.support.SessionStatus;
import org.springframework.web.bind.support.SimpleSessionStatus;

public class ModelAndViewContainer {

	private boolean ignoreDefaultModelOnRedirect = false;

	private Object view;

	private final ModelMap defaultModel = new BindingAwareModelMap();

	private ModelMap redirectModel;

	private boolean redirectModelScenario = false;

	private HttpStatus status;

	private final Set<String> noBinding = new HashSet<String>(4);

	private final Set<String> bindingDisabled = new HashSet<String>(4);

	private final SessionStatus sessionStatus = new SimpleSessionStatus();

	private boolean requestHandled = false;

	@Override
	public String toString() {
		StringBuilder sb = new StringBuilder("ModelAndViewContainer: ");
		if (!isRequestHandled()) {
			if (isViewReference()) {
				sb.append("reference to view with name '").append(this.view).append("'");
			}
			else {
				sb.append("View is [").append(this.view).append(']');
			}
			if (useDefaultModel()) {
				sb.append("; default model ");
			}
			else {
				sb.append("; redirect model ");
			}
			sb.append(getModel());
		}
		else {
			sb.append("Request handled directly");
		}
		return sb.toString();
	}

}

```

# WEBRequest

### RequestAttributes.java

```java
package org.springframework.web.context.request;

public interface RequestAttributes {

	int SCOPE_REQUEST = 0;
	int SCOPE_SESSION = 1;
	int SCOPE_GLOBAL_SESSION = 2;
	String REFERENCE_REQUEST = "request";
	String REFERENCE_SESSION = "session";

	Object getAttribute(String name, int scope);

	void setAttribute(String name, Object value, int scope);

	void removeAttribute(String name, int scope);

	String[] getAttributeNames(int scope);

	void registerDestructionCallback(String name, Runnable callback, int scope);

	Object resolveReference(String key);

	String getSessionId();

	Object getSessionMutex();

}

```

### WebRequest.java

```java
package org.springframework.web.context.request;

import java.security.Principal;
import java.util.Iterator;
import java.util.Locale;
import java.util.Map;


public interface WebRequest extends RequestAttributes {

	String getHeader(String headerName);

	String[] getHeaderValues(String headerName);

	Iterator<String> getHeaderNames();

	String getParameter(String paramName);

	String[] getParameterValues(String paramName);

	Iterator<String> getParameterNames();

	Map<String, String[]> getParameterMap();

	Locale getLocale();

	String getContextPath();

	String getRemoteUser();

	Principal getUserPrincipal();

	boolean isUserInRole(String role);

	boolean isSecure();

	boolean checkNotModified(long lastModifiedTimestamp);

	boolean checkNotModified(String etag);

	boolean checkNotModified(String etag, long lastModifiedTimestamp);

	String getDescription(boolean includeClientInfo);

}

```

#### NativeWebRequest.java

```java
package org.springframework.web.context.request;

public interface NativeWebRequest extends WebRequest {

	Object getNativeRequest();

	Object getNativeResponse();

	<T> T getNativeRequest(Class<T> requiredType);

	<T> T getNativeResponse(Class<T> requiredType);

}

```

# HandlerMethodArgumentResolver

## HandlerMethodArgumentResolver.java

```java
package org.springframework.web.method.support;

import org.springframework.core.MethodParameter;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;

public interface HandlerMethodArgumentResolver {
  
	boolean supportsParameter(MethodParameter parameter);

	Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception;

}

```


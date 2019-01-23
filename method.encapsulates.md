# MethodParameter

## ParameterNameDiscoverer.java

```java
package org.springframework.core;
/**
 * 参数名发现器
 */
public interface ParameterNameDiscoverer {

	String[] getParameterNames(Method method);

	String[] getParameterNames(Constructor<?> ctor);

}

```

### LocalVariableTableParameterNameDiscoverer.java

```java
package org.springframework.core;
/**
 * 通过ASM解析字节码获取本地变量表中的DEBUG信息获取参数名
 */
public class LocalVariableTableParameterNameDiscoverer implements ParameterNameDiscoverer {

	private static final Log logger = LogFactory.getLog(LocalVariableTableParameterNameDiscoverer.class);

	// marker object for classes that do not have any debug info
	private static final Map<Member, String[]> NO_DEBUG_INFO_MAP = Collections.emptyMap();

	// the cache uses a nested index (value is a map) to keep the top level cache relatively small in size
	private final Map<Class<?>, Map<Member, String[]>> parameterNamesCache =
			new ConcurrentHashMap<Class<?>, Map<Member, String[]>>(32);


	@Override
	public String[] getParameterNames(Method method) {
		Method originalMethod = BridgeMethodResolver.findBridgedMethod(method);
		Class<?> declaringClass = originalMethod.getDeclaringClass();
		Map<Member, String[]> map = this.parameterNamesCache.get(declaringClass);
		if (map == null) {
			map = inspectClass(declaringClass);
			this.parameterNamesCache.put(declaringClass, map);
		}
		if (map != NO_DEBUG_INFO_MAP) {
			return map.get(originalMethod);
		}
		return null;
	}

	@Override
	public String[] getParameterNames(Constructor<?> ctor) {
		Class<?> declaringClass = ctor.getDeclaringClass();
		Map<Member, String[]> map = this.parameterNamesCache.get(declaringClass);
		if (map == null) {
			map = inspectClass(declaringClass);
			this.parameterNamesCache.put(declaringClass, map);
		}
		if (map != NO_DEBUG_INFO_MAP) {
			return map.get(ctor);
		}
		return null;
	}

	/**
	 * Inspects the target class. Exceptions will be logged and a maker map returned
	 * to indicate the lack of debug information.
	 */
	private Map<Member, String[]> inspectClass(Class<?> clazz) {
		InputStream is = clazz.getResourceAsStream(ClassUtils.getClassFileName(clazz));
		if (is == null) {
			// We couldn't load the class file, which is not fatal as it
			// simply means this method of discovering parameter names won't work.
			if (logger.isDebugEnabled()) {
				logger.debug("Cannot find '.class' file for class [" + clazz +
						"] - unable to determine constructor/method parameter names");
			}
			return NO_DEBUG_INFO_MAP;
		}
		try {
			ClassReader classReader = new ClassReader(is);
			Map<Member, String[]> map = new ConcurrentHashMap<Member, String[]>(32);
			classReader.accept(new ParameterNameDiscoveringVisitor(clazz, map), 0);
			return map;
		}
		catch (IOException ex) {
			if (logger.isDebugEnabled()) {
				logger.debug("Exception thrown while reading '.class' file for class [" + clazz +
						"] - unable to determine constructor/method parameter names", ex);
			}
		}
		catch (IllegalArgumentException ex) {
			if (logger.isDebugEnabled()) {
				logger.debug("ASM ClassReader failed to parse class file [" + clazz +
						"], probably due to a new Java class file version that isn't supported yet " +
						"- unable to determine constructor/method parameter names", ex);
			}
		}
		finally {
			try {
				is.close();
			}
			catch (IOException ex) {
				// ignore
			}
		}
		return NO_DEBUG_INFO_MAP;
	}


	/**
	 * Helper class that inspects all methods (constructor included) and then
	 * attempts to find the parameter names for that member.
	 */
	private static class ParameterNameDiscoveringVisitor extends ClassVisitor {

		private static final String STATIC_CLASS_INIT = "<clinit>";

		private final Class<?> clazz;

		private final Map<Member, String[]> memberMap;

		public ParameterNameDiscoveringVisitor(Class<?> clazz, Map<Member, String[]> memberMap) {
			super(SpringAsmInfo.ASM_VERSION);
			this.clazz = clazz;
			this.memberMap = memberMap;
		}

		@Override
		public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
			// exclude synthetic + bridged && static class initialization
			if (!isSyntheticOrBridged(access) && !STATIC_CLASS_INIT.equals(name)) {
				return new LocalVariableTableVisitor(clazz, memberMap, name, desc, isStatic(access));
			}
			return null;
		}

		private static boolean isSyntheticOrBridged(int access) {
			return (((access & Opcodes.ACC_SYNTHETIC) | (access & Opcodes.ACC_BRIDGE)) > 0);
		}

		private static boolean isStatic(int access) {
			return ((access & Opcodes.ACC_STATIC) > 0);
		}
	}


	private static class LocalVariableTableVisitor extends MethodVisitor {

		private static final String CONSTRUCTOR = "<init>";

		private final Class<?> clazz;

		private final Map<Member, String[]> memberMap;

		private final String name;

		private final Type[] args;

		private final String[] parameterNames;

		private final boolean isStatic;

		private boolean hasLvtInfo = false;

		/*
		 * The nth entry contains the slot index of the LVT table entry holding the
		 * argument name for the nth parameter.
		 */
		private final int[] lvtSlotIndex;

		public LocalVariableTableVisitor(Class<?> clazz, Map<Member, String[]> map, String name, String desc, boolean isStatic) {
			super(SpringAsmInfo.ASM_VERSION);
			this.clazz = clazz;
			this.memberMap = map;
			this.name = name;
			this.args = Type.getArgumentTypes(desc);
			this.parameterNames = new String[this.args.length];
			this.isStatic = isStatic;
			this.lvtSlotIndex = computeLvtSlotIndices(isStatic, this.args);
		}

		@Override
		public void visitLocalVariable(String name, String description, String signature, Label start, Label end, int index) {
			this.hasLvtInfo = true;
			for (int i = 0; i < this.lvtSlotIndex.length; i++) {
				if (this.lvtSlotIndex[i] == index) {
					this.parameterNames[i] = name;
				}
			}
		}

		@Override
		public void visitEnd() {
			if (this.hasLvtInfo || (this.isStatic && this.parameterNames.length == 0)) {
				// visitLocalVariable will never be called for static no args methods
				// which doesn't use any local variables.
				// This means that hasLvtInfo could be false for that kind of methods
				// even if the class has local variable info.
				this.memberMap.put(resolveMember(), this.parameterNames);
			}
		}

		private Member resolveMember() {
			ClassLoader loader = this.clazz.getClassLoader();
			Class<?>[] argTypes = new Class<?>[this.args.length];
			for (int i = 0; i < this.args.length; i++) {
				argTypes[i] = ClassUtils.resolveClassName(this.args[i].getClassName(), loader);
			}
			try {
				if (CONSTRUCTOR.equals(this.name)) {
					return this.clazz.getDeclaredConstructor(argTypes);
				}
				return this.clazz.getDeclaredMethod(this.name, argTypes);
			}
			catch (NoSuchMethodException ex) {
				throw new IllegalStateException("Method [" + this.name +
						"] was discovered in the .class file but cannot be resolved in the class object", ex);
			}
		}

		private static int[] computeLvtSlotIndices(boolean isStatic, Type[] paramTypes) {
			int[] lvtIndex = new int[paramTypes.length];
			int nextIndex = (isStatic ? 0 : 1);
			for (int i = 0; i < paramTypes.length; i++) {
				lvtIndex[i] = nextIndex;
				if (isWideType(paramTypes[i])) {
					nextIndex += 2;
				}
				else {
					nextIndex++;
				}
			}
			return lvtIndex;
		}

		private static boolean isWideType(Type aType) {
			// float is not a wide type
			return (aType == Type.LONG_TYPE || aType == Type.DOUBLE_TYPE);
		}
	}

}

```

### StandardReflectionParameterNameDiscoverer.java

```java
package org.springframework.core;
/**
 * 使用JDK8的反射工具获取参数名称，基于"-parameters"编译标记
 */
@UsesJava8
public class StandardReflectionParameterNameDiscoverer implements ParameterNameDiscoverer {

	@Override
	public String[] getParameterNames(Method method) {
		Parameter[] parameters = method.getParameters();
		String[] parameterNames = new String[parameters.length];
		for (int i = 0; i < parameters.length; i++) {
			Parameter param = parameters[i];
			if (!param.isNamePresent()) {
				return null;
			}
			parameterNames[i] = param.getName();
		}
		return parameterNames;
	}

	@Override
	public String[] getParameterNames(Constructor<?> ctor) {
		Parameter[] parameters = ctor.getParameters();
		String[] parameterNames = new String[parameters.length];
		for (int i = 0; i < parameters.length; i++) {
			Parameter param = parameters[i];
			if (!param.isNamePresent()) {
				return null;
			}
			parameterNames[i] = param.getName();
		}
		return parameterNames;
	}

}

```

### PrioritizedParameterNameDiscoverer.java

```java
package org.springframework.core;
/**
 * 优先级参数名发现器。使用一个双向链表保存参数名发现器，一次遍历，直到获取到参数名。
 */
public class PrioritizedParameterNameDiscoverer implements ParameterNameDiscoverer {

	private final List<ParameterNameDiscoverer> parameterNameDiscoverers =
			new LinkedList<ParameterNameDiscoverer>();


	/**
	 * Add a further {@link ParameterNameDiscoverer} delegate to the list of
	 * discoverers that this {@code PrioritizedParameterNameDiscoverer} checks.
	 */
	public void addDiscoverer(ParameterNameDiscoverer pnd) {
		this.parameterNameDiscoverers.add(pnd);
	}


	@Override
	public String[] getParameterNames(Method method) {
		for (ParameterNameDiscoverer pnd : this.parameterNameDiscoverers) {
			String[] result = pnd.getParameterNames(method);
			if (result != null) {
				return result;
			}
		}
		return null;
	}

	@Override
	public String[] getParameterNames(Constructor<?> ctor) {
		for (ParameterNameDiscoverer pnd : this.parameterNameDiscoverers) {
			String[] result = pnd.getParameterNames(ctor);
			if (result != null) {
				return result;
			}
		}
		return null;
	}

}

```

#### DefaultParameterNameDiscoverer.java

```java
package org.springframework.core;
/**
 * 默认的参数名发现器。如果JDK8可用，先使用JDK8的反射获取参数名。失败回退使用本地变量表的DEBUG信息获取参数名。
 */
public class DefaultParameterNameDiscoverer extends PrioritizedParameterNameDiscoverer {

	private static final boolean standardReflectionAvailable = ClassUtils.isPresent(
			"java.lang.reflect.Executable", DefaultParameterNameDiscoverer.class.getClassLoader());


	public DefaultParameterNameDiscoverer() {
		if (standardReflectionAvailable) {
			addDiscoverer(new StandardReflectionParameterNameDiscoverer());
		}
		addDiscoverer(new LocalVariableTableParameterNameDiscoverer());
	}

}

```

## MethodParameter.java

```java
package org.springframework.core;

/**
 * 方法或构造函数的参数封装
 */
public class MethodParameter {

	private static final Annotation[] EMPTY_ANNOTATION_ARRAY = new Annotation[0];

	private static final Class<?> javaUtilOptionalClass;

	static {
		Class<?> clazz;
		try {
			clazz = ClassUtils.forName("java.util.Optional", MethodParameter.class.getClassLoader());
		}
		catch (ClassNotFoundException ex) {
			// Java 8 not available - Optional references simply not supported then.
			clazz = null;
		}
		javaUtilOptionalClass = clazz;
	}


	private final Method method;

	private final Constructor<?> constructor;

	private final int parameterIndex;

	private int nestingLevel = 1;

	/** Map from Integer level to Integer type index */
	Map<Integer, Integer> typeIndexesPerLevel;

	private volatile Class<?> containingClass;

	private volatile Class<?> parameterType;

	private volatile Type genericParameterType;

	private volatile Annotation[] parameterAnnotations;

	private volatile ParameterNameDiscoverer parameterNameDiscoverer;

	private volatile String parameterName;

	private volatile MethodParameter nestedMethodParameter;

	public MethodParameter(Method method, int parameterIndex) {
		this(method, parameterIndex, 1);
	}

	/**
	 * @param parameterIndex -1代表返回类型，0代表第一个参数，1代表第二个参数
	 * @param nestingLevel 内嵌等级，比如List<List<String>>,1代表List<List<String>>，2代表List<String>
	 */
	public MethodParameter(Method method, int parameterIndex, int nestingLevel) {
		Assert.notNull(method, "Method must not be null");
		this.method = method;
		this.parameterIndex = parameterIndex;
		this.nestingLevel = nestingLevel;
		this.constructor = null;
	}

	public MethodParameter(Constructor<?> constructor, int parameterIndex) {
		this(constructor, parameterIndex, 1);
	}

	public MethodParameter(Constructor<?> constructor, int parameterIndex, int nestingLevel) {
		Assert.notNull(constructor, "Constructor must not be null");
		this.constructor = constructor;
		this.parameterIndex = parameterIndex;
		this.nestingLevel = nestingLevel;
		this.method = null;
	}
	public MethodParameter(MethodParameter original) {
		Assert.notNull(original, "Original must not be null");
		this.method = original.method;
		this.constructor = original.constructor;
		this.parameterIndex = original.parameterIndex;
		this.nestingLevel = original.nestingLevel;
		this.typeIndexesPerLevel = original.typeIndexesPerLevel;
		this.containingClass = original.containingClass;
		this.parameterType = original.parameterType;
		this.genericParameterType = original.genericParameterType;
		this.parameterAnnotations = original.parameterAnnotations;
		this.parameterNameDiscoverer = original.parameterNameDiscoverer;
		this.parameterName = original.parameterName;
	}

	public Method getMethod() {
		return this.method;
	}

	public Constructor<?> getConstructor() {
		return this.constructor;
	}

	public Class<?> getDeclaringClass() {
		return getMember().getDeclaringClass();
	}

	public Member getMember() {
		if (this.method != null) {
			return this.method;
		}
		else {
			return this.constructor;
		}
	}

	public AnnotatedElement getAnnotatedElement() {
		if (this.method != null) {
			return this.method;
		}
		else {
			return this.constructor;
		}
	}

	public int getParameterIndex() {
		return this.parameterIndex;
	}

	public void increaseNestingLevel() {
		this.nestingLevel++;
	}

	public void decreaseNestingLevel() {
		getTypeIndexesPerLevel().remove(this.nestingLevel);
		this.nestingLevel--;
	}

	public int getNestingLevel() {
		return this.nestingLevel;
	}

	public void setTypeIndexForCurrentLevel(int typeIndex) {
		getTypeIndexesPerLevel().put(this.nestingLevel, typeIndex);
	}

	public Integer getTypeIndexForCurrentLevel() {
		return getTypeIndexForLevel(this.nestingLevel);
	}

	public Integer getTypeIndexForLevel(int nestingLevel) {
		return getTypeIndexesPerLevel().get(nestingLevel);
	}

	private Map<Integer, Integer> getTypeIndexesPerLevel() {
		if (this.typeIndexesPerLevel == null) {
			this.typeIndexesPerLevel = new HashMap<Integer, Integer>(4);
		}
		return this.typeIndexesPerLevel;
	}

	public MethodParameter nested() {
		if (this.nestedMethodParameter != null) {
			return this.nestedMethodParameter;
		}
		MethodParameter nestedParam = clone();
		nestedParam.nestingLevel = this.nestingLevel + 1;
		this.nestedMethodParameter = nestedParam;
		return nestedParam;
	}

	public boolean isOptional() {
		return (getParameterType() == javaUtilOptionalClass);
	}

	public MethodParameter nestedIfOptional() {
		return (isOptional() ? nested() : this);
	}

	void setContainingClass(Class<?> containingClass) {
		this.containingClass = containingClass;
	}

	public Class<?> getContainingClass() {
		return (this.containingClass != null ? this.containingClass : getDeclaringClass());
	}

	void setParameterType(Class<?> parameterType) {
		this.parameterType = parameterType;
	}

	public Class<?> getParameterType() {
		Class<?> paramType = this.parameterType;
		if (paramType == null) {
			if (this.parameterIndex < 0) {
				Method method = getMethod();
				paramType = (method != null ? method.getReturnType() : void.class);
			}
			else {
				paramType = (this.method != null ?
						this.method.getParameterTypes()[this.parameterIndex] :
						this.constructor.getParameterTypes()[this.parameterIndex]);
			}
			this.parameterType = paramType;
		}
		return paramType;
	}

	public Type getGenericParameterType() {
		Type paramType = this.genericParameterType;
		if (paramType == null) {
			if (this.parameterIndex < 0) {
				Method method = getMethod();
				paramType = (method != null ? method.getGenericReturnType() : void.class);
			}
			else {
				Type[] genericParameterTypes = (this.method != null ?
						this.method.getGenericParameterTypes() : this.constructor.getGenericParameterTypes());
				int index = this.parameterIndex;
				if (this.constructor != null && this.constructor.getDeclaringClass().isMemberClass() &&
						!Modifier.isStatic(this.constructor.getDeclaringClass().getModifiers()) &&
						genericParameterTypes.length == this.constructor.getParameterTypes().length - 1) {
					// Bug in javac: type array excludes enclosing instance parameter
					// for inner classes with at least one generic constructor parameter,
					// so access it with the actual parameter index lowered by 1
					index = this.parameterIndex - 1;
				}
				paramType = (index >= 0 && index < genericParameterTypes.length ?
						genericParameterTypes[index] : getParameterType());
			}
			this.genericParameterType = paramType;
		}
		return paramType;
	}

	public Class<?> getNestedParameterType() {
		if (this.nestingLevel > 1) {
			Type type = getGenericParameterType();
			for (int i = 2; i <= this.nestingLevel; i++) {
				if (type instanceof ParameterizedType) {
					Type[] args = ((ParameterizedType) type).getActualTypeArguments();
					Integer index = getTypeIndexForLevel(i);
					type = args[index != null ? index : args.length - 1];
				}
				// TODO: Object.class if unresolvable
			}
			if (type instanceof Class) {
				return (Class<?>) type;
			}
			else if (type instanceof ParameterizedType) {
				Type arg = ((ParameterizedType) type).getRawType();
				if (arg instanceof Class) {
					return (Class<?>) arg;
				}
			}
			return Object.class;
		}
		else {
			return getParameterType();
		}
	}

	public Type getNestedGenericParameterType() {
		if (this.nestingLevel > 1) {
			Type type = getGenericParameterType();
			for (int i = 2; i <= this.nestingLevel; i++) {
				if (type instanceof ParameterizedType) {
					Type[] args = ((ParameterizedType) type).getActualTypeArguments();
					Integer index = getTypeIndexForLevel(i);
					type = args[index != null ? index : args.length - 1];
				}
			}
			return type;
		}
		else {
			return getGenericParameterType();
		}
	}

	public Annotation[] getMethodAnnotations() {
		return adaptAnnotationArray(getAnnotatedElement().getAnnotations());
	}

	public <A extends Annotation> A getMethodAnnotation(Class<A> annotationType) {
		return adaptAnnotation(getAnnotatedElement().getAnnotation(annotationType));
	}

	public <A extends Annotation> boolean hasMethodAnnotation(Class<A> annotationType) {
		return getAnnotatedElement().isAnnotationPresent(annotationType);
	}

	public Annotation[] getParameterAnnotations() {
		Annotation[] paramAnns = this.parameterAnnotations;
		if (paramAnns == null) {
			Annotation[][] annotationArray = (this.method != null ?
					this.method.getParameterAnnotations() : this.constructor.getParameterAnnotations());
			int index = this.parameterIndex;
			if (this.constructor != null && this.constructor.getDeclaringClass().isMemberClass() &&
					!Modifier.isStatic(this.constructor.getDeclaringClass().getModifiers()) &&
					annotationArray.length == this.constructor.getParameterTypes().length - 1) {
				// Bug in javac in JDK <9: annotation array excludes enclosing instance parameter
				// for inner classes, so access it with the actual parameter index lowered by 1
				index = this.parameterIndex - 1;
			}
			paramAnns = (index >= 0 && index < annotationArray.length ?
					adaptAnnotationArray(annotationArray[index]) : EMPTY_ANNOTATION_ARRAY);
			this.parameterAnnotations = paramAnns;
		}
		return paramAnns;
	}

	public boolean hasParameterAnnotations() {
		return (getParameterAnnotations().length != 0);
	}

	@SuppressWarnings("unchecked")
	public <A extends Annotation> A getParameterAnnotation(Class<A> annotationType) {
		Annotation[] anns = getParameterAnnotations();
		for (Annotation ann : anns) {
			if (annotationType.isInstance(ann)) {
				return (A) ann;
			}
		}
		return null;
	}

	public <A extends Annotation> boolean hasParameterAnnotation(Class<A> annotationType) {
		return (getParameterAnnotation(annotationType) != null);
	}

	public void initParameterNameDiscovery(ParameterNameDiscoverer parameterNameDiscoverer) {
		this.parameterNameDiscoverer = parameterNameDiscoverer;
	}

	public String getParameterName() {
		ParameterNameDiscoverer discoverer = this.parameterNameDiscoverer;
		if (discoverer != null) {
			String[] parameterNames = (this.method != null ?
					discoverer.getParameterNames(this.method) : discoverer.getParameterNames(this.constructor));
			if (parameterNames != null) {
				this.parameterName = parameterNames[this.parameterIndex];
			}
			this.parameterNameDiscoverer = null;
		}
		return this.parameterName;
	}

	protected <A extends Annotation> A adaptAnnotation(A annotation) {
		return annotation;
	}

	protected Annotation[] adaptAnnotationArray(Annotation[] annotations) {
		return annotations;
	}


	@Override
	public boolean equals(Object other) {
		if (this == other) {
			return true;
		}
		if (!(other instanceof MethodParameter)) {
			return false;
		}
		MethodParameter otherParam = (MethodParameter) other;
		return (this.parameterIndex == otherParam.parameterIndex && getMember().equals(otherParam.getMember()));
	}

	@Override
	public int hashCode() {
		return (getMember().hashCode() * 31 + this.parameterIndex);
	}

	@Override
	public String toString() {
		return (this.method != null ? "method '" + this.method.getName() + "'" : "constructor") +
				" parameter " + this.parameterIndex;
	}

	@Override
	public MethodParameter clone() {
		return new MethodParameter(this);
	}

	public static MethodParameter forMethodOrConstructor(Object methodOrConstructor, int parameterIndex) {
		if (methodOrConstructor instanceof Method) {
			return new MethodParameter((Method) methodOrConstructor, parameterIndex);
		}
		else if (methodOrConstructor instanceof Constructor) {
			return new MethodParameter((Constructor<?>) methodOrConstructor, parameterIndex);
		}
		else {
			throw new IllegalArgumentException(
					"Given object [" + methodOrConstructor + "] is neither a Method nor a Constructor");
		}
	}

}

```

### SynthesizingMethodParameter.java

```java
package org.springframework.core.annotation;

import java.lang.annotation.Annotation;
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;

import org.springframework.core.MethodParameter;

/**
 * 一个MethodParameter变体，它综合了通过@AliasFor声明属性别名的注解。
 */
public class SynthesizingMethodParameter extends MethodParameter {

	public SynthesizingMethodParameter(Method method, int parameterIndex) {
		super(method, parameterIndex);
	}

	public SynthesizingMethodParameter(Method method, int parameterIndex, int nestingLevel) {
		super(method, parameterIndex, nestingLevel);
	}

	public SynthesizingMethodParameter(Constructor<?> constructor, int parameterIndex) {
		super(constructor, parameterIndex);
	}

	public SynthesizingMethodParameter(Constructor<?> constructor, int parameterIndex, int nestingLevel) {
		super(constructor, parameterIndex, nestingLevel);
	}

	protected SynthesizingMethodParameter(SynthesizingMethodParameter original) {
		super(original);
	}


	@Override
	protected <A extends Annotation> A adaptAnnotation(A annotation) {
		return AnnotationUtils.synthesizeAnnotation(annotation, getAnnotatedElement());
	}

	@Override
	protected Annotation[] adaptAnnotationArray(Annotation[] annotations) {
		return AnnotationUtils.synthesizeAnnotationArray(annotations, getAnnotatedElement());
	}


	@Override
	public SynthesizingMethodParameter clone() {
		return new SynthesizingMethodParameter(this);
	}

}

```

## HandlerMethod.java

```java
package org.springframework.web.method;
/**
 * SpringMVC处理器方法的封装
 */
public class HandlerMethod {

	/** Logger that is available to subclasses */
	protected final Log logger = LogFactory.getLog(getClass());

	private final Object bean;

	private final BeanFactory beanFactory;

	private final Class<?> beanType;

	private final Method method;

	private final Method bridgedMethod;

	private final MethodParameter[] parameters;

	private HttpStatus responseStatus;

	private String responseStatusReason;

	private HandlerMethod resolvedFromHandlerMethod;

	public HandlerMethod(Object bean, Method method) {
		Assert.notNull(bean, "Bean is required");
		Assert.notNull(method, "Method is required");
		this.bean = bean;
		this.beanFactory = null;
		this.beanType = ClassUtils.getUserClass(bean);
		this.method = method;
		this.bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
		this.parameters = initMethodParameters();
		evaluateResponseStatus();
	}

	public HandlerMethod(Object bean, String methodName, Class<?>... parameterTypes) throws NoSuchMethodException {
		Assert.notNull(bean, "Bean is required");
		Assert.notNull(methodName, "Method name is required");
		this.bean = bean;
		this.beanFactory = null;
		this.beanType = ClassUtils.getUserClass(bean);
		this.method = bean.getClass().getMethod(methodName, parameterTypes);
		this.bridgedMethod = BridgeMethodResolver.findBridgedMethod(this.method);
		this.parameters = initMethodParameters();
		evaluateResponseStatus();
	}

	public HandlerMethod(String beanName, BeanFactory beanFactory, Method method) {
		Assert.hasText(beanName, "Bean name is required");
		Assert.notNull(beanFactory, "BeanFactory is required");
		Assert.notNull(method, "Method is required");
		this.bean = beanName;
		this.beanFactory = beanFactory;
		this.beanType = ClassUtils.getUserClass(beanFactory.getType(beanName));
		this.method = method;
		this.bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
		this.parameters = initMethodParameters();
		evaluateResponseStatus();
	}

	protected HandlerMethod(HandlerMethod handlerMethod) {
		Assert.notNull(handlerMethod, "HandlerMethod is required");
		this.bean = handlerMethod.bean;
		this.beanFactory = handlerMethod.beanFactory;
		this.beanType = handlerMethod.beanType;
		this.method = handlerMethod.method;
		this.bridgedMethod = handlerMethod.bridgedMethod;
		this.parameters = handlerMethod.parameters;
		this.responseStatus = handlerMethod.responseStatus;
		this.responseStatusReason = handlerMethod.responseStatusReason;
		this.resolvedFromHandlerMethod = handlerMethod.resolvedFromHandlerMethod;
	}

	private HandlerMethod(HandlerMethod handlerMethod, Object handler) {
		Assert.notNull(handlerMethod, "HandlerMethod is required");
		Assert.notNull(handler, "Handler object is required");
		this.bean = handler;
		this.beanFactory = handlerMethod.beanFactory;
		this.beanType = handlerMethod.beanType;
		this.method = handlerMethod.method;
		this.bridgedMethod = handlerMethod.bridgedMethod;
		this.parameters = handlerMethod.parameters;
		this.responseStatus = handlerMethod.responseStatus;
		this.responseStatusReason = handlerMethod.responseStatusReason;
		this.resolvedFromHandlerMethod = handlerMethod;
	}


	private MethodParameter[] initMethodParameters() {
		int count = this.bridgedMethod.getParameterTypes().length;
		MethodParameter[] result = new MethodParameter[count];
		for (int i = 0; i < count; i++) {
			HandlerMethodParameter parameter = new HandlerMethodParameter(i);
			GenericTypeResolver.resolveParameterType(parameter, this.beanType);
			result[i] = parameter;
		}
		return result;
	}

	private void evaluateResponseStatus() {
		ResponseStatus annotation = getMethodAnnotation(ResponseStatus.class);
		if (annotation == null) {
			annotation = AnnotatedElementUtils.findMergedAnnotation(getBeanType(), ResponseStatus.class);
		}
		if (annotation != null) {
			this.responseStatus = annotation.code();
			this.responseStatusReason = annotation.reason();
		}
	}

	public Object getBean() {
		return this.bean;
	}

	public Method getMethod() {
		return this.method;
	}

	public Class<?> getBeanType() {
		return this.beanType;
	}

	protected Method getBridgedMethod() {
		return this.bridgedMethod;
	}

	public MethodParameter[] getMethodParameters() {
		return this.parameters;
	}

	protected HttpStatus getResponseStatus() {
		return this.responseStatus;
	}

	protected String getResponseStatusReason() {
		return this.responseStatusReason;
	}

	public MethodParameter getReturnType() {
		return new HandlerMethodParameter(-1);
	}

	public MethodParameter getReturnValueType(Object returnValue) {
		return new ReturnValueMethodParameter(returnValue);
	}

	public boolean isVoid() {
		return Void.TYPE.equals(getReturnType().getParameterType());
	}

	public <A extends Annotation> A getMethodAnnotation(Class<A> annotationType) {
		return AnnotatedElementUtils.findMergedAnnotation(this.method, annotationType);
	}

	public <A extends Annotation> boolean hasMethodAnnotation(Class<A> annotationType) {
		return AnnotatedElementUtils.hasAnnotation(this.method, annotationType);
	}

	public HandlerMethod getResolvedFromHandlerMethod() {
		return this.resolvedFromHandlerMethod;
	}

	public HandlerMethod createWithResolvedBean() {
		Object handler = this.bean;
		if (this.bean instanceof String) {
			String beanName = (String) this.bean;
			handler = this.beanFactory.getBean(beanName);
		}
		return new HandlerMethod(this, handler);
	}

	public String getShortLogMessage() {
		int args = this.method.getParameterTypes().length;
		return getBeanType().getName() + "#" + this.method.getName() + "[" + args + " args]";
	}


	@Override
	public boolean equals(Object other) {
		if (this == other) {
			return true;
		}
		if (!(other instanceof HandlerMethod)) {
			return false;
		}
		HandlerMethod otherMethod = (HandlerMethod) other;
		return (this.bean.equals(otherMethod.bean) && this.method.equals(otherMethod.method));
	}

	@Override
	public int hashCode() {
		return (this.bean.hashCode() * 31 + this.method.hashCode());
	}

	@Override
	public String toString() {
		return this.method.toGenericString();
	}

	protected class HandlerMethodParameter extends SynthesizingMethodParameter {

		public HandlerMethodParameter(int index) {
			super(HandlerMethod.this.bridgedMethod, index);
		}

		protected HandlerMethodParameter(HandlerMethodParameter original) {
			super(original);
		}

		@Override
		public Class<?> getContainingClass() {
			return HandlerMethod.this.getBeanType();
		}

		@Override
		public <T extends Annotation> T getMethodAnnotation(Class<T> annotationType) {
			return HandlerMethod.this.getMethodAnnotation(annotationType);
		}

		@Override
		public <T extends Annotation> boolean hasMethodAnnotation(Class<T> annotationType) {
			return HandlerMethod.this.hasMethodAnnotation(annotationType);
		}

		@Override
		public HandlerMethodParameter clone() {
			return new HandlerMethodParameter(this);
		}
	}

	private class ReturnValueMethodParameter extends HandlerMethodParameter {

		private final Object returnValue;

		public ReturnValueMethodParameter(Object returnValue) {
			super(-1);
			this.returnValue = returnValue;
		}

		protected ReturnValueMethodParameter(ReturnValueMethodParameter original) {
			super(original);
			this.returnValue = original.returnValue;
		}

		@Override
		public Class<?> getParameterType() {
			return (this.returnValue != null ? this.returnValue.getClass() : super.getParameterType());
		}

		@Override
		public ReturnValueMethodParameter clone() {
			return new ReturnValueMethodParameter(this);
		}
	}

}

```


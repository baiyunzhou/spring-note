# HandlerAdapter

## HandlerAdapter.java

```java
package org.springframework.web.servlet;
/**
 * 处理器适配器
 */
public interface HandlerAdapter {

	boolean supports(Object handler);

	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

	long getLastModified(HttpServletRequest request, Object handler);

}

```

### AbstractHandlerMethodAdapter.java

```java
package org.springframework.web.servlet.mvc.method;
/**
 * 支持HandlerMethod的抽象处理器适配器
 */
public abstract class AbstractHandlerMethodAdapter extends WebContentGenerator implements HandlerAdapter, Ordered {

	private int order = Ordered.LOWEST_PRECEDENCE;


	public AbstractHandlerMethodAdapter() {
		// no restriction of HTTP methods by default
		super(false);
	}

	public void setOrder(int order) {
		this.order = order;
	}

	@Override
	public int getOrder() {
		return this.order;
	}

	@Override
	public final boolean supports(Object handler) {
		return (handler instanceof HandlerMethod && supportsInternal((HandlerMethod) handler));
	}

	protected abstract boolean supportsInternal(HandlerMethod handlerMethod);

	@Override
	public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		return handleInternal(request, response, (HandlerMethod) handler);
	}

	protected abstract ModelAndView handleInternal(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception;

	@Override
	public final long getLastModified(HttpServletRequest request, Object handler) {
		return getLastModifiedInternal(request, (HandlerMethod) handler);
	}

	protected abstract long getLastModifiedInternal(HttpServletRequest request, HandlerMethod handlerMethod);

}

```

#### RequestMappingHandlerAdapter.java

```java
package org.springframework.web.servlet.mvc.method.annotation;

/**
 * 支持HandlerMethod的处理器适配器实现
 */
public class RequestMappingHandlerAdapter extends AbstractHandlerMethodAdapter
		implements BeanFactoryAware, InitializingBean {

	public static final MethodFilter INIT_BINDER_METHODS = new MethodFilter() {
		@Override
		public boolean matches(Method method) {
			return (AnnotationUtils.findAnnotation(method, InitBinder.class) != null);
		}
	};

	public static final MethodFilter MODEL_ATTRIBUTE_METHODS = new MethodFilter() {
		@Override
		public boolean matches(Method method) {
			return (AnnotationUtils.findAnnotation(method, RequestMapping.class) == null &&
					AnnotationUtils.findAnnotation(method, ModelAttribute.class) != null);
		}
	};
	//自定义的参数解析器
	private List<HandlerMethodArgumentResolver> customArgumentResolvers;
	//HandlerMethod参数解析器组合
	private HandlerMethodArgumentResolverComposite argumentResolvers;
	//initbinder方法参数解析器组合
	private HandlerMethodArgumentResolverComposite initBinderArgumentResolvers;
	//自定义返回值处理器
	private List<HandlerMethodReturnValueHandler> customReturnValueHandlers;
	//返回子处理器组合
	private HandlerMethodReturnValueHandlerComposite returnValueHandlers;
	//modelAndView解析器
	private List<ModelAndViewResolver> modelAndViewResolvers;
	//内容协商管理器
	private ContentNegotiationManager contentNegotiationManager = new ContentNegotiationManager();
	//HttpMessage转换器
	private List<HttpMessageConverter<?>> messageConverters;
	//请求响应体通知
	private List<Object> requestResponseBodyAdvice = new ArrayList<Object>();
	//web绑定初始化器
	private WebBindingInitializer webBindingInitializer;

	private AsyncTaskExecutor taskExecutor = new SimpleAsyncTaskExecutor("MvcAsync");

	private Long asyncRequestTimeout;

	private CallableProcessingInterceptor[] callableInterceptors = new CallableProcessingInterceptor[0];

	private DeferredResultProcessingInterceptor[] deferredResultInterceptors = new DeferredResultProcessingInterceptor[0];

	private boolean ignoreDefaultModelOnRedirect = false;

	private int cacheSecondsForSessionAttributeHandlers = 0;

	private boolean synchronizeOnSession = false;

	private SessionAttributeStore sessionAttributeStore = new DefaultSessionAttributeStore();
	//参数名查找器
	private ParameterNameDiscoverer parameterNameDiscoverer = new DefaultParameterNameDiscoverer();

	private ConfigurableBeanFactory beanFactory;


	private final Map<Class<?>, SessionAttributesHandler> sessionAttributesHandlerCache =
			new ConcurrentHashMap<Class<?>, SessionAttributesHandler>(64);

	private final Map<Class<?>, Set<Method>> initBinderCache = new ConcurrentHashMap<Class<?>, Set<Method>>(64);

	private final Map<ControllerAdviceBean, Set<Method>> initBinderAdviceCache =
			new LinkedHashMap<ControllerAdviceBean, Set<Method>>();

	private final Map<Class<?>, Set<Method>> modelAttributeCache = new ConcurrentHashMap<Class<?>, Set<Method>>(64);

	private final Map<ControllerAdviceBean, Set<Method>> modelAttributeAdviceCache =
			new LinkedHashMap<ControllerAdviceBean, Set<Method>>();


	public RequestMappingHandlerAdapter() {
		StringHttpMessageConverter stringHttpMessageConverter = new StringHttpMessageConverter();
		stringHttpMessageConverter.setWriteAcceptCharset(false);  // see SPR-7316 不写这个响应头，内容太多了还没用

		this.messageConverters = new ArrayList<HttpMessageConverter<?>>(4);
		this.messageConverters.add(new ByteArrayHttpMessageConverter());
		this.messageConverters.add(stringHttpMessageConverter);
		this.messageConverters.add(new SourceHttpMessageConverter<Source>());
		this.messageConverters.add(new AllEncompassingFormHttpMessageConverter());
	}

	public void setCustomArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
		this.customArgumentResolvers = argumentResolvers;
	}

	public List<HandlerMethodArgumentResolver> getCustomArgumentResolvers() {
		return this.customArgumentResolvers;
	}

	public void setArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
		if (argumentResolvers == null) {
			this.argumentResolvers = null;
		}
		else {
			this.argumentResolvers = new HandlerMethodArgumentResolverComposite();
			this.argumentResolvers.addResolvers(argumentResolvers);
		}
	}

	public List<HandlerMethodArgumentResolver> getArgumentResolvers() {
		return (this.argumentResolvers != null ? this.argumentResolvers.getResolvers() : null);
	}

	public void setInitBinderArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
		if (argumentResolvers == null) {
			this.initBinderArgumentResolvers = null;
		}
		else {
			this.initBinderArgumentResolvers = new HandlerMethodArgumentResolverComposite();
			this.initBinderArgumentResolvers.addResolvers(argumentResolvers);
		}
	}

	public List<HandlerMethodArgumentResolver> getInitBinderArgumentResolvers() {
		return (this.initBinderArgumentResolvers != null ? this.initBinderArgumentResolvers.getResolvers() : null);
	}

	public void setCustomReturnValueHandlers(List<HandlerMethodReturnValueHandler> returnValueHandlers) {
		this.customReturnValueHandlers = returnValueHandlers;
	}

	public List<HandlerMethodReturnValueHandler> getCustomReturnValueHandlers() {
		return this.customReturnValueHandlers;
	}

	public void setReturnValueHandlers(List<HandlerMethodReturnValueHandler> returnValueHandlers) {
		if (returnValueHandlers == null) {
			this.returnValueHandlers = null;
		}
		else {
			this.returnValueHandlers = new HandlerMethodReturnValueHandlerComposite();
			this.returnValueHandlers.addHandlers(returnValueHandlers);
		}
	}

	public List<HandlerMethodReturnValueHandler> getReturnValueHandlers() {
		return (this.returnValueHandlers != null ? this.returnValueHandlers.getHandlers() : null);
	}

	public void setModelAndViewResolvers(List<ModelAndViewResolver> modelAndViewResolvers) {
		this.modelAndViewResolvers = modelAndViewResolvers;
	}

	public List<ModelAndViewResolver> getModelAndViewResolvers() {
		return this.modelAndViewResolvers;
	}

	public void setContentNegotiationManager(ContentNegotiationManager contentNegotiationManager) {
		this.contentNegotiationManager = contentNegotiationManager;
	}

	public void setMessageConverters(List<HttpMessageConverter<?>> messageConverters) {
		this.messageConverters = messageConverters;
	}

	public List<HttpMessageConverter<?>> getMessageConverters() {
		return this.messageConverters;
	}

	public void setRequestBodyAdvice(List<RequestBodyAdvice> requestBodyAdvice) {
		if (requestBodyAdvice != null) {
			this.requestResponseBodyAdvice.addAll(requestBodyAdvice);
		}
	}

	public void setResponseBodyAdvice(List<ResponseBodyAdvice<?>> responseBodyAdvice) {
		if (responseBodyAdvice != null) {
			this.requestResponseBodyAdvice.addAll(responseBodyAdvice);
		}
	}

	public void setWebBindingInitializer(WebBindingInitializer webBindingInitializer) {
		this.webBindingInitializer = webBindingInitializer;
	}

	public WebBindingInitializer getWebBindingInitializer() {
		return this.webBindingInitializer;
	}

	public void setTaskExecutor(AsyncTaskExecutor taskExecutor) {
		this.taskExecutor = taskExecutor;
	}

	public void setAsyncRequestTimeout(long timeout) {
		this.asyncRequestTimeout = timeout;
	}

	public void setCallableInterceptors(List<CallableProcessingInterceptor> interceptors) {
		Assert.notNull(interceptors, "CallableProcessingInterceptor List must not be null");
		this.callableInterceptors = interceptors.toArray(new CallableProcessingInterceptor[interceptors.size()]);
	}

	public void setDeferredResultInterceptors(List<DeferredResultProcessingInterceptor> interceptors) {
		Assert.notNull(interceptors, "DeferredResultProcessingInterceptor List must not be null");
		this.deferredResultInterceptors = interceptors.toArray(new DeferredResultProcessingInterceptor[interceptors.size()]);
	}

	public void setIgnoreDefaultModelOnRedirect(boolean ignoreDefaultModelOnRedirect) {
		this.ignoreDefaultModelOnRedirect = ignoreDefaultModelOnRedirect;
	}

	public void setSessionAttributeStore(SessionAttributeStore sessionAttributeStore) {
		this.sessionAttributeStore = sessionAttributeStore;
	}

	public void setCacheSecondsForSessionAttributeHandlers(int cacheSecondsForSessionAttributeHandlers) {
		this.cacheSecondsForSessionAttributeHandlers = cacheSecondsForSessionAttributeHandlers;
	}

	public void setSynchronizeOnSession(boolean synchronizeOnSession) {
		this.synchronizeOnSession = synchronizeOnSession;
	}

	public void setParameterNameDiscoverer(ParameterNameDiscoverer parameterNameDiscoverer) {
		this.parameterNameDiscoverer = parameterNameDiscoverer;
	}

	@Override
	public void setBeanFactory(BeanFactory beanFactory) {
		if (beanFactory instanceof ConfigurableBeanFactory) {
			this.beanFactory = (ConfigurableBeanFactory) beanFactory;
		}
	}

	protected ConfigurableBeanFactory getBeanFactory() {
		return this.beanFactory;
	}


	@Override
	public void afterPropertiesSet() {
		initControllerAdviceCache();

		if (this.argumentResolvers == null) {
			List<HandlerMethodArgumentResolver> resolvers = getDefaultArgumentResolvers();
			this.argumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
		}
		if (this.initBinderArgumentResolvers == null) {
			List<HandlerMethodArgumentResolver> resolvers = getDefaultInitBinderArgumentResolvers();
			this.initBinderArgumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
		}
		if (this.returnValueHandlers == null) {
			List<HandlerMethodReturnValueHandler> handlers = getDefaultReturnValueHandlers();
			this.returnValueHandlers = new HandlerMethodReturnValueHandlerComposite().addHandlers(handlers);
		}
	}

	private void initControllerAdviceCache() {
		if (getApplicationContext() == null) {
			return;
		}
		if (logger.isInfoEnabled()) {
			logger.info("Looking for @ControllerAdvice: " + getApplicationContext());
		}

		List<ControllerAdviceBean> beans = ControllerAdviceBean.findAnnotatedBeans(getApplicationContext());
		AnnotationAwareOrderComparator.sort(beans);

		List<Object> requestResponseBodyAdviceBeans = new ArrayList<Object>();

		for (ControllerAdviceBean bean : beans) {
			Set<Method> attrMethods = MethodIntrospector.selectMethods(bean.getBeanType(), MODEL_ATTRIBUTE_METHODS);
			if (!attrMethods.isEmpty()) {
				this.modelAttributeAdviceCache.put(bean, attrMethods);
				if (logger.isInfoEnabled()) {
					logger.info("Detected @ModelAttribute methods in " + bean);
				}
			}
			Set<Method> binderMethods = MethodIntrospector.selectMethods(bean.getBeanType(), INIT_BINDER_METHODS);
			if (!binderMethods.isEmpty()) {
				this.initBinderAdviceCache.put(bean, binderMethods);
				if (logger.isInfoEnabled()) {
					logger.info("Detected @InitBinder methods in " + bean);
				}
			}
			if (RequestBodyAdvice.class.isAssignableFrom(bean.getBeanType())) {
				requestResponseBodyAdviceBeans.add(bean);
				if (logger.isInfoEnabled()) {
					logger.info("Detected RequestBodyAdvice bean in " + bean);
				}
			}
			if (ResponseBodyAdvice.class.isAssignableFrom(bean.getBeanType())) {
				requestResponseBodyAdviceBeans.add(bean);
				if (logger.isInfoEnabled()) {
					logger.info("Detected ResponseBodyAdvice bean in " + bean);
				}
			}
		}

		if (!requestResponseBodyAdviceBeans.isEmpty()) {
			this.requestResponseBodyAdvice.addAll(0, requestResponseBodyAdviceBeans);
		}
	}

	private List<HandlerMethodArgumentResolver> getDefaultArgumentResolvers() {
		List<HandlerMethodArgumentResolver> resolvers = new ArrayList<HandlerMethodArgumentResolver>();

		// Annotation-based argument resolution
		resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), false));
		resolvers.add(new RequestParamMapMethodArgumentResolver());
		resolvers.add(new PathVariableMethodArgumentResolver());
		resolvers.add(new PathVariableMapMethodArgumentResolver());
		resolvers.add(new MatrixVariableMethodArgumentResolver());
		resolvers.add(new MatrixVariableMapMethodArgumentResolver());
		resolvers.add(new ServletModelAttributeMethodProcessor(false));
		resolvers.add(new RequestResponseBodyMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
		resolvers.add(new RequestPartMethodArgumentResolver(getMessageConverters(), this.requestResponseBodyAdvice));
		resolvers.add(new RequestHeaderMethodArgumentResolver(getBeanFactory()));
		resolvers.add(new RequestHeaderMapMethodArgumentResolver());
		resolvers.add(new ServletCookieValueMethodArgumentResolver(getBeanFactory()));
		resolvers.add(new ExpressionValueMethodArgumentResolver(getBeanFactory()));
		resolvers.add(new SessionAttributeMethodArgumentResolver());
		resolvers.add(new RequestAttributeMethodArgumentResolver());

		// Type-based argument resolution
		resolvers.add(new ServletRequestMethodArgumentResolver());
		resolvers.add(new ServletResponseMethodArgumentResolver());
		resolvers.add(new HttpEntityMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
		resolvers.add(new RedirectAttributesMethodArgumentResolver());
		resolvers.add(new ModelMethodProcessor());
		resolvers.add(new MapMethodProcessor());
		resolvers.add(new ErrorsMethodArgumentResolver());
		resolvers.add(new SessionStatusMethodArgumentResolver());
		resolvers.add(new UriComponentsBuilderMethodArgumentResolver());

		// Custom arguments
		if (getCustomArgumentResolvers() != null) {
			resolvers.addAll(getCustomArgumentResolvers());
		}

		// Catch-all
		resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), true));
		resolvers.add(new ServletModelAttributeMethodProcessor(true));

		return resolvers;
	}

	private List<HandlerMethodArgumentResolver> getDefaultInitBinderArgumentResolvers() {
		List<HandlerMethodArgumentResolver> resolvers = new ArrayList<HandlerMethodArgumentResolver>();

		// Annotation-based argument resolution
		resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), false));
		resolvers.add(new RequestParamMapMethodArgumentResolver());
		resolvers.add(new PathVariableMethodArgumentResolver());
		resolvers.add(new PathVariableMapMethodArgumentResolver());
		resolvers.add(new MatrixVariableMethodArgumentResolver());
		resolvers.add(new MatrixVariableMapMethodArgumentResolver());
		resolvers.add(new ExpressionValueMethodArgumentResolver(getBeanFactory()));
		resolvers.add(new SessionAttributeMethodArgumentResolver());
		resolvers.add(new RequestAttributeMethodArgumentResolver());

		// Type-based argument resolution
		resolvers.add(new ServletRequestMethodArgumentResolver());
		resolvers.add(new ServletResponseMethodArgumentResolver());

		// Custom arguments
		if (getCustomArgumentResolvers() != null) {
			resolvers.addAll(getCustomArgumentResolvers());
		}

		// Catch-all
		resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), true));

		return resolvers;
	}

	private List<HandlerMethodReturnValueHandler> getDefaultReturnValueHandlers() {
		List<HandlerMethodReturnValueHandler> handlers = new ArrayList<HandlerMethodReturnValueHandler>();

		// Single-purpose return value types
		handlers.add(new ModelAndViewMethodReturnValueHandler());
		handlers.add(new ModelMethodProcessor());
		handlers.add(new ViewMethodReturnValueHandler());
		handlers.add(new ResponseBodyEmitterReturnValueHandler(getMessageConverters()));
		handlers.add(new StreamingResponseBodyReturnValueHandler());
		handlers.add(new HttpEntityMethodProcessor(getMessageConverters(),
				this.contentNegotiationManager, this.requestResponseBodyAdvice));
		handlers.add(new HttpHeadersReturnValueHandler());
		handlers.add(new CallableMethodReturnValueHandler());
		handlers.add(new DeferredResultMethodReturnValueHandler());
		handlers.add(new AsyncTaskMethodReturnValueHandler(this.beanFactory));

		// Annotation-based return value types
		handlers.add(new ModelAttributeMethodProcessor(false));
		handlers.add(new RequestResponseBodyMethodProcessor(getMessageConverters(),
				this.contentNegotiationManager, this.requestResponseBodyAdvice));

		// Multi-purpose return value types
		handlers.add(new ViewNameMethodReturnValueHandler());
		handlers.add(new MapMethodProcessor());

		// Custom return value types
		if (getCustomReturnValueHandlers() != null) {
			handlers.addAll(getCustomReturnValueHandlers());
		}

		// Catch-all
		if (!CollectionUtils.isEmpty(getModelAndViewResolvers())) {
			handlers.add(new ModelAndViewResolverMethodReturnValueHandler(getModelAndViewResolvers()));
		}
		else {
			handlers.add(new ModelAttributeMethodProcessor(true));
		}

		return handlers;
	}

	@Override
	protected boolean supportsInternal(HandlerMethod handlerMethod) {
		return true;
	}

	@Override
	protected ModelAndView handleInternal(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

		ModelAndView mav;
		checkRequest(request);

		// Execute invokeHandlerMethod in synchronized block if required.
		if (this.synchronizeOnSession) {
			HttpSession session = request.getSession(false);
			if (session != null) {
				Object mutex = WebUtils.getSessionMutex(session);
				synchronized (mutex) {
					mav = invokeHandlerMethod(request, response, handlerMethod);
				}
			}
			else {
				// No HttpSession available -> no mutex necessary
				mav = invokeHandlerMethod(request, response, handlerMethod);
			}
		}
		else {
			// No synchronization on session demanded at all...
			mav = invokeHandlerMethod(request, response, handlerMethod);
		}

		if (!response.containsHeader(HEADER_CACHE_CONTROL)) {
			if (getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
				applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
			}
			else {
				prepareResponse(response);
			}
		}

		return mav;
	}

	@Override
	protected long getLastModifiedInternal(HttpServletRequest request, HandlerMethod handlerMethod) {
		return -1;
	}

	private SessionAttributesHandler getSessionAttributesHandler(HandlerMethod handlerMethod) {
		Class<?> handlerType = handlerMethod.getBeanType();
		SessionAttributesHandler sessionAttrHandler = this.sessionAttributesHandlerCache.get(handlerType);
		if (sessionAttrHandler == null) {
			synchronized (this.sessionAttributesHandlerCache) {
				sessionAttrHandler = this.sessionAttributesHandlerCache.get(handlerType);
				if (sessionAttrHandler == null) {
					sessionAttrHandler = new SessionAttributesHandler(handlerType, sessionAttributeStore);
					this.sessionAttributesHandlerCache.put(handlerType, sessionAttrHandler);
				}
			}
		}
		return sessionAttrHandler;
	}

	protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

		ServletWebRequest webRequest = new ServletWebRequest(request, response);
		try {
			WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
			ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);

			ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
			invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
			invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
			invocableMethod.setDataBinderFactory(binderFactory);
			invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);

			ModelAndViewContainer mavContainer = new ModelAndViewContainer();
			mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
			modelFactory.initModel(webRequest, mavContainer, invocableMethod);
			mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);

			AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
			asyncWebRequest.setTimeout(this.asyncRequestTimeout);

			WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
			asyncManager.setTaskExecutor(this.taskExecutor);
			asyncManager.setAsyncWebRequest(asyncWebRequest);
			asyncManager.registerCallableInterceptors(this.callableInterceptors);
			asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);

			if (asyncManager.hasConcurrentResult()) {
				Object result = asyncManager.getConcurrentResult();
				mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
				asyncManager.clearConcurrentResult();
				if (logger.isDebugEnabled()) {
					logger.debug("Found concurrent result value [" + result + "]");
				}
				invocableMethod = invocableMethod.wrapConcurrentResult(result);
			}

			invocableMethod.invokeAndHandle(webRequest, mavContainer);
			if (asyncManager.isConcurrentHandlingStarted()) {
				return null;
			}

			return getModelAndView(mavContainer, modelFactory, webRequest);
		}
		finally {
			webRequest.requestCompleted();
		}
	}

	protected ServletInvocableHandlerMethod createInvocableHandlerMethod(HandlerMethod handlerMethod) {
		return new ServletInvocableHandlerMethod(handlerMethod);
	}

	private ModelFactory getModelFactory(HandlerMethod handlerMethod, WebDataBinderFactory binderFactory) {
		SessionAttributesHandler sessionAttrHandler = getSessionAttributesHandler(handlerMethod);
		Class<?> handlerType = handlerMethod.getBeanType();
		Set<Method> methods = this.modelAttributeCache.get(handlerType);
		if (methods == null) {
			methods = MethodIntrospector.selectMethods(handlerType, MODEL_ATTRIBUTE_METHODS);
			this.modelAttributeCache.put(handlerType, methods);
		}
		List<InvocableHandlerMethod> attrMethods = new ArrayList<InvocableHandlerMethod>();
		// Global methods first
		for (Entry<ControllerAdviceBean, Set<Method>> entry : this.modelAttributeAdviceCache.entrySet()) {
			if (entry.getKey().isApplicableToBeanType(handlerType)) {
				Object bean = entry.getKey().resolveBean();
				for (Method method : entry.getValue()) {
					attrMethods.add(createModelAttributeMethod(binderFactory, bean, method));
				}
			}
		}
		for (Method method : methods) {
			Object bean = handlerMethod.getBean();
			attrMethods.add(createModelAttributeMethod(binderFactory, bean, method));
		}
		return new ModelFactory(attrMethods, binderFactory, sessionAttrHandler);
	}

	private InvocableHandlerMethod createModelAttributeMethod(WebDataBinderFactory factory, Object bean, Method method) {
		InvocableHandlerMethod attrMethod = new InvocableHandlerMethod(bean, method);
		attrMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
		attrMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);
		attrMethod.setDataBinderFactory(factory);
		return attrMethod;
	}

	private WebDataBinderFactory getDataBinderFactory(HandlerMethod handlerMethod) throws Exception {
		Class<?> handlerType = handlerMethod.getBeanType();
		Set<Method> methods = this.initBinderCache.get(handlerType);
		if (methods == null) {
			methods = MethodIntrospector.selectMethods(handlerType, INIT_BINDER_METHODS);
			this.initBinderCache.put(handlerType, methods);
		}
		List<InvocableHandlerMethod> initBinderMethods = new ArrayList<InvocableHandlerMethod>();
		// Global methods first
		for (Entry<ControllerAdviceBean, Set<Method>> entry : this.initBinderAdviceCache.entrySet()) {
			if (entry.getKey().isApplicableToBeanType(handlerType)) {
				Object bean = entry.getKey().resolveBean();
				for (Method method : entry.getValue()) {
					initBinderMethods.add(createInitBinderMethod(bean, method));
				}
			}
		}
		for (Method method : methods) {
			Object bean = handlerMethod.getBean();
			initBinderMethods.add(createInitBinderMethod(bean, method));
		}
		return createDataBinderFactory(initBinderMethods);
	}

	private InvocableHandlerMethod createInitBinderMethod(Object bean, Method method) {
		InvocableHandlerMethod binderMethod = new InvocableHandlerMethod(bean, method);
		binderMethod.setHandlerMethodArgumentResolvers(this.initBinderArgumentResolvers);
		binderMethod.setDataBinderFactory(new DefaultDataBinderFactory(this.webBindingInitializer));
		binderMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);
		return binderMethod;
	}

	protected InitBinderDataBinderFactory createDataBinderFactory(List<InvocableHandlerMethod> binderMethods)
			throws Exception {

		return new ServletRequestDataBinderFactory(binderMethods, getWebBindingInitializer());
	}

	private ModelAndView getModelAndView(ModelAndViewContainer mavContainer,
			ModelFactory modelFactory, NativeWebRequest webRequest) throws Exception {

		modelFactory.updateModel(webRequest, mavContainer);
		if (mavContainer.isRequestHandled()) {
			return null;
		}
		ModelMap model = mavContainer.getModel();
		ModelAndView mav = new ModelAndView(mavContainer.getViewName(), model, mavContainer.getStatus());
		if (!mavContainer.isViewReference()) {
			mav.setView((View) mavContainer.getView());
		}
		if (model instanceof RedirectAttributes) {
			Map<String, ?> flashAttributes = ((RedirectAttributes) model).getFlashAttributes();
			HttpServletRequest request = webRequest.getNativeRequest(HttpServletRequest.class);
			RequestContextUtils.getOutputFlashMap(request).putAll(flashAttributes);
		}
		return mav;
	}

}

```

### HttpRequestHandlerAdapter.java

```java
package org.springframework.web.servlet.mvc;
/**
 * 支持HttpRequestHandler的处理器映射器
 */
public class HttpRequestHandlerAdapter implements HandlerAdapter {

	@Override
	public boolean supports(Object handler) {
		return (handler instanceof HttpRequestHandler);
	}

	@Override
	public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		((HttpRequestHandler) handler).handleRequest(request, response);
		return null;
	}

	@Override
	public long getLastModified(HttpServletRequest request, Object handler) {
		if (handler instanceof LastModified) {
			return ((LastModified) handler).getLastModified(request);
		}
		return -1L;
	}

}

```

### SimpleControllerHandlerAdapter.java

```java
package org.springframework.web.servlet.mvc;
/**
 * 支持Controller的处理器适配器
 */
public class SimpleControllerHandlerAdapter implements HandlerAdapter {

	@Override
	public boolean supports(Object handler) {
		return (handler instanceof Controller);
	}

	@Override
	public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		return ((Controller) handler).handleRequest(request, response);
	}

	@Override
	public long getLastModified(HttpServletRequest request, Object handler) {
		if (handler instanceof LastModified) {
			return ((LastModified) handler).getLastModified(request);
		}
		return -1L;
	}

}

```

### SimpleServletHandlerAdapter.java

```java
package org.springframework.web.servlet.handler;
/**
 * 支持Servlet的处理器适配器
 */
public class SimpleServletHandlerAdapter implements HandlerAdapter {

	@Override
	public boolean supports(Object handler) {
		return (handler instanceof Servlet);
	}

	@Override
	public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		((Servlet) handler).service(request, response);
		return null;
	}

	@Override
	public long getLastModified(HttpServletRequest request, Object handler) {
		return -1;
	}

}

```





# END
springMVC找Controller的流程



扫描整个项目，定义Map集合

拿到所有加了@Controller注解的类gethandle

遍历类里面所有方法对象

判断方法是否加了@RequestMapping注解

把@RequestMapping注解的value，作为map集合的Key给put进去

根据用户发送的请求 拿到请求的URI（URL是全部的）

使用请求的uri作为map的key 去map里面get 看看是否有返回值







controller 定义的方式 有2种类型和三种实现

spring提供了两种HandlerMapping以及三种HandlerAdapter.

这边从他的入口开始开始讲起来javaweb的内容

![DispatcherServlet](D:\制作地点\DispatcherServlet.png)

他们继承了HttpServlet doget（），dopost()都是调用processRequest（）方法没有去处理请求的类型的这边点击这个方法，

而这个processRequest（）没有对请求进行核心的处理，然后调用doService()方法。

doService()方法是一个抽象的方法，实现就是在DispatcherServlet类中

```
public abstract class HttpServlet extends GenericServlet{
...
doGet();
doPost();
...
}
```

```
public abstract class HttpServletBean extends HttpServlet implements  EnvironmentCapable, EnvironmentAware {...}
```

```
public abstract class FrameworkServlet extends HttpServletBean implements ApplicationContextAware {
...
...
//这个还是没有实现
protected abstract void doService(HttpServletRequest request, HttpServletResponse response)
			throws Exception;


//1
@Override
	protected final void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		processRequest(request, response);
	}
	
@Override
	protected final void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		processRequest(request, response);
	}
	
	
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		long startTime = System.currentTimeMillis();
		Throwable failureCause = null;

		LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
		LocaleContext localeContext = buildLocaleContext(request);

		RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
		ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
		asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

		initContextHolders(request, localeContext, requestAttributes);

		try {
			doService(request, response);
		}
		catch (ServletException | IOException ex) {
			failureCause = ex;
			throw ex;
		}
		catch (Throwable ex) {
			failureCause = ex;
			throw new NestedServletException("Request processing failed", ex);
		}

		finally {
			resetContextHolders(request, previousLocaleContext, previousAttributes);
			if (requestAttributes != null) {
				requestAttributes.requestCompleted();
			}
			logResult(request, response, failureCause, asyncManager);
			publishRequestHandledEvent(request, response, startTime, failureCause);
		}
	}
	...
	...
}
```



doService（）调用了doDispatch（）方法，所以这边就是入口，开始真正处理逻辑的时候了，寻找controller

```
//1.检查文件上传的请求，这边是一个http的规范里面
processedRequest = checkMultipart(request);

//取得处理当前请求的controller，这边点进去
mappedHandler = getHandler(processedRequest);
```

//DispatcherServlet

```
public class DispatcherServlet extends FrameworkServlet {


@Override
	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		logRequest(request);

		// Keep a snapshot of the request attributes in case of an include,
		// to be able to restore the original attributes after the include.
		Map<String, Object> attributesSnapshot = null;
		if (WebUtils.isIncludeRequest(request)) {
			attributesSnapshot = new HashMap<>();
			Enumeration<?> attrNames = request.getAttributeNames();
			while (attrNames.hasMoreElements()) {
				String attrName = (String) attrNames.nextElement();
				if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
					attributesSnapshot.put(attrName, request.getAttribute(attrName));
				}
			}
		}

		// Make framework objects available to handlers and view objects.
		request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
		request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
		request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
		request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

		if (this.flashMapManager != null) {
			FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
			if (inputFlashMap != null) {
				request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
			}
			request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
			request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
		}

		try {
			doDispatch(request, response);
		}
		finally {
			if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
				// Restore the original attribute snapshot, in case of an include.
				if (attributesSnapshot != null) {
					restoreAttributesAfterInclude(request, attributesSnapshot);
				}
			}
		}
	}


//2.

	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {

		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
			//1.检查文件上传的请求，这边是一个http的规范里面
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.
				//取得处理当前请求的controller，这边点进去
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}




//3.

	@Nullable
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
	//映射controller的主键,这样我
		if (this.handlerMappings != null) {
			for (HandlerMapping mapping : this.handlerMappings) {
				HandlerExecutionChain handler = mapping.getHandler(request);
				//这边有两种定义的方式HttpRequestHandler
				//这边可以通过BeanNameUrlHandlerMappiing
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}




/** List of HandlerMappings used by this servlet. */
@Nullable
private List<HandlerMapping> handlerMappings;


}
```

![image-20200527000114987](C:\Users\drew\AppData\Roaming\Typora\typora-user-images\image-20200527000114987.png)

```
//映射controller的主键,这样我们可以通过这个handlerMappings来找我们的controller，
		if (this.handlerMappings != null) {}
//	RequestMappingHandler 这边在点进去到 mapping.getHandler(request);	
//1.ctrl+alt出现两种
HandlerExecutionChain handler = mapping.getHandler(request);
```

2.Object handler = getHandlerInternal(request);点击这里

3.找到抽象方法ctrl+alt找出实现类

```

@Override
	protected HandlerMethod getHandlerInternal(HttpServletRequest request) throws Exception {
	//帮助你去获取请求路径
		String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
		request.setAttribute(LOOKUP_PATH, lookupPath);
		this.mappingRegistry.acquireReadLock();
		try {
		//4.这边调用了这个方法
			HandlerMethod handlerMethod = lookupHandlerMethod(lookupPath, request);
			return (handlerMethod != null ? handlerMethod.createWithResolvedBean() : null);
		}
		finally {
			this.mappingRegistry.releaseReadLock();
		}
	}

```

```
//5.
List<T>directPathMatches=this.mappingRegistry.getMappingsByUrl(lookupPath);
```

```

通过这种方式拿到uri的
@Nullable
public List<T> getMappingsByUrl(String urlPath) {   
	return this.urlLookup.get(urlPath);
}
```

// AbstractHandlerMapping

```
public abstract class AbstractHandlerMapping extends WebApplicationObjectSupport
		implements HandlerMapping, Ordered, BeanNameAware {
	
    //
	@Nullable
	protected abstract Object getHandlerInternal(HttpServletRequest request) throws Exception;
	
		
		
		
		

	@Override
	@Nullable
	public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		//这边
		Object handler = getHandlerInternal(request);
		if (handler == null) {
			handler = getDefaultHandler();
		}
		if (handler == null) {
			return null;
		}
		// Bean name or resolved handler?
		if (handler instanceof String) {
			String handlerName = (String) handler;
			handler = obtainApplicationContext().getBean(handlerName);
		}

		HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);

		if (logger.isTraceEnabled()) {
			logger.trace("Mapped to " + handler);
		}
		else if (logger.isDebugEnabled() && !request.getDispatcherType().equals(DispatcherType.ASYNC)) {
			logger.debug("Mapped to " + executionChain.getHandler());
		}

		if (hasCorsConfigurationSource(handler)) {
			CorsConfiguration config = (this.corsConfigurationSource != null ? this.corsConfigurationSource.getCorsConfiguration(request) : null);
			CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
			config = (config != null ? config.combine(handlerConfig) : handlerConfig);
			executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
		}

		return executionChain;
	}





@Override
	protected HandlerMethod getHandlerInternal(HttpServletRequest request) throws Exception {
	//帮助你去获取请求路径
		String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
		request.setAttribute(LOOKUP_PATH, lookupPath);
		this.mappingRegistry.acquireReadLock();
		try {
			HandlerMethod handlerMethod = lookupHandlerMethod(lookupPath, request);
			return (handlerMethod != null ? handlerMethod.createWithResolvedBean() : null);
		}
		finally {
			this.mappingRegistry.releaseReadLock();
		}
	}



}
```



这边开始讲配置器了

三种实现 if (this.handlerAdapters != null) 这边出现三种实现

if (adapter.supports(handler)) //这边supports

![image-20200526231347770](C:\Users\drew\AppData\Roaming\Typora\typora-user-images\image-20200526231347770.png)

1.AbstractHandlerMethodAdapter(使用注解的方法)

```
	@Override
	public final boolean supports(Object handler) {
		return (handler instanceof HandlerMethod && supportsInternal((HandlerMethod) handler));
	}
```

2.HttpRequestHandlerAdapter（）使用implements

```
public class HttpRequestHandlerAdapter implements HandlerAdapter {

	@Override
	public boolean supports(Object handler) {
		return (handler instanceof HttpRequestHandler);
	}
	...
	...
}
```



当加的是@Controller注解的时候就是HandlerMethod

//DispatcherServlet

```
	// Determine handler adapter for the current request.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

```



这边是点击

HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

```
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
		if (this.handlerAdapters != null) {
			for (HandlerAdapter adapter : this.handlerAdapters) {
				if (adapter.supports(handler)) {
					return adapter;
				}
			}
		}
		throw new ServletException("No adapter for handler [" + handler +
				"]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
	}
```



```
// Actually invoke the 
handler.mv = ha.handle(processedRequest, response,mappedHandler.getHandler());
```



//AbstractHandlerMethodAdapter

```
	@Override
	@Nullable
	public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
//返回了这个handleInternal ，找到他的实现位置
		return handleInternal(request, response, (HandlerMethod) handler);
	}

```

ctrl+alt

RequestMappingHandlerAdapter

```
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

```

这边调用了这个,点进去

```
// No HttpSession available -> no mutex necessary
mav = invokeHandlerMethod(request, response, handlerMethod);
```



这边只是初始化你的请求而已，读取配置而已

//重点是这里
			invocableMethod.**invokeAndHandle**(webRequest, mavContainer);点击这个方法

```
@Nullable
	protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

		ServletWebRequest webRequest = new ServletWebRequest(request, response);
		try {
			WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
			ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);

			ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
			if (this.argumentResolvers != null) {
				invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
			}
			if (this.returnValueHandlers != null) {
				invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
			}
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
				LogFormatUtils.traceDebug(logger, traceOn -> {
					String formatted = LogFormatUtils.formatValue(result, !traceOn);
					return "Resume with async result [" + formatted + "]";
				});
				invocableMethod = invocableMethod.wrapConcurrentResult(result);
			}
//重点是这里
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


```

//重点是这里
			invocableMethod.**invokeAndHandle**(webRequest, mavContainer);点击这个方法



```
package org.springframework.web.servlet.mvc.method.annotation;
```

然后点击	

Object returnValue = **invokeForRequest**(webRequest, mavContainer, providedArgs);

ServletInvocableHandlerMethod

```
public void  invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

		Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
		setResponseStatus(webRequest);

		if (returnValue == null) {
			if (isRequestNotModified(webRequest) || getResponseStatus() != null || mavContainer.isRequestHandled()) {
				disableContentCachingIfNecessary(webRequest);
				mavContainer.setRequestHandled(true);
				return;
			}
		}
		else if (StringUtils.hasText(getResponseStatusReason())) {
			mavContainer.setRequestHandled(true);
			return;
		}

		mavContainer.setRequestHandled(false);
		Assert.state(this.returnValueHandlers != null, "No return value handlers");
		try {
			this.returnValueHandlers.handleReturnValue(
					returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
		}
		catch (Exception ex) {
			if (logger.isTraceEnabled()) {
				logger.trace(formatErrorForReturnValue(returnValue), ex);
			}
			throw ex;
		}
	}
```

这边就是通过反射调用的方法

传的参数可能是个对象就使用映射来

--》			args[i] = this.resolvers.resolveArgument不同的对象有不同的对象去处理

InvocableHandlerMethod

```
	@Nullable
	public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

	--》	Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
		if (logger.isTraceEnabled()) {
			logger.trace("Arguments: " + Arrays.toString(args));
		}
	--》	return doInvoke(args);
	}
	
	
	
	//参数的处理，传递参数的处理
	
	protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

//定义了实参数组 然后将传到map里面去
		MethodParameter[] parameters = getMethodParameters();
		if (ObjectUtils.isEmpty(parameters)) {
			return EMPTY_ARGS;
		}

		Object[] args = new Object[parameters.length];
		for (int i = 0; i < parameters.length; i++) {
			MethodParameter parameter = parameters[i];
			parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
			args[i] = findProvidedArgument(parameter, providedArgs);
			if (args[i] != null) {
				continue;
			}
	--》		if (!this.resolvers.supportsParameter(parameter)) {
				throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
			}
			try {
	--》			args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
			}
			catch (Exception ex) {
				// Leave stack trace for later, exception may actually be resolved and handled...
				if (logger.isDebugEnabled()) {
					String exMsg = ex.getMessage();
					if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
						logger.debug(formatArgumentError(parameter, exMsg));
					}
				}
				throw ex;
			}
		}
		return args;
	}
```



HandlerMethodArgumentResolver

HandlerMethodArgumentResolverComposite

```
@Nullable
	private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
	//如果找到了就返回
		HandlerMethodArgumentResolver result = this.argumentResolverCache.get(parameter);
		if (result == null) {
			for (HandlerMethodArgumentResolver resolver : this.argumentResolvers) {
			//判断这个参数是否需要处理
				if (resolver.supportsParameter(parameter)) {
					result = resolver;
					this.argumentResolverCache.put(parameter, result);
					break;
				}
			}
		}
		return result;
	}

```



```
public interface HandlerMethodArgumentResolver {
//判断这个参数需不需要去处理
	boolean supportsParameter(MethodParameter parameter);

//给你这个参数去赋值，返回的值最终就是springmvc返回的值
	@Nullable
	Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;

}

```




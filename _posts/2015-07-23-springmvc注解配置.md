---
layout:default
title:spring mvc代码配置
comments:false
category: java
---

##注解方式配置
```java
@Configuration //标注此类为配置类
@EnableWebMvc
@PropertySource("classpath:service.properties") //加载配置文件
@Import({ExceptionControllerAdvice.class, SecurityConfig.class}) //扫描单个类文件
//扫描包
@ComponentScan(basePackages = {"com.learninggenie.api.controller",
        "com.learninggenie.api.service"
        })
public class MVCConfig extends WebMvcConfigurerAdapter {
    @Autowired
    Environment env;
    @Autowired
    SimpleClientHttpRequestFactory httpClientFactory;

    //注入一个对象
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }

    @Bean(name = "viewResolver")
    public InternalResourceViewResolver getViewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/");
        viewResolver.setSuffix(".jsp");
        return viewResolver; 
    }

    @Bean(name = "httpClientFactory")
    public SimpleClientHttpRequestFactory getSimpleClientHttpRequestFactory() {
        SimpleClientHttpRequestFactory scrf = new SimpleClientHttpRequestFactory();
        scrf.setConnectTimeout(8000);
        scrf.setReadTimeout(8000);
        return scrf;
    }

    @Bean(name = "restTemplate")
    @DependsOn("httpClientFactory")
    public RestTemplate getRestTemplate() {
        return new RestTemplate(httpClientFactory);
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/index.html")
                .addResourceLocations("/");
    }

    @Bean(name = "documentationController")
    public JSONDocController docController() {
        JSONDocController jsonDocController = new JSONDocController("1.0", "http://localhost:8080/api", Arrays.asList("com.learninggenie.api.controller"));
        jsonDocController.setPlaygroundEnabled(true);
        jsonDocController.setDisplayMethodAs(JSONDoc.MethodDisplay.URI);
        return jsonDocController;
    }

    @Bean(name="simpleMappingExceptionResolver")
    public SimpleMappingExceptionResolver simpleMappingExceptionResolver() {
        env.getProperty("prop.common");
        SimpleMappingExceptionResolver r =
                new SimpleMappingExceptionResolver();

        Properties mappings = new Properties();
        mappings.setProperty("DatabaseException", "databaseError");
        mappings.setProperty("InvalidCreditCardException", "creditCardError");

        r.setExceptionMappings(mappings);  // None by default
        r.setDefaultErrorView("error");    // No default
        r.setExceptionAttribute("ex");     // Default is "exception"
        r.setWarnLogCategory("example.MvcLogger");     // No default
        return r;
    }
}
```
##注册配置类
```java
public class WebAppInitializer implements WebApplicationInitializer {
	@Override
	public void onStartup(ServletContext container) throws ServletException {
		log.info("starting web application initialization");

		// Create the 'root' Spring application context
		AnnotationConfigWebApplicationContext rootContext = new AnnotationConfigWebApplicationContext();
		rootContext.register(MVCConfig.class);

		// Manage the lifecycle of the root application context
		container.addListener(new ContextLoaderListener(rootContext));

		ResourcePropertySource props;
		try {
			props = new ResourcePropertySource("classpath:service.properties");
		} catch (IOException ex) {
			log.error("could not load properties file to use for environment configuration", ex);
			throw new ServletException(ex);
		}
		log.info("web application initialization properties:" + props.getSource());

		// Register and map the dispatcher servlet
		ServletRegistration.Dynamic dispatcher = container.addServlet(
				"dispatcher", new DispatcherServlet(rootContext));
		dispatcher.setLoadOnStartup(1);
		dispatcher.addMapping("/");
    FilterRegistration.Dynamic filterRegistration=container.addFilter("filterRegistration",new SimpleCORSFilter());
    filterRegistration.setAsyncSupported(true);
    filterRegistration.addMappingForUrlPatterns(EnumSet.allOf(DispatcherType.class), false, "/*");
    }

```

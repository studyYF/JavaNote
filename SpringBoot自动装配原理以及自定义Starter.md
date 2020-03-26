> SpringBoot的核心思想是约定优于配置，它简化了之前使用SpringMVC时候的大量配置xml，使得开发者能够快速的创建一个Web项目。那么SpringBoot是如何做到的呢？

#### @SpringBootApplication

当我们创建一个`SpringBoot`项目完成后，会有一个启动类，直接就可以运行`web`项目了。所以我们首先从这个启动类的注解上出发，看看`SpringBoot`是如何实现的。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
  //...
}
```

可以看到`@SpringBootApplication`主要是三个注解的复合注解。

##### @SpringBootConfiguration

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}
```

这个注解最简单，它是对`@Configuration`的封装，`@Configuration`我们最熟悉不过了，这里就不做分析了。

##### @ComponentScan

这个注解的作用主要是扫描定义的包下的所有的包含`@Controller`、`@Service`、`@Component`、`@Repository`等注解的类，把他们注册到`Spring`的容器中。具体是如何扫描，如何加载注解信息、如何生成`Bean`以及如何注册到`Spring`容器中，这里的逻辑相对来说比较复杂，不是本文的重点，不具体分析了。

##### @EnableAutoConfiguration

`EnableAutoConfiguration`这个注解就是比较核心的了，实现自动装配就是依赖这个注解，下面我们一步一步来看是如何实现的。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

可以看到EnableAutoConfiguration注解中，主要依赖两个注解，`AutoConfigurationPackage`和`Import(AutoConfigurationImportSelector.class)`，这两个注解的作用都是根据条件动态的加载`Bean`到`Spring`容器中，下面具体分析。

##### @AutoConfigurationPackage

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {

}
```

这里主要是`@Import(AutoConfigurationPackages.Registrar.class)`注解，`Import`注解一定很熟悉了，主要是将`import`的类注入`Spring`容器中，下面具体分析`AutoConfigurationPackages.Registrar`。

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

  	//重写这个方法，根据AnnotationMetadata将bean注册到spring容器中
  	//这里的AnnotationMetadata就是@SpringBootApplication注解的元数据
		@Override
		public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
      //new PackageImport(metadata).getPackageName()返回的是SpringBootApplication注解对应的包名
      //也就是启动类所在的包名，所以，SpringBoot项目的启动类和包名是有一定的要求的，这也是SpringBoot约定大约配置
      //的一种体现
			register(registry, new PackageImport(metadata).getPackageName());
		}

		@Override
		public Set<Object> determineImports(AnnotationMetadata metadata) {
			return Collections.singleton(new PackageImport(metadata));
		}

	}
```

##### AutoConfigurationImportSelector

下面要分析的这个类就是整个自动装配最关键的类了。查看源码可以知道，`AutoConfigurationImportSelector`实现了`ImportSelector`接口，`ImportSelector`接口中的`selectImports`方法会根据返回的`String[]`数组，然后`Spring`根据数组中的类的全路径类名把响应的类注入到`Spring`容器中。接着我们来看一下返回了哪些类。

```java
@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
    //加载元数据，这里面主要是一些Condition条件，目的是为了根据条件判断是否需要注入某个类
		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader);
    //加载所有自动装载的元素
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
				annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}
```

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata,
			AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
  	//根据注解获取注解的属性
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
  	//使用SpringFactoryLoader加载classpath下所有的META-INF/spring.factories中，key是			
  	//org.springframework.boot.autoconfigure.EnableAutoConfiguration的值
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
  	//删除重复的类
		configurations = removeDuplicates(configurations);
  	//删除被排除的类
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
  	//根据上面loadMetadata方法加载的condition条件信息，过滤掉不符合条件的类
		configurations = filter(configurations, autoConfigurationMetadata);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}
```

**注意：**这里`getCandidateConfigurations`方法是重点，`SpringBoot`中依赖的所有的starter都是基于此实现自动装配的。这里用到了`SPI`。

> SPI全称为Service Provier Interface，是一种服务发现机制。SPI的本质是将接口实现类的全限定名配置在文件中，并由服务加载器读取配置文件，加载实现类。

`SpringBoot`的各种`starter`依赖都是基于此实现的，每个maven依赖的`starter`的包下都会有一个`META-INF/spring.factories`配置文件，里面都会有`org.springframework.boot.autoconfigure.EnableAutoConfiguration`键和对应的需要自动加载的类的全限定名。

#### 实现一个Starter

根据上面的分析，下面简单实现一个`starter`。

用`IDEA`创建一个简单的`Maven`项目，然后创建相应的包名和类。

![image-20200326133419432](/Users/yangf/Personal/Note/image-20200326133419432.png)



简单说明一下这个`starter`中的作用

- `FormatTemplate`类提供一个模版方法`doFormat`，可以将传入的泛型对象输出一个字符串

```java
public class FormatTemplate {
    private FormatProcessor formatProcessor;

    public FormatTemplate(FormatProcessor formatProcessor) {
        this.formatProcessor = formatProcessor;
    }
    public <T>String doFormat(T data) {
        return formatProcessor.format(data);
    }
}
```

- `FormatProcessor`是一个接口，提供了一个`format`的方法，它有两个实现，`StringFormatProcessor`直接返回传入对象的`toString`，`JsonFormatProcessor`根据用`fastjson`将传入的对象转成`json`字符串。

```java
public class JsonFormatProcessor implements FormatProcessor {
    @Override
    public <T> String format(T data) {
        return JSON.toJSONString(data);
    }
}
public class StringFormatProcessor implements FormatProcessor {
    @Override
    public <T> String format(T data) {
        return data.toString();
    }
}
```

- `FormatAutoConfiguration`利用`@Configuration`分别将JsonFormatProcessor和StringFormatProcessor注入到spring容器中，这里用了`@Condition`条件注解，只有当项目中引用了`fastjson`的时候，才会注入`JsonFormatProcessor`

```java
@Configuration
public class FormatAutoConfiguration {

    @Bean
    @Primary
    @ConditionalOnClass(name = "com.alibaba.fastjson.JSON")
    public FormatProcessor jsonFormat(){
        return new JsonFormatProcessor();
    }

    @Bean
    @ConditionalOnMissingClass("com.alibaba.fastjson.JSON")
    public FormatProcessor stringFormat(){
        return new StringFormatProcessor();
    }

}
```

- `TemplateAutoConfiguration`引用`FormatAutoConfiguration`并注入了一个`FormatTemplate`。

```java
@Configuration
@Import(FormatAutoConfiguration.class)
public class TemplateAutoConfiguration {

    @Bean
    public FormatTemplate formatTemplate(FormatProcessor formatProcessor) {
        return new FormatTemplate(formatProcessor);
    }

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(TemplateAutoConfiguration.class);
        FormatTemplate bean = context.getBean(FormatTemplate.class);
        FormatProcessor formatProcessor = context.getBean(FormatProcessor.class);
        System.out.printf(bean.doFormat("aaa"));
        System.out.println(formatProcessor.format("bbb"));
    }
}
```

- `resources/META-INF`下的`spring.factories`中定义了要自动装配的类的全路径，即`TemplateAutoConfiguration`

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.cross.springbootdemo.autoconfiguration.TemplateAutoConfiguration
```

编写好后，将项目进行打包，然后在其他项目中，就可以引入了，使用的时候直接可以用`@Autowired`引入`FormatTemplate`了。

```java
@RestController
public class TestController {

    @Autowired
    private FormatTemplate formatTemplate;

    @GetMapping(value = "test")
    public String test() {
        User user = new User();
        user.setName("crossyf---");
        user.setAge(18);
        return formatTemplate.doFormat(user);
    }
}
```

**一个简单的starter就完成了。**


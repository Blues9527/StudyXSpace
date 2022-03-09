[toc]

# Koin

## 什么是Koin
> 是一个kotlin语言写的轻量级依赖注入库，可以支持java

### 什么是依赖注入
> 依赖注入，是指程序运行过程中，如果需要调用另一个对象协助时，无须在代码中创建被调用者，而是依赖于外部的注入

### 使用依赖注入的好处
- 解耦
- 减少重复代码

## Koin的简单使用

- 初始化koin
> 在application中的onCreate()进行初始化
```kotlin
startKoin {
    //设置日志级别
    androidLogger()
    //设置全局context
    androidContext(this@KoinExampleApplication)
    //注入modules
    modules(listOf(viewModelModule))
}
```

- 注入module

```kotlin
//注入view model
val viewModelModule = module {
    
    //这里只做演示，使用场景不一定是对的！！！
    
    //工厂模式注入
    factory { WeatherViewModel(get()) }
    
    //单例模式注入
    single { WeatherViewModel(get()) }
    
    //带参数工厂注入
    factory { (weatherRepo: WeatherRepository)->WeatherViewModel(weatherRepo) }
    
    //带参数单例注入
    single { (weatherRepo: WeatherRepository)->WeatherViewModel(weatherRepo) }
    
    //scope形式注入
    scope(named("weather")) {
        scoped {
            WeatherViewModel(get())
        }
        
        factory {
            WeatherViewModel(get())
        }
        
        //❌ scope内部能使用single注入
        //single {
        //    WeatherViewModel(get())
        //}
    }
    
    //bind形式，绑定父类或者接口
    factory{ User("Blues") } bind AbsUser::class bind Gamer::class
}

class WeatherViewModel(
    private val weatherRepo: WeatherRepository
) : ViewModel() {

    //...
}
```

- 使用已注入module

```kotlin

class WeatherFragment : Fragment() {

    //无参
    private val exampleViewModel: WeatherViewModel by viewModel()
    
    //有参
    //private val exampleViewModel2 : WeatherViewModel by viewModel(){
    //    parametersOf(WeatherRepository())
    //}
}
```

## Koin的实现原理

### Koin是怎么注入的？

#### 通过single
> 顾名思义，每次获取通过single注入的实例都是同一个

```kotlin
inline fun <reified T> single(
    qualifier: Qualifier? = null,
    createdAtStart: Boolean = false,
    noinline definition: Definition<T>
): Pair<Module, InstanceFactory<T>> {

    //single默认在root scope，scope相当于作用域
    val def = createDefinition(Kind.Singleton, qualifier, definition, scopeQualifier = rootScopeQualifier)
    val mapping = indexKey(def.primaryType, qualifier, rootScopeQualifier)
    val instanceFactory = SingleInstanceFactory(def)
    saveMapping(mapping, instanceFactory)
    if (createdAtStart || this.createdAtStart) {
        eagerInstances.add(instanceFactory)
    }
    return Pair(this, instanceFactory)
}
```

#### 通过factory
> 顾名思义:工厂，每次获取通过factory注入的实例都是新的

```
internal inline fun <reified T> factory(
    qualifier: Qualifier? = null,
    noinline definition: Definition<T>,
    scopeQualifier: Qualifier
): Pair<Module, InstanceFactory<T>> {

    //factory可以指定scope
    val def = createDefinition(Kind.Factory, qualifier, definition, scopeQualifier = scopeQualifier)
    val mapping = indexKey(def.primaryType, qualifier, scopeQualifier)
    val instanceFactory = FactoryInstanceFactory(def)
    saveMapping(mapping, instanceFactory)
    return Pair(this, instanceFactory)
}
```

#### 通过scoped
> 

```kotlin
fun scope(qualifier: Qualifier, scopeSet: ScopeDSL.() -> Unit) {
    ScopeDSL(qualifier, this).apply(scopeSet)
    scopes.add(qualifier)
}


inline fun <reified T> scoped(
    qualifier: Qualifier? = null,
    noinline definition: Definition<T>
): Pair<Module, InstanceFactory<T>> {
    val def = createDefinition(Kind.Scoped, qualifier, definition, scopeQualifier = scopeQualifier)
    val mapping = indexKey(def.primaryType, qualifier, scopeQualifier)
    val instanceFactory = ScopedInstanceFactory(def)
    module.saveMapping(mapping, instanceFactory)
    return Pair(module, instanceFactory)
}
```

#### 具体流程
- createDefinition()
> 构建一个BeanDefinition
```
typealias IndexKey = String
typealias Definition<T> = Scope.(ParametersHolder) -> T

inline fun <reified T> createDefinition(
    kind: Kind = Kind.Singleton,//类型：Singleton, Factory, Scoped
    qualifier: Qualifier? = null,//限定符：StringQualifier, TypeQualifier
    noinline definition: Definition<T>,//Scope.(ParametersHolder) -> T
    secondaryTypes: List<KClass<*>> = emptyList(),
    scopeQualifier: Qualifier //scope限定符
): BeanDefinition<T> {
    return BeanDefinition(
        scopeQualifier,
        T::class,
        qualifier,
        definition,
        kind,
        secondaryTypes = secondaryTypes
    )
}
```

- indexKey()
> 生成一个key
```kotlin
fun indexKey(clazz: KClass<*>, typeQualifier: Qualifier?, scopeQualifier: Qualifier): String {
    val tq = typeQualifier?.value ?: ""
    return "${clazz.getFullName()}:$tq:$scopeQualifier"
}
```

- 构建一个InstanceFactory

- saveMapping()
> 放入map，key是String，value是InstanceFactory
```kotlin
@PublishedApi
internal fun saveMapping(mapping: IndexKey, factory: InstanceFactory<*>, allowOverride : Boolean = false) {
    //不能覆盖 && map已存在
    if (!allowOverride && mappings.contains(mapping)) {
        //抛异常
        overrideError(factory, mapping)
    }
    //放入map
    mappings[mapping] = factory
}
```

#### 注入

- KoinApplication.modules()
```kotlin
fun modules(modules: List<Module>): KoinApplication {
    if (koin.logger.isAt(Level.INFO)) {
        val duration = measureDuration {
            loadModules(modules)
        }
        val count = koin.instanceRegistry.size()
        koin.logger.info("loaded $count definitions - $duration ms")
    } else {
        loadModules(modules)
    }
    return this
}
```
- KoinApplication.loadModules()
> 内部调用koin实例的loadModules()
```kotlin
private fun loadModules(modules: List<Module>) {
    //这里allowOverride默认是true
    koin.loadModules(modules, allowOverride = allowOverride)
}
```

- Koin.loadModules()

```
fun loadModules(modules: List<Module>, allowOverride : Boolean = true) {
    //注入
    instanceRegistry.loadModules(modules, allowOverride)
    scopeRegistry.loadScopes(modules)
    createEagerInstances()
}
```

- InstanceRegistry.loadModules()

```
internal fun loadModules(modules: List<Module>, allowOverride: Boolean) {
    modules.forEach { module ->
        loadModule(module, allowOverride)
        eagerInstances.addAll(module.eagerInstances)
    }
}
```

- InstanceRegistry.loadModule()

```kotlin
private fun loadModule(module: Module, allowOverride: Boolean) {
    //这里将module里的mapping遍历，然后放入InstanceRegistry的map里，
    module.mappings.forEach { (mapping, factory) ->
        saveMapping(allowOverride, mapping, factory)
    }
}
```

- saveMapping()

```kotlin
@KoinInternalApi
fun saveMapping(
    allowOverride: Boolean,
    mapping: IndexKey,
    factory: InstanceFactory<*>,
    logWarning: Boolean = true
) {
    if (_instances.containsKey(mapping)) {
        if (!allowOverride) {
            overrideError(factory, mapping)
        } else {
            if (logWarning) _koin.logger.info("Override Mapping '$mapping' with ${factory.beanDefinition}")
        }
    }
    //...
    
    //放入map
    _instances[mapping] = factory
}
```


### Koin是如何获取已注入实例的？
> 利用kotlin的 by 特性

#### 直接调用 get()

##### 举个例子
```kotlin
val networkModule = module {
    //如果okhttp client已注入过，直接get()获取okhttp client的实例
    //这里注入了Retrofit的实例
    single { provideRetrofit(get()) }
    
    //仅供演示
    //single { (okHttpClient: OkHttpClient) -> provideRetrofit(okHttpClient)}
}

fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
    return Retrofit.Builder()
        .baseUrl("https://www.baidu.com")
        .client(okHttpClient)
        .addConverterFactory(GsonConverterFactory.create())
        .build()
}

or 

class MainActivity:AppComapctActivity(){
    private val retrofit:Retrofit = get()
    
    //private val retrofit:Retrofit = get{
    //    parametersOf(OkHttpBuilder.buid())
    //}
}
```

##### 源码
- Scope.get()
    
```kotlin
inline fun <reified T : Any> get(
    qualifier: Qualifier? = null,
    noinline parameters: ParametersDefinition? = null
): T {
    return get(T::class, qualifier, parameters)
}

fun <T> get(
    clazz: KClass<*>,
    qualifier: Qualifier? = null,
    parameters: ParametersDefinition? = null
): T {
    //...
    resolveInstance(qualifier, clazz, parameters)
}
```
- Scope.resolveInstance()
```kotlin
private fun <T> resolveInstance(
    qualifier: Qualifier?,
    clazz: KClass<*>,
    parameterDef: ParametersDefinition?
): T {
    //...
    
    val instanceContext = InstanceContext(_koin, this, parameters)
    val value = resolveValue<T>(qualifier, clazz, instanceContext, parameterDef)
    //...
    
    return value
}

```


- Scope.resolveValue()

```kotlin
private fun <T> resolveValue(
    qualifier: Qualifier?,
    clazz: KClass<*>,
    instanceContext: InstanceContext,
    parameterDef: ParametersDefinition?
) = (_koin.instanceRegistry.resolveInstance(qualifier, clazz, this.scopeQualifier, instanceContext)
    ?: //...)    
```

- InstanceRegistry.resolveInstance()

```kotlin
internal fun <T> resolveInstance(
    qualifier: Qualifier?,
    clazz: KClass<*>,
    scopeQualifier: Qualifier,
    instanceContext: InstanceContext
): T? {
    val indexKey = indexKey(clazz, qualifier, scopeQualifier)
    //从map里拿
    return _instances[indexKey]?.get(instanceContext) as? T
}
```

- FactoryInstanceFactory

```kotlin
class FactoryInstanceFactory<T>(beanDefinition: BeanDefinition<T>) :
    InstanceFactory<T>(beanDefinition) {

    override fun isCreated(context: InstanceContext?): Boolean = false


    //调用父类的create()
    override fun get(context: InstanceContext): T {
        return create(context)
    }
}
```

- SingleInstanceFactory

```kotlin
class SingleInstanceFactory<T>(beanDefinition: BeanDefinition<T>) :
    InstanceFactory<T>(beanDefinition) {

    private var value: T? = null

    private fun getValue() : T = value ?: error("Single instance created couldn't return value")

    override fun isCreated(context: InstanceContext?): Boolean = (value != null)


    override fun create(context: InstanceContext): T {
        return if (value == null) {
            super.create(context)
        } else getValue()
    }

    override fun get(context: InstanceContext): T {
        KoinPlatformTools.synchronized(this) {
            if (!isCreated(context)) {
                value = create(context)
            }
        }
        return getValue()
    }
}
```

- ScopedInstanceFactory

```kotlin
class ScopedInstanceFactory<T>(beanDefinition: BeanDefinition<T>) :
    InstanceFactory<T>(beanDefinition) {

    private var values = hashMapOf<ScopeID,T>()

    override fun isCreated(context: InstanceContext?): Boolean = (values[context?.scope?.id] != null)

    override fun create(context: InstanceContext): T {
        return if (values[context.scope.id] == null) {
            super.create(context)
        } else values[context.scope.id] ?:  error("Scoped instance not found for ${context.scope.id}")
    }

    override fun get(context: InstanceContext): T {
        if (context.scope.scopeQualifier != beanDefinition.scopeQualifier){
            error("Wrong Scope: trying to open instance for ${context.scope.id} in $beanDefinition")
        }
        KoinPlatformTools.synchronized(this) {
            if (!isCreated(context)) {
                values[context.scope.id] = create(context)
            }
        }
        return values[context.scope.id] ?:  error("Scoped instance not found for ${context.scope.id}")
    }

}
```

- InstanceFactory.create()

```kotlin
open fun create(context: InstanceContext): T {
    val koin = context.koin

    try {
        val parameters: ParametersHolder = context.parameters ?: emptyParametersHolder()
        //看这里实现
        return beanDefinition.definition.invoke(
            context.scope,
            parameters
        )
    } catch (e: Exception) {
        //...
    }
}
```




#### by viewModel()
##### 举个例子

```kotlin
private val exampleViewModel: WeatherViewModel by viewModel()
```

##### 源码
- ViewModelStoreOwnerExt#ViewModelStoreOwner.viewModel()
```kotlin
inline fun <reified T : ViewModel> ViewModelStoreOwner.viewModel(
        qualifier: Qualifier? = null,
        mode: LazyThreadSafetyMode = LazyThreadSafetyMode.SYNCHRONIZED,
        noinline parameters: ParametersDefinition? = null,
): Lazy<T> {
    return lazy(mode) {
        getViewModel<T>(qualifier, parameters)
    }
}

//...
```
- ScopeExt#Scope.getViewModel()

```
fun <T : ViewModel> Scope.getViewModel(viewModelParameters: ViewModelParameter<T>): T {
    val viewModelProvider = createViewModelProvider(viewModelParameters)
    return viewModelProvider.resolveInstance(viewModelParameters)
}


//创建view model
internal fun <T : ViewModel> Scope.createViewModelProvider(
        viewModelParameters: ViewModelParameter<T>,
): ViewModelProvider {
    return ViewModelProvider(
            viewModelParameters.viewModelStore, pickFactory(viewModelParameters)
    )
}

```

- ViewModelResolver#ViewModelProvider.resolveInstance()

```
internal fun <T : ViewModel> ViewModelProvider.resolveInstance(viewModelParameters: ViewModelParameter<T>): T {
    val javaClass = viewModelParameters.clazz.java
    return get(viewModelParameters, viewModelParameters.qualifier, javaClass)
}
```
- ViewModelResolver#ViewModelProvider.get()

```kotlin
internal fun <T : ViewModel> ViewModelProvider.get(
        viewModelParameters: ViewModelParameter<T>,
        qualifier: Qualifier?,
        javaClass: Class<T>,
): T {
    return if (viewModelParameters.qualifier != null) {
        //调用ViewModelProvider的get()方法
        get(qualifier.toString(), javaClass)
    } else {
        get(javaClass)
    }
}
```

#### by inject()
##### 举个例子

```kotlin
private val preferences: Preferences by inject()
```

##### 源码
- ComponentCallbackExt#ComponentCallbacks.inject()
```kotlin
inline fun <reified T : Any> ComponentCallbacks.inject(
        qualifier: Qualifier? = null,
        mode: LazyThreadSafetyMode = LazyThreadSafetyMode.SYNCHRONIZED,
        noinline parameters: ParametersDefinition? = null,
) = lazy(mode) { get<T>(qualifier, parameters) }
```
- ComponentCallbackExt#ComponentCallbacks.get()

```
inline fun <reified T : Any> ComponentCallbacks.get(
        qualifier: Qualifier? = null,
        noinline parameters: ParametersDefinition? = null,
): T {
    return getDefaultScope().get(qualifier, parameters)
}
```
- Scope.get()
> 这里的get()就是上述介绍的get()，inject()和get()的区别就是通过委托实现懒加载
```
inline fun <reified T : Any> get(
    qualifier: Qualifier? = null,
    noinline parameters: ParametersDefinition? = null
): T {
    return get(T::class, qualifier, parameters)
}
```

### 其他问题
- 如何区分注入的相同类型的实例？
> 在注入的时候可以指定Qualifier，然后获取的时候也指定Qualifier即可
- 如何一启动就注入实例?
> 使用双参module的时候第一个参数传true
- 相同实例如何覆盖/不覆盖之前的注入？
> 设置allowOverride(true/false)
- 支持组件化吗？
> 使用loadKoinModules()进行加载其他组件的模块注入

## Koin技术总结
- 注入的容器使用的是ConcurrentHashMap，key是String类型，value是InstanceFactory<*>

```kotlin
//key
"${clazz.getFullName()}:${typeQualifier?.value ?: ""}:$scopeQualifier"
```

- koin大量使用inline和reified关键字

- 利用kotlin特性，委托、Top-Level函数、函数类型参数、lambda表达式等

## Koin与Hilt/Dagger2的区别
[全方面分析 Hilt 和 Koin 性能](https://www.jianshu.com/p/015eec1b1ce0)
### Koin的优缺点

- 优点
    - 从开发角度: 简洁、简单、易上手
    - 从技术角度: 轻量级、无注解没有APT技术、无额外代码生成

- 缺点
    - 运行时进行注入，会增加冷启动的耗时
    - 如果代码编写失误导致注入失败不容易察觉，导致运行时报错
    - 注入的module特别多的时候，管理是一个问题


### Dagger-Hilt的优缺点

- 优点
    - 使用的人比较多，上限比较高
    - 不增加冷启动耗时
    - 代码编写失误导致注入失败编译时会进行错误提示

- 缺点
    - dagger通过注解+APT技术提供依赖，编译耗时明显提升，代码量更多
    - Dagger上手难度高，但是hilt的出现大大降低了上手难度
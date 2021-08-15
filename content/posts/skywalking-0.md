---
title: "Skywalking-0 底层接口简述"
date: 2021-08-09T13:52:36+08:00
draft: false
toc: false
images:
tags:
  - skywalking
  - apm
---

![skywalking-module-diagram](../skywalking/skywalking-module-diagram.png)
## 简述
#### Service:  功能的具体实现; 空接口

#### ModuleServiceHolder: 管理某个模块实现的类
  * `registerServiceImplementation(Class<? extends Service> serviceType, Service service)` 注册 服务，（一个模块中，可能有多个实现）
  * `<T extends Service> T getService(Class<T> serviceType)` 获取实现服务
  
#### ModuleProvider: 模块提供者管理，是 ModuleServiceHolder 的抽象实现
  * 持有 `ModuleDefine` 和 `ModuleManager` 的引用
  * 管理 Service 的 Map: `Map<Class<? extends Service>, Service> services`; 当执行 registerServiceImplementation 时，即往该 map 中添加记录
  * 抽象方法：
    + `Class<? extends ModuleDefine> module()` 获取所属模块的 class
	+ `ModuleConfig createConfigBeanIfAbsent()`： 创建 模块配置
	+ `prepare()`： 准备动作：  In prepare stage, the moduleDefine should initialize things which are irrelative other modules.
	+ `start()`： 开始动作：In start stage, the moduleDefine has been ready for interop.
	+ `notifyAfterCompleted()`：当所有模块都启动完成，作为回调通知： This callback executes after all modules start up successfully.
	+ `String[] requiredModules()`：  当前模块依赖其它模块的名称： moduleDefine names which does this moduleDefine require?
	+ `String name()`： provider 的名称
	
#### ModuleProviderHolder: 模块提供者的管理， 接口类型
  * `ModuleServiceHolder provider()`： 获取模块提供者
  

#### ModuleDefine: 模块的定义，是 ModuleProviderHolder 的抽象实现，
  * 持有 ModuleProvier 的引用 `private ModuleProvider loadedProvider = null`
  * `prepare(ModuleManager moduleManager, ApplicationConfiguration.ModuleConfiguration configuration, ServiceLoader<ModuleProvider> moduleProviderLoader):` 
      module 的 prepare 阶段， 找到所有的 providers 并且让他们执行 prepare
  * 抽象方法：
    + `Class[] services()`： 该模块提供的 Service； 

#### ModuleDefineHolder: 模块定义的管理， 接口类型
  * `boolean has(String moduleName)`： 是否有某个模块
  * `ModuleProviderHolder find(String moduleName)`： 通过模块名获取模块管理者
  
#### ModuleManager: 模块管理者的具体实现
  * 持有所有模块的引用： `final Map<String, ModuleDefine> loadedModules`
  * 变量 isInPrepareStage： 是否在 prepare 的阶段
  * `init(ApplicationConfiguration applicationConfiguration)`: 接收 ApplicationConfiguration 配置进行初始化：
    + a. 

#### ModuleConfig: 表示配置管理类， 抽象类

#### ApplicationConfiguration: 模块化配置类， 模块管理器启动，查找 等都会基于此配置类运行
  * 持有 ModuleConfig 引用： `HashMap<String, ModuleConfiguration> modules`
  * 内部类 
    + ModuleConfiguration: 模块配置类,持有模块提供者的配置引用： `HashMap<String, ProviderConfiguration> providers`
    + ProviderConfiguration: 模块提供者的配置，持有 Properties 对象的引用，表示该 provider 的配置，最终都转化为 Properties 对象
    
#### BootstrapFlow: 启动流程

## 简述总结
skywalking 分模块管理，有多个模块，默认启动 23 个，每个 module 都有一个 provider， 每个 provider 可以注册多个 Service, 而 Service 可以理解为 具体功能的实现； 与 Provider 一起工作；
 

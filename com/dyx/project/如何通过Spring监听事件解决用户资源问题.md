因为本博客是具有权限校验功能的，在项目启动时就会加载用户资源，以便进行权限判断，减少数据库访问次数；但是在项目上线阶段才发现了这个问题，因个人原因，直到现在才解决。
在解决方案的这一块考虑了很多，但都行不通。后面想到了Spring 监听时间，完美解决了这个问题，就是增加了一点访问次数，后续有好的方案再进行替换。
## 实例
在复杂的软件系统中，模块之间的通信和解耦是至关重要的。事件机制是一种广泛使用的设计模式，通过发布和订阅事件，允许对象之间进行松散耦合的通信。Spring框架提供了一个强大且灵活的事件机制，可以方便地实现这一模式。
#### 事件机制概述
事件机制基于发布-订阅模式，核心思想是通过发布事件来通知订阅者，从而触发相应的处理逻辑。这种机制非常适合处理异步操作和解耦场景，特别是在需要通知多个组件或模块某个事件发生时。
#### 核心组件
Spring的事件机制由以下几个核心部分组成：

1. **事件（Event）**：表示发生的某个动作或变化，通常是一个继承自**ApplicationEvent**的类。
2. **事件发布者（Event Publisher）**：负责发布事件的对象，通常通过**ApplicationEventPublisher**接口来发布事件。
3. **事件监听器（Event Listener）**：负责处理事件的对象，通过**@EventListener**注解来标识方法，该方法会在特定事件发生时被调用。
### 事件类
事件类是继承自`**ApplicationEvent**`的类，用于表示特定的事件。它可以包含相关的信息，传递给监听器。
```java
import org.springframework.context.ApplicationEvent;

public class ReloadSecurityEvent extends ApplicationEvent {
    public ReloadSecurityEvent(Object source) {
        super(source);
    }
}
```
### 事件发布者
事件发布者负责发布事件，可以在任何需要通知其他组件的地方使用。通过`**ApplicationEventPublisher**`接口实现事件发布，这里以删除资源信息为例。
```java
@Api(tags = "后台资源管理")
@RestController
@RequestMapping("/resource")
public class ResourceController {

    @Autowired
    private IResourceService resourceService;

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @ApiOperation("删除资源信息")
    @DeleteMapping("deleteResource")
    public ResultObject deleteResource(Long resourceId){
        if (resourceId == null) return ResultObject.failed("资源ID不能为空！");

        if (resourceService.deleteResource(resourceId)){
            eventPublisher.publishEvent(new ReloadSecurityEvent(this));
            return ResultObject.success();
        }
        return ResultObject.failed();
    }
}
```
### 事件监听器
事件监听器是一个带有**@EventListener**注解的方法，当相应的事件发布时，该方法会被调用。
```java
public class DynamicSecurityMetadataSource implements FilterInvocationSecurityMetadataSource {

    private static Map<String, ConfigAttribute> configAttributeMap = null;
    @Autowired
    private DynamicSecurityService dynamicSecurityService;

    @PostConstruct
    public void loadDataSource() {
        configAttributeMap = dynamicSecurityService.loadDataSource();
    }

    public void clearDataSource() {
       if (configAttributeMap != null){
           configAttributeMap.clear();
           configAttributeMap = null;
       }
    }

    public void reloadDataSource() {
        clearDataSource();
        loadDataSource();
        System.out.println("Data reloaded.");
    }

    @EventListener
    public void handleReloadSecurityEvent(ReloadSecurityEvent event) {
        reloadDataSource();
    }
}
```
当`**ReloadSecurityEvent**`事件发布时，`**handleReloadSecurityEvent**`方法会被调用，从而重新加载数据源。

## 论坛网站
[Camunda 中文站 | docs.camunda.org (shaochenfeng.com)](http://camunda-cn.shaochenfeng.com/)
[Camunda Platform ](https://forum.camunda.io/)

camunda支持流程实例的迁移，比如同一个流程有多个实例，多个流程版本，不同流程实例运行在不同的版本中，camunda支持任意版本的实例迁移到指定的流程版本中，并可以在迁移的过程中支持从哪个节点开始。

用户-->组-->租户

## 实际应用
### 实现业务审核退回

#### 业务需求
1. 实现草稿功能，发起人点击保存时将流程保存为草稿并发起流程，点击提交时发起流程并自动完成第一个环节
2. 流程审核过程中，下一步的审核人与上一步的审核人相同时无需指定下一步审核人
3. 流程可以任意一步退回给发起人或退回给第一个环节。在退回过程中无需指定退回给某个处理人
4. 在审核通过时会指定下一步的处理人，退回时无需指定处理人

#### 实现逻辑
1. 监听任务创建
2. 将环节变量设置为“人类型-操作环节类型”，在监听程序将其拆分判断操作人类型和环节
3. 草稿与自动完成发起审核功能，通过操作码与操作类型来判断是否需要完成审核
4. 非提交环节判断是否有指定处理人，有指定处理人说明为审核通过，并设置当前环节流程变量的处理人。无指定处理人说明为审核退回，则取当前环节流程变量处理人。（通过保存变量人，退回则取这个变量的值）
5. 在设置task的人类型相同，没有指定审核人时直接取人类型对应的变量值
6. 退回第一环节读取流程变量initiator

#### 代码实现
```java
import org.apache.commons.lang.StringUtils;
import org.camunda.bpm.engine.TaskService;
import org.camunda.bpm.engine.delegate.DelegateTask;
import org.camunda.bpm.engine.delegate.TaskListener;
import org.springframework.stereotype.Component;
import javax.annotation.Resource;

@Component
public class TaskCreateListener implements TaskListener {
    @Resource
    private TaskService taskService;

    @Override
    public void notify(DelegateTask delegateTask) {
	    //taskDefinitionKey 用来区分人类型和操作环节
        String[] taskDefinitionKey = delegateTask.getTaskDefinitionKey().split("-");
        String operateCode = (String)delegateTask.getVariable("operateCode");
        // 提交环节 提交或退回发起人
        if ("submit".equals(taskDefinitionKey[1])) {
            taskService.setAssignee(delegateTask.getId(), (String)delegateTask.getVariable("initiator"));
            // 是否为草稿，非草稿完成发起任务
            if ("1".equals(operateCode)) {
                taskService.complete(delegateTask.getId());
            }
        } else {
            // 根据节点指派下一步审核人 无nextAssignee则旧的受让人，实现无需指定退回或无需当前任务审核完成后指定与上一步相同的审核人
            String nextAssignee = (String)delegateTask.getVariable("nextAssignee");
            if (StringUtils.isNotBlank(nextAssignee)) {
                taskService.setAssignee(delegateTask.getId(), nextAssignee);
                delegateTask.setVariable(taskDefinitionKey[0], nextAssignee);
            } else {
                String oldAssignee = (String)delegateTask.getVariable(taskDefinitionKey[0]);
                taskService.setAssignee(delegateTask.getId(), oldAssignee);
            }
        }
    }
}
```

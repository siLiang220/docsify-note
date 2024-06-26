[权限体系设计：融合了组织和岗位的权限模型长啥样？ | 人人都是产品经理 (woshipm.com)](https://www.woshipm.com/it/3934803.html)

## 二、RBAC权限控制

### 2.1 RBAC的组成
#### 2.1.1基础模型


在RBAC模型里面有3个基础组成部分，分别是：User用户、Role角色、Permission权限。用户-角色之间的映射关系。角色-权限之间的映射关系

RBAC通过定义角色的权限，并对用户授予某个角色从而来控制用户的权限，实现了用户和权限的逻辑分离（区别于ACL模型)。简化了用户和权限的关系易扩展、易维护。但RBAC模型没有提供操作顺序的控制机制，比较难适应有严格要求操作次序的系统

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307083418.png)


**角色继承：** 例如角色1，角色2，角色3在于角色建立上下级关系后，鉴权授权的前提下会对下属san个角色进行权限继承。而继承的方式又分为"一般继承"和"受限继承"
#### 2.1.2 数据组成

**用户管理:** 用户管理中的用户主要是功能系统的使用者，对应业务的用户有着基本相似的系统功能使用需求和权限等级，对个体进行关联性的初步分群或者分组

**角色管理:** 角色是基于业务管理需求而预先在系统中设定好的，每个角色对应明确的系统权限，是众多最小权限颗粒的组成
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307083638.png)
- **功能权限：** 功能权限是系统执行权限控制的基本单元。针对功能模块来划分用户权限是比较粗颗粒度的一种划分方式。不同角色的用户查看相同的数据字段，但可执行的功能操作不同
- **数据权限：** 数据字段权限是较细颗粒度，实现不同角色用户进入同一模块时，受限于不同的角色权限可见的数据字段都有差异
- **菜单权限：** 控制当前账号可以看到的菜单页面内容

### 2.2 RBAC的模型

#### 2.2.1 RBAC模型

成熟的权限模型强调用户与角色是多对多关系，角色与权限是多对多关系。以“用户”为单位的权限设计；以“权限”为单位的权限设计；以“用户”与“权限”结合的权限设计

#### 2.2.2 RABC-0模型
RBAC0是RBAC模型的核心基础思想，即用户通过被赋予角色和权限进行关联。用户、角色和权限之间的两两关系均是多对多关系，即一个用户可以被赋予多个角色，一个角色可以被赋予多个用户。一个角色可以拥有多项权限，一项权限可以赋予多个角色。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307084107.png)
#### 2.2.3 RBAC-1模型

RBAC1是引入继承关系的RBAC模型，一个角色可以从另一个角色继承许可权，即角色具有上下级的关系.角色间的继承关系可分为一般继承关系和受限继承关系。一般继承关系允许角色间的多继承，受限继承关系则进一步要求角色继承关系是一个树结构

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307084219.png)

从上图举例，角色B继承自角色A，角色A拥有权限1-2，则无须再单独赋予角色B。角色B自动拥有权限1-2，并且还可以单独赋予权限3-4。例如一个部门中的多个业务小组，每个业务小组只能看到自己小组的数据，但部门经理可以看到全部小组的数据。

#### 2.2.3 RBAC-2模型

RBAC-2模型在用户与角色间和角色与角色之间加入了一些规则限制。互斥角色是指各自权限可以互相制约的两个角色。且不能同时获得两个角色的使用权。

规定了权限被赋予角色时或角色被赋予用户时所应遵循的强制性规则。角色互斥、基数约束、先决条件角色等

静态职责分离：同一用户只能分配到一组互斥角色集合中至多一个角色。例如一个用户不能既是“产品”又是“研发”，即当用户被分配为产品时权限页面无法给于其研发的角色权限

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307084531.png)

- **互斥角色**：同一用户在两个互斥角色中只能选择一个。例如在审计活动中，一个角色不能同时被指派给会计角色和审计员角色
- **先决条件角色** ：指要想获得较高的权限要首先拥有低一级的权限。角色A为角色B的上级，要想为用户分配角色A，则必须先分配角色B
- **基数约束**：一个角色被分配的用户数量受限；一个用户可拥有的角色数目受限
- **运行时互斥：** 允许一个用户具有两个角色，但在运行中不可同时激活这两个角色。

静态职责分离：允许一个用户拥有两个角色，但在登录运行中不可同时激活两个角色。即当用户为登录时只能选择以一个角色身份登录，不能以两种角色同时登录

### 2.3 权限关系

#### 2.3.1 用户权限组

系统用户数量或角色种类增多时。可以把相同属性的用户进行归类为用户组。管理员只需对用户组进行角色分配。让用户组中的每个用户自动获取该角色

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307084747.png)

这样用户除了拥有自身的权限外，还拥有了所属用户组的所有权限。同时用户在组中时，同样不影响对该用户单独赋予权限。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307084809.png)

可以把组织与角色进行关联，用户加入组织后，就会自动获得该组织的全部角色，无须管理员手动授予，大大减少工作量，同时用户在调岗时，只需调整组织，角色即可批量调整。

在创建部门下属的账号时要选择这个账号属于哪个层级，则就能看到当前层级及以下层级的所有数据

### 2.4 功能数据权限

#### 2.4.1 功能权限

用户-角色-权限的设计方式是最为简单的设计方式。把操作权限分配给角色，新增一个新的角色以后，就会给这个新增的角色分配相应的操作权限，包括操作权限和数据查看权限

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307085204.png)

功能权限定义：为可见、可以操作的功能范围。控制用户对字段的可见性，可编辑性

- 读写权限：用户具备该字段的最大权限，可见可编辑列表和详情页该字段
- 只读权限：可见列表和详情页该字段。但不可编辑
- 不可见权限：不可见不可编辑列表和详情页该字段

需要注意的一个问题是，权限控制的最小粒度。如果要实现每一个权限的控制，相当于每一个权限对应功能都需要做封装
#### 2.4.2 数据权限

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307085349.png)

数据权限定义：数据是多维的，是抽象的，主要控制某条数据记录对用户是否可见，结合功能权限可以更灵活的配置业务过程中每一位员工的功能操作权限及数据可见范围

- 基础数据权限：即根据数据的负责人来决定
- 数据共享：根据基础数据权限中的数据记录所属将其共享给其它用户查看或编辑

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307085420.png)

数据共享规则是将某个部门/用户(数据来源)的负责的数据(可部分)共享给某个部门、用户或者用户组(共享范围)。配置数据共享规则后，被共享方对共享方所负责的所有数据可见，并具备共享权限对应的操作权限

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307085450.png)

- 数据来源于：即需要共享的数据，选择用户即指该用户负责的记录数据，选择部门即指该部门下员工负责的记录数据
- 共享的数据：选择需共享的对象，比如将用户A负责的客户数据共享给用户B。 数据共享到：被共享方，可选择用户、部门或用户组，被选择的用户、部门或用户组成员将可以看到共享的数据
- 共享后的权限：配置被共享方可对数据查看或是可编辑的权限。如果配置为“读写”权限后，被共享方对共享数据的权限可类比于负责人的权限

### 2.5 权限授权

#### 2.5.1 授权流程
- 授权流程可分为手动授权和审批授权。权限中心可同时配置这两种，可提高授权的灵活性
- 手动授权：管理员登录权限中心为用户授权，给用户添加角色，给角色添加用户
- 审批授权：即用户申请某个职位角色，然后由上级审批该用户即可拥有该角色

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307085612.png)


### 2.6 参考文章

[引用文章](https://zhuanlan.zhihu.com/p/104849603)

## 三、ReBAC权限控制 Zanzibar： 谷歌一致性的全局授权系统
- [ Zanzibar: Google’s Consistent, Global Authorization System](https://zanzibar.tech/)
- [Zanzibar: Google’s Consistent, Global Authorization System](https://storage.googleapis.com/gweb-research2023-media/pubtools/pdf/10683a8987dbf0c6d4edcafb9b4f05cc9de5974a.pdf)
- [Zanzibar的开源实现spicedb](https://github.com/authzed/spicedb)

### 3.1 Zanzibar 的基本实现

Zanzibar 权限系统利用有向无环图（DAG）来模型话权限的关系，其中节点代表资源或者权限等级。在这样的表示中，只要从A节点出发，无论通过多少条路径到达B节点，也不论每条路径上设置了多少层访问控制列表（ACL），只要存在可达路径，A节点关联的实体就具备最终访问到B节点所表示资源的能力。将复杂的权限寻路抽象为按照特定语法的path解析

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307160712.png)


![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20240307160728.png)


例如：
-  对于某一个用户可以直接访问content 
- 对这个用户设置为admin时可以访问content
- 这个用户属于conetnt team 并且content team 属于content operator 也可以访问content

 
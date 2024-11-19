

背景
==

Jenkins 一般用作团队项目持续集成环境，所以就会设计多用户的情况，我们需要为不同人员设置不同的角色，进行权限管理。

可以使用`Role-based-Authorization Strategy`插件，通过基于角色策略来管理 Jenkins 用户权限。

安装插件
====

首先在插件管理菜单搜索下载插件，如下所示：

![1650721985340-f98d163d-3458-48eb-92b8-f0b3c084ef55.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/329b3519727b41cbbcdae33efa43bd9a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

打开全局安全配置，将授权策略改为`Rele-Based Strategy`。

![1650722978987-4441dbfd-dee0-4523-8dda-6fee252af021.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53398bf0f2434690be832fd3f3d0b0d9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

![1650723125172-97360552-a7e0-41b4-8f0f-160818c8b4d0.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/73ee12bc0d654c5ea81c3f44a4bbd012~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

这样，在系统管理菜单里面，在安全区域就可以看到`Manage and Assign Reoles`设置选项了。

![1650723443543-8d0e2654-5ca4-4105-9364-6f66074ac15e.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c723c21b539e4b629c030c1891309658~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

管理角色
====

管理角色，即我们可以创建不同的角色，然后将角色赋予给不同的用户。可以添加3种类型的角色，全局角色，项目角色，节点角色。

![1650723821238-04472a14-8636-4f15-bd4d-2cc80f2c574d.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba1494c60b6a4faba9fef7c70431ac7b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

全局角色
----

默认有个 admin 的全局角色，拥有全部权限，如下所示：

![1650724105619-9fb54b8d-cab5-48fe-b3f1-6efe7f4f8a9d.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3491bc084ec643dda0d7815d3affeb5f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

我们可以创建新的全局角色，对其设置不同的权限，如下：

![1650724959935-5154f4f7-c81c-4839-bbd6-00aa5e427f74.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ceb873142f54ad690d6403f89e7280b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

项目角色
----

我们可以针对不同的项目分配不同的角色，而且还支持通配符，即对项目名称进行匹配。以下创建一个用于 chenpi-mall 项目的角色，`chenpi-mall.*`通配符此角色的用户可以对 chenpi-mall 开头的项目有权限。

![1650726734487-97d356d3-7d0f-4a6c-b0ba-409893e2f06b.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/657194e27f9344b7b7001b5951fc9484~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

节点角色
----

节点角色主要是用于主从 Jenkins 部署时。

![1650727439904-163c3c2f-4db5-4aa9-a1a1-a82fce9c7a8f.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e377838492a449f8c41441a68c82fa7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

管理用户
====

我们可以新增删除用户，如下所示：

![1650727881249-3293bebc-4b8c-436e-876b-b00bbf3ae257.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b8eaf133aaa4210b1279b4df8d6ee87~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

安装好的 Jenkins 已经有一个我们初始化时设置的 root 用户，当然我们也可以新建用户，如下：

![1650727989635-b4a6b1b9-8fd3-443f-8621-ecce26af7428.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37de4aa8d9a44a0a982f4fc84ba5e52f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

![1650728025779-883c18d7-8c76-4f57-a592-70bd9f4204db.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a13a87900e4b4a3e8a4342812171d85d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

新创建的用户，因为还未对其分配权限，所以登录后如下所示：

![1650728236353-5700a7a9-b61c-4dde-87fb-32e7f5ddf7f1.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d45c3f4f3b743f584192885c8ce7666~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

也可以对用户进行删除，但是不能删除 root 用户，如下所示：

![1650728158987-ff4b4a61-3cc4-4370-804c-7fe68620a357.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fde119e978154e45b761a63886fbc1ff~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

分配角色
====

我们添加好角色之后，就可以将这些角色分配给不同的用户了。

![1650727821824-0661ee4b-d37d-482f-8f0b-b178c3918e41.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf9e14b6f70c4933921846683669e38f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

![1650728300583-ddff8419-77d9-4d96-a102-c43190075cee.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6fa098a5ab247ebb96474fccb1d890c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

如下所示，我们对 chenpi 这个用户添加全局角色和项目角色，如下所示：

![1650728347505-80e315d8-5b75-4ac4-baeb-03f9be365880.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49241a89a1184527af07fa6da389bc18~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

使用 root 用户创建一个`chenpi-mall-order`任务。

![1650728647923-466b826c-bb6f-4f1d-b531-7f5e1194cb5b.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ff2c31adcc746ee869a4d912e6b6fc0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

使用 chenpi 用户登录 Jenkins，即可看到项目角色能看到的项目了，如下所示：

![1650728728500-d1f5515e-79df-46e9-9063-faec508054ce.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/635f7a9b33294bc88872f4f4b095fac7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

* * *


<!-- TITLE: First File -->
<!-- SUBTITLE: A quick summary of First File -->

# 角色的获取和权限的计算

1	权限和角色的理解
角色的不同，在于该角色下所赋予的权限的不同。用户因被赋予了角色，所以在系统中，才拥有了各种各样的权限。
1.1 权限的定义
定义于sever/core/constant.js；
1.1.1	全局权限（全局角色权限）

```javascript
global: {
            	message: { //模块名
                	__display_position: 100,//模块位置
                	create_public_channel: {//权限名
                    __display_position: 100,//显示的位置
                    __storage_position: 0,//存储的位置
                    __admin_default: __.is.yes,
                    key: 'create_public_channel’//键名
                },
			}
```

	Get /api/team/roles/:roleId/privileges通过该接口获取数据，展示到前台页面；
其中一点要注意，展示到前端中文释义是怎么来的？
I18n这个文件中，进行了定义；

1.1.2 模块权限

```text
Module：{
message: { //模块名
                	__display_position: 100,//模块位置
                	create_public_channel: {//权限名
                    __display_position: 100,//显示的位置
                    __storage_position: 0,//存储的位置
                    __admin_default: __.is.yes,
                    key: 'create_public_channel’//键名
                },
```

}
Global和module定义的方式是完全一样的，区别在于两者的使用场景并不一致；
Global用于定义全局的角色，即用户在团队中的角色；而module则是定义在用户在各个模块中角色，通过角色来定义每个人的权限。

2	角色的创建和角色权限的赋予
2.1	角色的创建
2.1.1	全局角色的创建
 
2.1.2	模块角色的创建
 
2.2	全局角色创建的时候，当用户是所有者时，他将拥有整个系统的所有权限，包括模块的所有权限；
2.3	模块 项目权限为“管理”时，他将拥有这个项目的所有权限。

3	权限的应用
全局角色，通过你的

4	模块权限
以任务权限为例：
4.1	项目
项目分为私有项目和公开项目，创建公开项目必须具有创建公开项目的权限（系统权限，创建时，需要知道自己的系统角色是否具有创建公开项目的权限）；私有项目，正常情况下，系统默认的成员角色都会具有该权限。
4.1.1	公开项目
公开项目虽然不能创建项目角色，但它赋予每个成员相同的权限，每一个系统用户都可以对其进行操作。（系统所有者拥有全部的权限）
4.1.2	私有项目
私有项目可以创建项目角色， 项目创建者默认拥有“管理”角色，拥有对该项目的所有操作权限；
4.2	任务
任务包括收件箱任务和项目任务；
4.2.1	收件箱任务即当前用户为任务责任人的任务；可以是项目任务也可以自建任务。
4.2.2	项目任务包括公共项目任务和私有项目任务。系统中的每个人（除了所有者）都有对公共任务有相同的操作权限；私有项目任务每个人根据在项目中的角色（某些系统中可能还会根据“工作者角色”和是否是“任务责任人”）来决定在任务中的权限。
5	代码解析
5.1	数据库存储角色
角色存储位置：server/data/model/team/team.role.js
各个字段释义：
team: Schema.Types.ObjectId,
        	role_group: Schema.Types.ObjectId,//角色分组，为系统角色字段
        	name: {type: String, default: ""},
        	icon: {type: String , default: ""},
        	is_system: {type: Number, default: core.constant.isSystem.no}, //是否为系统默认
type: {type: Number, default: core.constant.team.roleType.normal}, //角色类型，该字段现在基本已经废弃不用；
        	position: {type: Number},
        	desc: {type: String, default: ""},
        	is_deleted: {type: Number, default: 0},
        	created_at: {type: Number, default: core.util.getNowTime},
        	created_by: {type: String, default: ""},
        	updated_at: {type: Number, default: core.util.getNowTime},
        	updated_by: {type: String, default: ""},
        	is_disabled: {type: Number, default: core.constant.isDisabled.no},
        	category: {type: Number, default: core.constant.roleType.global}, //全局／模块内
        	is_default: {type: Number, default: core.constant.is.no},
        	module: {type: Number},//指向模块，为模块角色字段；如果是null或者undefined则为系统角色；
			privileges: {type:Schema.Object.fixed},
//privileges是一个复合类型的结构，当该角色为系统角色时，privileges是一个对象，内部属性为模块的属性，和数据范围，以及范围修改人及修改时间等信息；事例如下：
Privilges:{
misssion:{
“scope_updated_by”: "831424a2ea4b4031862fb4bc6f150a07",(uid)
“scope_updated_at”: 1491907530,（时间戳）
“scope”:100,
“value”:“0000”
}
}
当它为模块角色时，priviliges是一个对象，内部有一个value属性，value的值即为各个权限的拼接字段。
事例如下：
	privileges:{
		value:’0000000111111111111’
};
5.2	权限的获取和计算
用户登录账户后，根据用户设置的权限，获取自己可操作的模块；
系统角色的获取
接口：get /api/team
方法：privileges.base.getCombinedGlobalPrivilegesByUser()
combine

 
 








6	获取当前用户所有参与的项目，包括公共项目和私有的项目；公共项目和私有项目的判断是project.visibility === core.constant.mission.visibility.public如果表达式为true则是公共项目，反之则为私有项目；
7	获取项目组所有成员，排除禁用成员；
8	找出当前用户参与的所有项目，并通过设置角色来计算在对应项目中的权限
8.1	从用户参与的所有项目中检索用户角色，以及将来我们将要使用的角色；
8.2	循环遍历每个项目，判断是否是公共项目还是私有项目，如果是私有项目，则将获取的所有角色进行传入到calculateUserPrivilegesByProject这个函数中，如果是公共项目，则将获取的角色中的默认角色传入到calculateUserPrivilegesByProject函数中；
8.3	在上面的函数中进行判断，当前用户是团队所有者还是项目所有者角色，如果是则拥有所有对该任务操作的权限；如果不是则要通过项目和传入的角色进行判断
8.3.1	如果是公开的项目，就使用项目的角色；
8.3.2	如果是私人的项目，如果当前用户是会员，使用`成员。角色`，否则使用任务模块默认的角色（is_default = = =是的）
8.4	

通过privileges字段来获取当前用户所有的权限，对项目以及对任务所有权限，

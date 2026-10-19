创建sys_account用户表

```mysql
CREATE TABLE `sys_account` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `phone` varchar(11) DEFAULT NULL COMMENT '手机号',
  `user_name` varchar(20) DEFAULT NULL COMMENT '用户名',
  `password` varchar(32) DEFAULT NULL COMMENT '密码',
  `position` varchar(150) DEFAULT NULL COMMENT '职位',
  `status` tinyint(1) DEFAULT NULL COMMENT '状态 0：禁用 1：启用',
  `roles` varchar(100) DEFAULT NULL COMMENT '用户拥有的角色多个用逗号隔开',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`user_id`),
  UNIQUE KEY `idx_phone` (`phone`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1000000 DEFAULT CHARSET=utf8mb4 COMMENT='账号信息';
```

创建sys_menu菜单表

```mysql
CREATE TABLE sys_menu(
	menu_id int(11) NOT NULL AUTO_INCREMENT COMMENT 'menu_id,自增主键',
	menu_name VARCHAR(32) NOT NULL COMMENT '菜单名',
	menu_type Int(11) NOT NULL COMMENT '菜单类型 0：菜单 1：按钮',
	menu_url VARCHAR(256) DEFAULT NULL COMMENT '菜单跳转到的地址',
	p_id INT(11) NOT NULL COMMENT '上级菜单ID',
	sort TINYINT(4) DEFAULT '0' COMMENT '菜单排序',
	permission_code VARCHAR(50) DEFAULT NULL COMMENT '权限编码',
	icon VARCHAR(50) DEFAULT NULL COMMENT '图标',
	PRIMARY KEY (menu_id)
) ENGINE=INNODB AUTO_INCREMENT=1000 DEFAULT CHARSET=utf8 COMMENT='菜单表';
```

创建sys_role角色表

```mysql
CREATE TABLE sys_role(
	role_id int(11) NOT NULL AUTO_INCREMENT COMMENT '角色ID',
	role_name VARCHAR(100) NOT NULL COMMENT '角色名称',
	role_desc VARCHAR(255) DEFAULT NULL COMMENT '角色描述',
	create_time datetime DEFAULT NULL COMMENT '创建时间',
	last_update_time TIMESTAMP NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间',
	PRIMARY KEY (role_id)
) ENGINE=INNODB AUTO_INCREMENT=1000 DEFAULT CHARSET=utf8mb4 COMMENT='系统角色表';
```

创建sys_role_2_menu角色对应菜单表

```mysql
CREATE TABLE sys_role_2_menu(
	role_id int(11) NOT NULL COMMENT '角色ID',
	menu_id int(11) NOT NULL COMMENT '菜单ID',
	check_type TINYINT(1) DEFAULT NULL COMMENT '0：半选 1：全选',
	PRIMARY KEY (role_id)
) ENGINE=INNODB DEFAULT CHARSET=utf8mb4 COMMENT='角色对应的菜单权限表';
```


```mysql
CREATE TABLE category(
	category_id INT(11) NOT NULL AUTO_INCREMENT COMMENT '分类ID',
	category_name VARCHAR(30) DEFAULT NULL COMMENT '名称',
	sort TINYINT(4) DEFAULT NULL COMMENT '排序',
	icon_path VARCHAR(150) DEFAULT NULL COMMENT '图标',
	bg_color VARCHAR(10) DEFAULT NULL COMMENT '背景颜色',
	type TINYINT(1) DEFAULT NULL COMMENT '0:问题分类 1:考题分类 2:问题和考题分类',
	PRIMARY KEY (category_id)
) ENGINE=INNODB AUTO_INCREMENT=10000 DEFAULT CHARSET=utf8mb4 COMMENT='分类';
```
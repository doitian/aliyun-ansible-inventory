# Aliyun Ansbile Inventory

[Aliyun][]'s Ansible  [Dynamic Inventory][] script

[Ansible][] 是自动化配置工具，Dynamic Inventory 允许使用脚本获到需要配置的机器列表和信息。

可以参考 hosts.yml 的示例 ansible playbook 如何为所有 ecs 生成 hosts 文件来方便访问，如果换成调用 DNSPod API 就能实现自动更新 DNS 记录。


# 开始

## 安装

可以直接 clone 本项目作为 ansible playbook 的根目录，或者把 inventory 目录复制到您 ansible playbook 的根目录下，并使用 inventory 目录作为 inventtory host file

> 设置 host file 可以使用命令参数 `-i` 或者在 ansible.cfg 中配置，参考本项目中的 ansible.cfg

如果您还有其它的 inventory 条目，也放到 inventory 目录下，参考 ansible multiple inventory sources [相关文档](http://docs.ansible.com/intro_dynamic_inventory.html#using-multiple-inventory-sources)

脚本 aliyun.py 使用 python 2，在默认 python 版本为 3 的环境下使用自行修改 aliyun.py 的第一行。

该脚本依赖阿里去的命令行工具，可以使用 `pip` 安装

	sudo pip install aliyuncli

## 配置

首先需要配置好 `aliyuncli`

	aliyuncli configure

复制示例配置文件并进行修改。配置文件必须命名为 `aliyun.ini` 并且和 `aliyun.py` 在同一目录下。

	cp inventory/aliyun.example.ini inventory/aliyun.ini

然后配置 SSH 连接信息，ecs 对应云主机，配置中支持 Python 的 `%` 替换，比如 ecs 中的 `%(InstanceName)s` 会替换成云主机的名称。括号中可以是任何 `aliyuncli` 返回结果中的字段，另外为了方便使用 IP，还有以下额外字段可以使用

-	`PublicIp` BGP 机房出口 IP
-	`InnerIp` 内网 IP

配置文件还支持对某个 ecs 进行单独配置，只要新建新的小节，以资源类型和名称作为小节的名字，比如 ecs ops 会优先使用 `ecs.ops` 小节中的配置，见下面例子。命名规则见本文档后面的内容。

	[ecs.ops]
	host = ops.example.com
	port = 2222
	user = ops

## 测试

直接运行脚本 `aliyun.py`，没有错误应该会打印出符合 ansible dynamic inventory 要求的 JSON，然后可以运行 ansible 列表所有机器

	ansible all -i inventory --list-hosts

如果一切正常可以测试下 demo playbook hosts.yml

	ansible-playbook hosts.yml

该 playbook 会在当前目录生成 hosts，如果覆盖 /etc/hosts 可以使用里面配置的主机名比如 `ops.ecs` 来访问 ecs 主机了。而如果云主机已经配置能使用 ubuntu sudo 进行 ansible 操作，那么这些主机名也更新到所有的云主机上了。

## 缓存

示例配置中默认开启了缓存，如果改变了 aliyun 的设置想要立即更新主机信息，可以手动执行下面命令刷新缓存

	inventory/aliyun.py --refresh-cache

# 命名规则

## 主机名

即相应资源在 ansible 中使用的 host 名称。

ecs 使用 InstanceName 字段, 也就是管理后台中可以修改的名称。

## 主机组

首先所有的资源都按照类型进行了分组，ecs 主机都属于 ecs 这个组。

另外 ecs 还支持自定义分组，规则是使用『描述』(对应 API 返回结果中的 Description)。业务组名称使用英文逗号分隔之后并加上 `tag_` 前缀即为该主机要加入的 ansible 主机组。比如主机 ops 的业务组名称是 `dev,public` 那么在 ansible 中会包含在组 `tag_dev` 和 `tag_public` 中。

## 主机变量名

所有 API 返回结果以及上面提到的额外 IP 字段都会嵌套在主机变量 aliyun 下，比如在 jinja2 模板中引用出口 IP

    {{ aliyun.PublicIp }}

[ansible]: http://www.ansible.com
[dynamic inventory]: http://docs.ansible.com/intro_dynamic_inventory.html
[aliyun]: http://www.aliyun.com

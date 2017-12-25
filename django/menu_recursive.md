### 需求是对菜单表中的每条记录以树形结构的json格式输出

#### model
```
class Menus(models.Model):
    # 此菜单表也可以理解成权限表
    url = models.CharField(max_length=128)  # 每个url可以当做一个权限
    title = models.CharField(max_length=50)
    menu_desc = models.CharField(max_length=100, null=True, blank=True)
    is_virtual = models.BooleanField(default=False)
    parent = models.ForeignKey('self', null=True, blank=True)   # 自关联

    def __str__(self):
        return self.menu_desc

    class Meta:
        unique_together = ('parent', 'url')
        verbose_name = '菜单列表'
        verbose_name_plural = '菜单列表'
        db_table = "u_menus"
```

### 如下是树形结构两种递归实现方式（严格和非严格）
严格模式: 如果父菜单没被选择，子菜单就算选中也不显示
非严格模式: 如果有子菜单被选择，其对应的父菜单也被选中
```

import pprint
import os
import django
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "wiseops.settings")
django.setup()
from users import models


# -----------------------------------------------------------
#    递归获取权限菜单：第一种实现方式(严格模式)
# -----------------------------------------------------------


"""
def get_menus(user=None, role=None, parent_id=None):
    result = []
    main_menus = get_menu_queryset(user=user, role=role, parent_id=parent_id).distinct()
    for menu in main_menus:
        menu_dict = {
            'id': menu.id,
            'url': menu.url,
            'title': menu.title,
            'is_virtual': menu.is_virtual,
            'children': []
        }
        insert_children_recursive(menu_dict, user=user, role=role)
        result.append(menu_dict)
    return result


def insert_children_recursive(menu_dict, user=None, role=None):
    children_menu = get_menu_queryset(user=user, role=role, parent_id=menu_dict['id']).distinct()
    for m in children_menu:
        child_menu_dict = {
            'id': m.id,
            'url': m.url,
            'title': m.title,
            'is_virtual': m.is_virtual,
            'children': []
        }
        child_menu_dict_append = insert_children_recursive(child_menu_dict, user=user, role=role)
        menu_dict['children'].append(child_menu_dict_append)
    return menu_dict


def get_menu_queryset(user=None, role=None, parent_id=None):
    '''
        分类过滤, 可能会出现如下两种意思
        1、user以role的身份，可以拥有这个菜单
        2、user有这个菜单同时role有这个菜单
    
        例子：
        User.objects.filter(shop__title__contains='shop', shop__gmt_create__year=2008)  # 于2008年注册过商店名包含'shop'的所有用户
        User.objects.filter(shop__title__contains='shop').filter(shop__gmt_create__year=2008)  # 商店名有shop，并且于2008年注册过商店的用户
        User.objects.exclude(shop__title__contains='shop', shop__id=3)  # 过滤到有店id是3并且名字有shop的用户
        User.objects.exclude(shop__title__contains='shop', shop__name__contains='ew')  # 过滤所有title包含shop和name包含ew的
    '''

    menus = models.Menus.objects.filter(parent_id=parent_id)  # 当parent_id = None时，获取的是一级主菜单，否则获取的是子菜单
    if user:
        # menus = menus.filter(role_set__user_set=user) # 默认情况下方向查找规则是model_set
        menus = menus.filter(roles__users=user)
    if role:
        menus = menus.filter(roles=role)
    return menus

"""


# ----------------------------------------------------------
#    递归获取权限菜单：第二种实现方式(非严格模式)
# ----------------------------------------------------------


# """
def get_menus(user=None, role=None):
    result = []
    menus = models.Menus.objects.all()
    if user:
        menus = menus.filter(roles__users=user)
    if role:
        menus = menus.filter(roles=role)
    for menu in menus:
        if menu.parent is None:
            add_menu_to_result(menu, result)
        else:
            menus_recursive(menu, result)
    return result


def add_menu_to_result(menu, result):
    exist = False
    for inner_menu in result:
        if inner_menu['id'] == menu.id:
            exist = True
    if exist:
        return result
    else:
        result.append(
            {
                'id': menu.id,
                'url': menu.url,
                'title': menu.title,
                'is_virtual': menu.is_virtual,
                'children': []
            })


def menus_recursive(menu, result):
    final = result
    parent_menu = get_all_parent_menu(menu)
    parent_menu.reverse()
    for m in parent_menu:
        tmp = get_or_add_menu(m, result)
        result = tmp['children']
    return final


def get_all_parent_menu(menu):
    result = [menu]
    while menu.parent:
        result.append(menu.parent)
        menu = menu.parent
    return result


def get_or_add_menu(menu, result):
    exist = False
    for menu_inner in result:
        if menu_inner['id'] == menu.id:
            exist = True
    if not exist:
        result.append(
            {
                'id': menu.id,
                'url': menu.url,
                'title': menu.title,
                'is_virtual': menu.is_virtual,
                'children': []
            })

    for menu_inner in result:
        if menu_inner['id'] == menu.id:
            return menu_inner
    return result

# """


def get_menus_id_list(menus_json, result=None):
    """
    此函数是为了把菜单列表(嵌套的json格式)转换为菜单id的列表
    :param menus_json: 菜单列表(嵌套的json格式)
    :param result: 菜单id的列表(无嵌套)
    :return: 菜单id的list
    """
    if result is None:
        result = []
    final = result
    for menu in menus_json:
        final.append(menu['id'])
        if menu['children']:
            menu_children = menu['children']
            get_menus_id_list(menu_children, final)
    return final


if __name__ == '__main__':
    user = models.Users.objects.get(id=2)
    role = models.Roles.objects.get(id=3)
    # pprint.pprint(get_menus(user=user))
    menu_json = get_menus(role=role)
    pprint.pprint(menu_json)
    pprint.pprint(get_menus_id_list(menu_json))
    # pprint.pprint(get_menus(user=user, role=None, parent_id=None))
    # print(get_menus(user=user, role=None, parent_id=None))

```

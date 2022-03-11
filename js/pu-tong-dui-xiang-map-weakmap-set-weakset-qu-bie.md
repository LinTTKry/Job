# 普通对象，Map, WeakMap, Set, WeakSet区别

| 普通对象    | 键值只允许字符串和Symbol                                                                                                                                                           |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Map     | <p>键值可以为任意值</p><p>(size\set\get\has\delete\clear\keys</p><p>\values\entries\forEach)</p>                                                                                  |
| WeakMap | <p>只接受对象为键值；</p><p>键值所引用的对象为弱引用，若其引用的对象的其他引用都被清除，垃圾回收会释放该对象的内存；</p><p>不能遍历，只可用set,get,has,delete</p>                                                                      |
| Set     | <p>类似于数组，成员值唯一;</p><p>去除数组中的重复元素[...new Set(array)]或者去除字符串中的重复字符[...new Set('abss')].join(''</p><p>（size/add/get/has/delete/clear/keys/values/entries</p><p>/forEach）</p> |
| WeakSet | <p>成员只能是对象；</p><p>不计入垃圾回收；</p><p>只可用add/get/has/delete;</p>                                                                                                               |


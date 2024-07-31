# CSS 的 position 定位
> position 属性指定了元素的定位类型

五个属性

| 属性     | 说明                                                                                  |
| -------- | ------------------------------------------------------------------------------------- |
| static   | 没有定位，遵循正常的文档流对象，静态定位的元素不会受到 top, bottom, left, right影响。 |
| fixed    | 相对于**浏览器窗口**是固定位置，即使窗口是滚动的它也不会移动                          |
| relative | 相对其**正常位置**的偏移，本所占的空间不会改变                                        |
| absolute | 相对于最近的已定位父元素，如果元素没有已定位的父元素，那么它的位置相对于`<html>`      |
| sticky   | 基于用户的滚动位置来定位，在 position:relative 与 position:fixed 定位之间切换         |

[全部CSS定位属性](https://www.runoob.com/css/css-positioning.html)



---
title: Home Assistant - 打造3D房间
date: 2024-06-23T15:00:00+08:00
tags:
  - 智能家居
categories: 
draft: false
---
在 Home Assistant 中，我们可以将一些传感器或者开关以卡片或者图片的形式添加在UI 界面上，对实体进行控制，比如智能家居里常见的灯泡。但是，如果没有做好清晰的标记的话，经常会搞不清楚这个设备位于家中的哪个位置，于是这时候 Home Assistant 内置的 Picture Elements 卡片便能派上用场了，根据官网描述，该卡片允许你在图片上摆置文本，按钮和服务，并且给了一个例子是在户型图添加功能性的组件，实现对应房间里的开关控制：
![](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-06-23%2015.19.21.png)

根据官网例子，可以轻松的实现通过平面图来控制自己家中的设备，首先我们需要一张家里的户型图，比如这张：
![floorplan.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/floorplan.jpg)
将户型图放在 Home Assistant 的 www 目录下（通常位于 /config 目录中），比如文件名为 floorplant.jpg，下一步，就可以在 Home Assistant 的 lovelace 界面上添加图片元素：
![截屏2024-06-23 15.29.15.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-06-23%2015.29.15.jpg)
编辑配置文件，type 定义为 picture-elements，image 为房间平面图的路径：
```yaml
type: picture-elements
image: /local/floorplan.jpg
elements:
  - type: state-badge
    entity: sensor.sun_next_dawn
    style:
      top: 80%
      left: 10%
```
右侧就可以看到 picture elements 的实时预览：
![截屏2024-06-23 15.33.33.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-06-23%2015.33.33.jpg)
在 elements 字段中，可以编辑添加我们房间中对应的实体，展示在图片上，官方提供了好几种 element 类型：
- [State badge](https://www.home-assistant.io/dashboards/picture-elements/#state-badge)
- [State Icon](https://www.home-assistant.io/dashboards/picture-elements/#state-icon)
- [State Label](https://www.home-assistant.io/dashboards/picture-elements/#state-label)
- [Service Call Button](https://www.home-assistant.io/dashboards/picture-elements/#service-call-button)
- [Icon](https://www.home-assistant.io/dashboards/picture-elements/#icon-element)
- [Image](https://www.home-assistant.io/dashboards/picture-elements/#image-element)
- [Conditional](https://www.home-assistant.io/dashboards/picture-elements/#conditional-element)
- [Custom](https://www.home-assistant.io/dashboards/picture-elements/#custom-elements)

这些类型拥有各自的配置格式，比如我们需要添加一个卧室里的灯泡实体，并且以一个灯泡图标的形式展示在房间位置的中间，打开的时候灯泡图标变量，关闭的时候灯泡图标变量，便可以尝试使用 state-icon 元素：
```yaml
type: picture-elements
image: /local/floorplan.jpg
elements:
  - type: state-icon
    tap_action:
      action: toggle
    entity: input_boolean.zhuwo_light
    style:
      top: 33%
      left: 22%
```
编辑保存后，当去点击灯泡图标时，就会实现将房间灯泡打开，再次点击时，就能够将房间的灯泡关闭。
以此类推，便可以将所有的设备实体以图标或者徽章的形式放置在平面图中的不同位置，起到对房间设备的所在位置有了一目了然的效果。

但是，在家庭房间的实际布置中，可能因为装修或者设计问题会跟平面图存在一些出入，会导致 2D 平面图的呈现还不够直观，或者说达不到心里想要的效果的话，这时便可以继续升级，打造房间的 3D 平面图。
首先还是需要一张户型图，此外还需要用到 3D 设计软件，在这里我用的是上手操作较为简易 Sweet Home 3D，安装好软件后，可以新建一张画布，然后右键导入背景图像，这时可以把户型图进行导入，并输入蓝线对应房间的实际尺寸即可：
![截屏2024-06-23 15.58.43.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-06-23%2015.58.43.jpg)
导入后得到一张带有实际尺寸的户型平面图：
![截屏2024-06-23 16.00.41.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-06-23%2016.00.41.jpg)
点击右键选择绘制墙体，这里按照房屋的实际构造，一般是按照户型图上的墙体区域，划线完成绘制，便得到拥有墙体结构的 3d 户型了，然后把房屋里对应的门窗，家具和床等移动到房屋中的各个区域，调整摆放位置以及长宽高，完成后，右键选择绘制房间，这里就是对房屋中的房间区域进行划分，比如主卧，次卧和客厅等等。绘制完房间之后，最后对房间的地板纹理完成设置，添加房间中的灯光，得到最终的 3d 平面图：
![截屏2024-06-23 16.10.40.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-06-23%2016.10.40.jpg)
在下方的 3d 视图中，可以通过鼠标拖拽找到一个合适的视觉角度，这时候右键选择保存视点，保证接下来导出时都是在同一个视角。右键选择照相，选择最高质量，比例设置为 16:9，时间可以设置当天日落的时间，这样可以使最终渲染的效果明亮度更合适，照相完成后，可以对渲染的图片进行保存。这里我们得到的是一张所有灯光全开的房屋图片，在实际操作中，我们需要渲染一张房屋灯光全关的照片，以及每个房间灯光分别打开的照片，这样才能方便可以展示出不同灯光开启关闭组合的各种情况。
在 Sweet Home 3D 中，将所有灯光的可见性通过取消勾选设置为不可见，给房屋照相，得出一张没有打开任何灯光的房屋照片，这张图片会作为 picture-elements 的底图。接下来，打开选定房间的灯光，开始照相，得出来一张房间区域灯光打开的照片，这时候，打开 Gimp 工具，导入这张图片：
![截屏2024-06-23 16.23.11.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-06-23%2016.23.11.jpg)
对图片右键选择添加透明通道，然后使用套索工具，对房间需要展示的区域通过连线绘制，然后右键选择反转，删除绘制圈外的区域，留下透明的涂层，导出文件，得到一张房间灯光打开的照片：
![截屏2024-06-23 16.23.02.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-06-23%2016.23.02.jpg)
以此类推，完成房屋中所有房间打开灯光的照片导出，在我的例子中，一共会有 9 张图片，图片尺寸完全一致：
- 房屋平面，灯光全关
- 主卧区域，灯光打开
- 次卧区域，灯光打开
- 书房区域，灯光打开
- 卫生间区域，灯光打开
- 客厅区域，客厅灯开
- 客厅区域，厨房灯开
- 客厅区域，客厅等和厨房灯打开
- 透明蒙层

接下来回到 Home Assistant，配置 Picture Elements 卡片，首先将房屋灯光关闭的图片作为底部背景导入：
```yaml
type: picture-elements
image: /local/render/house_dark.png
elements:
```
接下来添加所有房间区域的 image 组件，添加 state_image，on 状态对应为的房间区域灯光打开的照片，off 状态设置为一个透明的图片。

```yaml
  - type: image
    entity: input_boolean.zhuwo_light
    tap_action: none
    hold_action: none
    state_image:
      'on': /local/render/zhuwo_lighton.png
      'off': /local/render/transparent.png
      unavailable: /local/render/transparent.png
    style:
      top: 50%
      left: 50%
      width: 100%
  - type: image
    entity: input_boolean.ciwo_light
    tap_action: none
    hold_action: none
    state_image:
      'on': /local/render/ciwo_light_on.png
      'off': /local/render/transparent.png
      unavailable: /local/render/transparent.png
    style:
      top: 50%
      left: 50%
      width: 100%
  - type: image
    entity: input_boolean.shufang_light
    tap_action: none
    hold_action: none
    state_image:
      'on': /local/render/shufang_light_on.png
      'off': /local/render/transparent.png
      unavailable: /local/render/transparent.png
    style:
      top: 50%
      left: 50%
      width: 100%
  - type: image
    entity: input_boolean.weishengjian_light
    tap_action: none
    hold_action: none
    state_image:
      'on': /local/render/weishengjian_light_on.png
      'off': /local/render/transparent.png
      unavailable: /local/render/transparent.png
    style:
      top: 50%
      left: 50%
      width: 100%
```
然后还要再房间中添加灯泡的图标，tap_action 设置为 toggle，来控制各个房间区域的图片状态切换，在此之前记得要将房间区域的 tap_action 设置为 none：
```yaml
  - type: state-icon
    entity: input_boolean.zhuwo_light
    tap_action:
      action: toggle
    hold_action: none
    icon: mdi:lightbulb
    style:
      top: 40%
      left: 30%
  - type: state-icon
    entity: input_boolean.ciwo_light
    tap_action:
      action: toggle
    hold_action: none
    icon: mdi:lightbulb
    style:
      top: 30%
      left: 50%
  - type: state-icon
    entity: input_boolean.shufang_light
    tap_action:
      action: toggle
    hold_action: none
    icon: mdi:lightbulb
    style:
      top: 70%
      left: 42%
  - type: state-icon
    entity: input_boolean.keting_light
    tap_action:
      action: toggle
    hold_action: none
    icon: mdi:lightbulb
    style:
      top: 40%
      left: 64%
  - type: state-icon
    entity: input_boolean.weishengjian_light
    tap_action:
      action: toggle
    hold_action: none
    icon: mdi:lightbulb
    style:
      top: 65%
      left: 52%
  - type: state-icon
    entity: input_boolean.chufang_light
    tap_action:
      action: toggle
    hold_action: none
    icon: mdi:lightbulb
    style:
      top: 80%
      left: 67%
```
不过这里遇到了一个问题是因为客厅和厨房都在同一个区域，那么房间区域的灯光就会存在四种状态：客厅关&厨房关，客厅开&厨房关，客厅关&厨房开，客厅开&厨房开，虽然可以很好处理只有客厅灯开或者只有厨房灯开的这种情况，但是当两个区域的灯都打开时，图片便会出现重叠，造成图片割裂的现象，对于这种情况用传感器来代替单一灯光便是一个很好的选择，首先得在 Sweet Home 3D 中渲染出客厅灯和厨房灯同时打开的照片，然后继续用 Gimp 将客厅区域的图层抠出来，保存待用。编辑 configuration.yml 配置文件，这里参考了论坛的做法，添加一个客厅灯光判断的传感器，编辑 value_template，对应到上面的这四种情形：
```yaml
sensor:
  - platform: template
    sensors:
      keting_choose:
        friendly_name: '客厅灯光判断'
        value_template: >
          {% if is_state('input_boolean.keting_light', 'on') %}
            {% if is_state('input_boolean.chufang_light', 'off') or is_state('input_boolean.chufang_light', 'unavailable') %}
              1
            {% else %}
              3
            {% endif %}
          {% elif is_state('input_boolean.chufang_light', 'on') %}
            {% if is_state('input_boolean.keting_light', 'off') or is_state('input_boolean.keting_light', 'unavailable') %}
              2
            {% else %}
              3
            {% endif %}
          {% else %}
            0
          {% endif %}
```
在 Home Assistant 的配置中添加实体为这个传感器的 image 组件：
```yaml
  - type: image
    entity: sensor.keting_choose
    tap_action: none
    hold_action: none
    state_image:
      '0': /local/render/transparent.png
      '1': /local/render/keting_light_on.png
      '2': /local/render/kitchen_light_on.png
      '3': /local/render/keting_all_light_on.png
      'off': /local/render/transparent.png
      unavailable: /local/render/transparent.png
    style:
      top: 50%
      left: 50%
      width: 100%
```
在这步之后，基本上就完成了 3d 房间的配置了，接下来可以用开关来调试打开时 3d 平面图展示的效果：
![截屏2024-06-23 16.58.22.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-06-23%2016.58.22.jpg)
甚至，可以在此基础上进行一些小定制，比如用 service-button 来完成“回家模式”以及“离家模式”：
```yaml
  - type: service-button
    title: 回家模式
    style:
      top: 80%
      left: 10%
    service: homeassistant.turn_on
    service_data:
      entity_id:
        - input_boolean.chufang_light
        - input_boolean.keting_light
  - type: service-button
    title: 离开模式
    style:
      top: 90%
      left: 10%
    service: homeassistant.turn_off
    service_data:
      entity_id:
        - input_boolean.chufang_light
        - input_boolean.keting_light
        - input_boolean.weishengjian_light
        - input_boolean.zhuwo_light
        - input_boolean.ciwo_light
        - input_boolean.shufang_light
```
当回家时，打开客厅的所有灯光，而在离家时，关闭房屋里的所有灯光。来测试一下回家模式的效果：
![截屏2024-06-23 17.02.25.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-06-23%2017.02.25.jpg)
再来对比一下客厅灯和厨房灯分别打开的效果：
![截屏2024-06-23 17.03.53.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-06-23%2017.03.53.jpg)
![截屏2024-06-23 17.04.03.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-06-23%2017.04.03.jpg)

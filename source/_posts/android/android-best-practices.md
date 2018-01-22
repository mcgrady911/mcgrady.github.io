# android-best-practices



## 摘要

- 把密码和敏感数据放在`gradle.properties`中，并在版本控制中忽略掉




## 资源文件

### （1）布局文件

| **Type**    | **Format**   |
| ----------- | ------------ |
| Activity    | activity_    |
| Fragment    | fragment_    |
| Dialog      | dialog_      |
| PopupWindow | popup_       |
| Menu        | menu_        |
| Adapter     | layout_item_ |

### （2）图片
| **Suffix** | **Meaning** |
| ---------- | ----------- |
| bg_xxx     | 背景图片        |
| btn_xxx    | Button      |
| ic_xxx     | Icon        |
| bg_描述_状态   | 不同状态的背景     |
| btn_描述_状态  | 不同状态的Button |
| chx_描述_状态  | 选择框         |

### （3）values
| **类别**  | **命名**        | **示例**            |
| ------- | ------------- | ----------------- |
| strings | 模块名_逻辑名称      | main_menu_about   |
| colors  | 模块名_颜色名称      | main_info_bg_gray |
| style   | 模块名_逻辑名称（驼峰法） | main_tabBottom    |
| dimen   | 模块名_逻辑名称      | main_menu_padding |


## 附录
表1：UI控件缩写表
| **控件**         | **缩写** | **示例**    |
| -------------- | ------ | --------- |
| LinearLayout   | ll     | menuLL    |
| RelativeLayout | rl     |           |
| FrameLayout    | fl     |           |
| Button         | btn    | searchBtn |
| ImageButton    | ibtn   | playIBtn  |
| TextView       | tv     | nameTV    |
| ListView       | lv     | carLV     |
| GridView       | gv     | photoGV   |
| ProgressBar    | pb     |           |
| RadioButton    | rbtn   |           |
| RadioGroup     | rg     |           |
| ScrollView     | sv     |           |
| RecyclerView   | rv     |           |
| WebView        | wv     |           |
| Spinner        | spn    |           |
|                |        |           |
|                |        |           |
|                |        |           |
|                |        |           |


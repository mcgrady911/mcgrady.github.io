# Android 编码规范

## 命名规范

### 方法名

`lowerCamelCase `风格

| 方法     | 说明 |
| -------- | ---- |
| `initXX()` |  初始化相关方法，使用 init 为前缀标识，如初始化布局 |
| `initView()`   `isXX()`, `checkXX()` | 方法返回值为 boolean 型的请使用 is/check 为前缀标识 |
| `getXX()` | 返回某个值的方法，使用 get 为前缀标识 |
| `setXX()` | 设置某个属性值 |
| `handleXX()`, `processXX()` | 对数据进行处理的方法 |
| `displayXX()`, `showXX()` | 弹出提示框和提示信息，使用 display/show 为前缀标识 |
| `updateXX()` | 更新数据 |
| `saveXX()`, `insertXX()` | 保存或插入数据 |
| `resetXX()` | 重置数据 |
| `clearXX()` | 清除数据 |
| `removeXX()`, `deleteXX()` | 移除数据或者视图等，如 `removeView()` |
| `drawXX()` | 绘制数据或效果相关的，使用 draw 前缀标识 |

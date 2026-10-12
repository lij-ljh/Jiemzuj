# Android 界面编程实验报告

## 项目概述

本项目完成了 Android 界面编程的 5 个实验，涵盖 `ListView`、`AlertDialog`、`XML 菜单`、`ActionMode` 和 `RecyclerView` 等核心 UI 组件。

## 开发环境

| 项目 | 版本 |
|------|------|
| Android Gradle Plugin | 8.7.3 |
| Gradle | 8.11.1 |
| Kotlin | 2.0.21 |
| compileSdk / targetSdk | 34 |
| minSdk | 24 |
| JDK | 17 (Corretto) |
| IDE | Android Studio |

## 项目结构

```
app/src/main/
├── AndroidManifest.xml
├── java/com/example/myapplication/
│   ├── LauncherActivity.kt       # 主入口（导航菜单）
│   ├── Exp1Activity.kt           # 实验1：ListView + SimpleAdapter + Notification
│   ├── Exp2Activity.kt           # 实验2：自定义 AlertDialog
│   ├── Exp3Activity.kt           # 实验3：XML 菜单
│   ├── Exp4Activity.kt           # 实验4：ActionMode 上下文菜单
│   └── Exp5Activity.kt           # 实验5：RecyclerView 动物列表
└── res/
    ├── layout/
    │   ├── activity_exp1.xml / item_exp1.xml
    │   ├── activity_exp2.xml / dialog_custom.xml
    │   ├── activity_exp3.xml
    │   ├── activity_exp4.xml / item_exp4.xml
    │   └── activity_exp5.xml / item_exp5.xml
    ├── menu/
    │   ├── menu_exp3.xml
    │   └── menu_actionmode.xml
    └── drawable/
        ├── animal_cat.png / animal_monkey.png / animal_tiger.png
        ├── animal_lion.png / animal_elephant.png
        └── ...
```

---

## 实验1：ListView + SimpleAdapter + Notification

### 实验要求
- 使用 `SimpleAdapter` 实现带图片的列表
- 单击列表项显示 Toast
- 单击后发送通知（Icon 为应用图标，Title 为列表项内容）

### 实现方案
- **数据**：每一项包含 `icon`、`title`、`desc` 三个字段，存入 `ArrayList<HashMap<String, Any>>`
- **SimpleAdapter 映射**：`from = arrayOf("icon","title","desc")` → `to = intArrayOf(R.id.iv_icon, R.id.tv_title, R.id.tv_desc)`
- **点击事件**：`setOnItemClickListener` 中同时弹出 Toast 和发送通知
- **通知**：使用 `NotificationCompat.Builder`，小图标 `R.mipmap.ic_launcher`，点击通知跳转回 Activity
- **通知渠道**：Android 8.0+（API 26）必须创建 `NotificationChannel`

### 关键代码
```kotlin
val adapter = SimpleAdapter(this, data, R.layout.item_exp1, from, to)
listView.adapter = adapter

listView.setOnItemClickListener { _, _, position, _ ->
    val item = data[position]
    Toast.makeText(this, "选中：${item["title"]}", Toast.LENGTH_SHORT).show()
    sendNotification(item["title"] as String, item["desc"] as String)
}
```

### 运行效果
- 列表显示 7 种动物（图标 + 名称 + 描述）
- 点击任意一项，Toast 显示选中项
- 通知栏同步出现一条通知，Title 为动物名称，展开可看到描述

---

## 实验2：自定义布局 AlertDialog

### 实验要求
- 创建自定义布局的对话框
- 使用 `AlertDialog.Builder.setView()` 将布局嵌入对话框

### 实现方案
- **自定义布局** `dialog_custom.xml`：包含 姓名输入框、年龄输入框、同意条款复选框
- **对话框构建**：`AlertDialog.Builder(this).setTitle("用户信息").setView(view).setPositiveButton().setNegativeButton()`
- **结果处理**：点击"确定"后读取输入内容，用 Toast 汇总显示

### 关键代码
```kotlin
val view = layoutInflater.inflate(R.layout.dialog_custom, null)
AlertDialog.Builder(this)
    .setTitle("用户信息")
    .setView(view)
    .setPositiveButton("确定") { _, _ ->
        // 读取 EditText 和 CheckBox，Toast 输出
    }
    .setNegativeButton("取消", null)
    .show()
```

### 运行效果
- 主界面一个按钮"打开对话框"
- 点击弹出自定义对话框，可填写姓名、年龄、勾选同意条款
- 确定后 Toast 显示所有填写内容

---

## 实验3：XML 定义菜单

### 实验要求
- 字体大小菜单（小 10号 / 中 16号 / 大 20号）
- 字体颜色菜单（红色 / 黑色）
- 普通菜单项（点击弹 Toast）

### 实现方案
- **菜单资源** `menu_exp3.xml`：使用 `<group>` + `<item>` 构建子菜单
- **创建菜单**：`onCreateOptionsMenu` 中 `menuInflater.inflate(R.menu.menu_exp3, menu)`
- **事件处理**：`onOptionsItemSelected` 中根据 `item.itemId` 分支处理，调用 `tvTest.textSize` 和 `tvTest.setTextColor()` 修改测试文本

### 菜单结构
```
menu_exp3.xml
├── 字体大小 (子菜单)
│   ├── 小 (10号)
│   ├── 中 (16号)
│   └── 大 (20号)
├── 字体颜色 (子菜单)
│   ├── 红色
│   └── 黑色
└── 关于 (普通项)
```

### 运行效果
- ActionBar 右上角显示菜单图标
- 点击展开菜单，可切换字体大小和颜色
- 中心测试文本实时响应变化，每次操作弹 Toast 提示

---

## 实验4：ActionMode 上下文菜单

### 实验要求
- 使用 ListView 实现列表
- 长按列表项触发 ActionMode 上下文菜单
- 菜单包含分享、收藏、删除等操作

### 实现方案
- **列表**：`ArrayAdapter` 简单字符串列表
- **触发方式**：`setOnItemLongClickListener` 中调用 `startActionMode(callback)`
- **ActionMode 回调**：实现 `ActionMode.Callback` 的四个方法
  - `onCreateActionMode`：初始化菜单 + 设置标题
  - `onActionItemClicked`：处理分享 / 收藏 / 删除
  - `onDestroyActionMode`：清理选中状态
- **切换选中**：ActionMode 激活状态下，点击其他项切换选中

### 关键代码
```kotlin
listView.setOnItemLongClickListener { _, _, position, _ ->
    selectedPos = position
    listView.setItemChecked(position, true)
    actionMode = startActionMode(callback)
    true
}
```

### 运行效果
- 长按任意列表项 → 顶部 ActionBar 变为上下文菜单（标题为选中项）
- 三个操作按钮：分享、收藏、删除
- 删除后列表立即更新，ActionMode 自动关闭
- 点击其他项可切换选中，ActionMode 标题同步更新

---

## 实验5：RecyclerView 动物列表

### 实验要求
- 使用 `RecyclerView` + `Adapter` + `ViewHolder` 展示动物图片和名称
- 单击显示名称，发送通知
- 添加一条记录后列表立即更新
- 滚动正常、点击正确、更新无错位

### 实现方案
- **数据模型**：`data class Animal(val name: String, val iconRes: Int)`
- **Adapter**：继承 `RecyclerView.Adapter<VH>`，内部维护 `selectedPos` 实现**单选高亮**
- **点击反馈**：
  1. 背景变红色（`Color.RED`）
  2. Toast 显示动物名称
  3. 发送系统通知（Icon = 应用图标，Title = 动物名）
- **切换高亮**：点击新项时，先 `notifyItemChanged(prev)` 清除旧高亮，再 `notifyItemChanged(position)` 设置新高亮
- **动态添加**：按钮点击 → `animals.add(new)` → `adapter.notifyItemInserted(size-1)` → `scrollToPosition` 滚动到新项
- **通知权限**：Android 13+ 动态请求 `POST_NOTIFICATIONS`

### 关键代码
```kotlin
holder.itemView.setOnClickListener {
    val prev = selectedPos
    selectedPos = position
    notifyItemChanged(prev)
    notifyItemChanged(position)
    onClick(a)  // → Toast + Notification
}
```

### 运行效果
- 启动显示 5 种动物（图片 + 名称）
- 点击动物 → 背景变红 + Toast + 通知栏通知
- 再点另一只 → 前一只恢复透明，新一只变红
- 点"添加动物"按钮 → 列表底部立即插入新动物并自动滚动
- 滚动列表流畅，View 复用正确无错位

---

## 实验总结

| 实验 | 核心组件 | 关键知识点 |
|------|----------|------------|
| 1 | ListView + SimpleAdapter | 列表适配器映射、Notification + Channel |
| 2 | AlertDialog.Builder | 自定义 setView、布局复用 |
| 3 | Options Menu (XML) | menuInflater、onOptionsItemSelected、子菜单 |
| 4 | ActionMode | startActionMode、Callback、CHOICE_MODE_SINGLE |
| 5 | RecyclerView | ViewHolder 模式、notifyItemChanged、单选高亮、动态添加 |

## 构建与运行

Android Studio 打开项目根目录 → 等待 Gradle Sync 完成 → 选择模拟器/真机 → 点击 ▶ Run

APK 输出：`app/build/outputs/apk/debug/app-debug.apk`

**注意**：修改 `AndroidManifest.xml` 权限后需**卸载旧版本重新安装**，否则 Android 不会授予新增的权限（如 POST_NOTIFICATIONS）。

## 参考资料

- [Android 官方 RecyclerView 文档](https://developer.android.google.cn/guide/topics/ui/layout/recyclerview)
- [Android 官方通知文档](https://developer.android.com/develop/ui/views/notifications)
- [Android 官方菜单文档](https://developer.android.com/develop/ui/views/menus)

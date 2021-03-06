# DevTools



## Idea

### 快捷键

| 快捷键             | 作用                        | 类别    |
| ------------------ | --------------------------- | ------- |
| Ctrl-N             | 查找类                      | 查找    |
| Ctrl-Shift-N       | 查找文件                    |         |
| Ctrl-Shift-Alt-N   | 查找符号（方法，属性）      |         |
| Ctrl-F             | 当前文件查找（F3,SHITF-F3） |         |
| Ctrl-Shift-F       | 项目全局查找                |         |
| Ctrl-]/[           | 匹配大括号                  | 浏览    |
| Ctrl-Shit-]/[      | 选中括号代码块              |         |
| Ctrl-H             | 显示类继承层次              |         |
| Alt-7              | 显示（类，POM）文件结构     | Alt-1-7 |
| Ctrl-B             | 查看定义                    |         |
| Ctrl-X             | 删除当前行                  | 编辑    |
| Ctrl-D             | 复制当前行                  |         |
| Ctrl-Alt-L         | 格式化代码                  |         |
| Ctrl-Alt-O         | 优化导入                    |         |
| Ctrl-Alt-I         | 缩进                        |         |
| Alt-Insert         | 生成代码                    |         |
| Ctrl-Shift-U       | 大小写切换                  |         |
| Ctrl-Shift-UP/Down | 代码上(下)移一行            |         |
| Ctrl-G             | 去那一行                    |         |
| Shift-F6           | 重命名                      | 重构    |
| Ctrl-/             | 注释                        |         |
| Ctrl-J             | 插入模板                    | 模板    |
| Ctrl-Alt-J         | 模板环绕                    |         |
| sout               | System.out.println          |         |



## MVN

- 多模块更新版本

  

- 指定模块打包

  - -pl --project 指定模块（多个模块以逗号分隔）
  - -am --also-make 同时处理选定模块所依赖的模块
  - -amd --also-make-dependents 同时处理依赖选定模块的模块
  - -N --Non-recursive 不递归子模块
  - -rf --resume-from 从指定模块开始继续处理

  ```bash
  mvn clean package -pl mcps-cache-starter -am
  ```
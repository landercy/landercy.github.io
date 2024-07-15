# UML - Unified Modeling Language

包括：

- 使用案例图 Use case diagrams
- 类图 Class diagrams
- 序列图 Sequence diagrams
- 合作图 Collaboration diagrams
- 状态图 Statechart diagrams
- 活动图 Activity diagrams
- 构件图 Component diagrams
- 部署图 Deployment diagrams

## 类图

- 类名

  - 正体字：可实例化的类
  - 斜体字：抽象类
  - `<<interface>>`: 接口
- 属性和方法
  - `:`：变量类型或方法返回值类型
  - `+`：public
  - `-`： private
  - `#`: protected
  - 下划线：静态属性或静态方法

类图中的静态关系

- 一般化关系 Generalization
  - 继承（泛化）关系：实线+空心三角，从子类指向父类
  - 实现关系：虚线+空心三角，从实现类指向接口
- 关联关系 Association
  - 类与类之间的关系：如A类的某属性是B类的实例
  - 实线+箭头，可标注关联名称
    - 单向有箭头指示关联方向
    - 双向可有两箭头或无箭头
  - 在关联的端点，可标注基数 Multiplicity
    - 0..1：零个或一个实例
    - 0..* 或 *：零个或多个实例（无限制）
    - 1：只有一个实例
    - 1..*：一个或多个实例（至少一个）
- 聚合关系 Aggregation
  - 整体和个体之间的关系，是更“强”的“关联关系”
  - 空心菱形+实线+箭头：从整体指向个体
  - 个体可以脱离整体单独存在，整体销毁不影响个体
- 合成（组合）关系 Composing
  - 更“强”的聚合关系，整体负责个体的生命周期
  - 实心菱形+实线+箭头：从整体指向个体
  - 个体不能脱离整体单独存在，整体销毁则个体销毁
- 依赖关系
  - 类与类之间的关系
  - 虚线+箭头
  - 依赖总是单向的
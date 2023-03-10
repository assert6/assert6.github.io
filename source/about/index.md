# 张城铭

> Hyperf 开发组成员, 前Swoft 开发组成员

## 曾任
#### KK集团 | 深圳后端组负责人 (2021.07 - 2023.02)
负责全集团后端岗位技术初面, 近一年筛选简历400+ 面试150+
担任**KK讲师**, 为后端同事提供技术培训, 著有《Hyperf 正确打开方式》、《手把手拆解Hyperf》
制定后端工作流程规范, 梳理CodeReview 规范及模板, 通过安利快捷键及扩展, 提高全员开发效率
开展IDP 面谈, 进一步激发团队成员的目标感, 提高团队主动性和Owner 意识

#### KK集团 | 电商中台部技术负责人 (2018.12 - 2021.07)
负责电商中台部系统技术架构选型, 参与团队日常需求及技术方案评审
主导完成**原FPM 框架迁移Hyperf**、单体架构拆分**微服务**架构、**分布式事务**解决方案等重难点技术突破
兼任系统运维工作, 完成生产环境Swarm 迁移K8s 集群, 搭建线上环境自动部署、监控、日志、报警等运维工作

#### 前海果树财富 | 后端工程师 (2018.03 - 2018.12)
负责公司业务系统的开发、维护及运维工作
参与公司业务系统的技术选型、架构设计、技术方案评审
参与公司业务系统的性能优化、安全加固、代码重构等工作

## 项目经验
深度参与新零售行业全链路项目开发, 涵盖零售终端POS 系统、电商系统、供应链系统、仓储系统、ERP 系统, 以及客制化系统的商业化转型经验;

#### FPM 框架迁移Hyperf
- 项目背景
  - 2019年之前, 公司电商业务系统采用FPM 框架开发, 由于FPM 同步阻塞的模型决定了性能无法得到数量级的提升, 框架整体已经无法满足公司电商业务系统的快速发展需求
  - 此时每逢业务高峰期, 业务系统的响应时间都会达到秒级, 严重影响用户体验, 甚至导致业务系统宕机, 严重影响公司业务
  - 在此背景下, 由我主导将公司电商业务系统迁移Hyperf 框架, 以期实现业务系统的性能提升
- 项目亮点
  - 通过Hyperf 框架的协程模型, 使得业务系统性能得到数量级的提升, 业务系统QPS 从原来的300+ 提升到了3000+
  - 在经过半年的使用过程中, 业务系统稳定性得到了大幅度的提升, 平稳度过618 大促后, Hyperf 在620 开源
  - 摸索出了FPM 框架迁移Hyperf 的最佳实践, 也是首个Hyperf 框架的生产环境应用, 为公司业务系统的快速发展提供了可靠的技术保障

#### 单体架构拆分微服务
- 项目背景
  - 公司电商业务系统在迁移Hyperf 之初, 还是保持单体架构开发, 由于单体架构的代码耦合度高, 业务系统的扩展性差, 庞大的单体代码库已经严重限制了业务系统的快速迭代
  - 庞大的代码库导致无法快速编译, 在Hyperf 2 版本时, 冷启动一次需要7分钟, 严重影响开发效率, 甚至导致修复线上BUG 时无法快速发布上线
  - 单体架构的弹性伸缩能力差, 单个业务模块的性能瓶颈会拖累整个业务系统的性能, 只能整体扩容无法针对单模块扩容, 浪费了大量的服务器资源
- 项目亮点
  - 通过微服务架构, 将庞大的单体代码库拆分成多个微服务, 弹性伸缩能力、开发效率、稳定性得到大幅度的提升, 也加强了业务系统的快速迭代能力
  - 服务间采用`serialize` 协议的RPC 调用, 与传统JSON RPC 调用相比, 参数可以传递对象, 更加面向对象, 代码更简洁易理解, 更适合团队协作开发
  - 通过开发熔断、降级、链路追踪等逻辑, 在业务流量洪峰时, 保证了微服务架构的稳定性, 也顺势开源相关组件代码, 为社区贡献了一份力量

#### 电商小程序平台化
- 项目背景
  - 在KK 品牌电商业务整体蓬勃发展的背景下, 集团内的TheColorist、X11、调色之家等线下零售品牌, 也开始筹措资源, 准备发展线上电商业务
  - 为了统一管理线上电商业务, 集团内的各个品牌, 都需要开发自己的电商小程序, 但是各个品牌的电商小程序业务数据要求相互隔离, 不能相互影响
  - 前期进行了调研, 各品牌小程序独立部署会导致大量资源浪费, 全面SaaS 化的方案则投入大风险高, 无法快速迭代, 最终选择了平台化的方案
- 项目亮点
  - 通过平台化的方案, 将各个品牌的电商小程序业务数据隔离, 但是共享同一套基础设施, 降低了资源浪费, 也节省了运维成本
  - 在后续搭建新的零售品牌电商小程序时, 只需要在原有系统进行配置, 就可以快速搭建出一个全新的电商小程序, 不到一周即可上线
  - 平台化组件通过Hyperf AOP及DI 功能, 在不修改原有业务代码的情况下, 将底层的平台化逻辑注入到各个服务中, 降低了开发者的学习成本

#### Agent 代理商服务
- 项目背景
  - 在三年疫情的影响下, 整个零售行业收入出现下滑, 此时线上电商的战略方向必须从有调性的宣传转变为增加实际营收, 成为公司业绩的另一个重要支柱
  - 自营小程序的流量虽然有逐步提升, 但是远远无法与淘宝、京东、天猫等第三方平台企及, 必须通过第三方平台的流量, 才能实现线上电商的快速发展
  - 如何通过技术支撑, 给运营团队提供更好的工具, 从而实现电商战略的U型转弯, 是技术团队需要思考的问题
- 项目亮点
  - 通过对第三方平台接入流程的抽象, 将第三方平台抽象为一个个的Adapter, 通过Interface 约束, 使得开发同事后续接入第三方平台时, 只需要实现对应的接口, 就可以快速完成订单同步、商品同步、商品库存同步等功能
  - 通过Hyperf AOP功能, 封装了`@Shop` 注解, 可以无感的将各个平台账号注入到业务逻辑中, 自动遍历平台账号维度, 执行对应的业务逻辑
  - 国内已接入小红书、京东、得物、抖音、天猫、拼多多, 国外覆盖ShopOnline、Tokopedia、Tiktok、Lazada、Shopee, 未来还会继续扩展更多的平台
  - Agent 代理商服务上线短短一年, 营业额已经超过了自营电商小程序, 与运营团队一起实现了公司的电商战略U型转弯

## 我能提供
如果是初创公司, 可以满足从初创到上市前的技术把控, 涵盖技术选型、架构设计、技术方案评审及开发、服务部署、线上运维等; 
如果是行业翘楚, 可以参与需求研发、系统重构, 开展内部技术培训, 规范工作流程及推动内源项目, 带动团队成员的技术成长; 

## 联系我
- 微信: assert666
- 邮箱: z@hyperf.io
- Github: [https://github.com/assert6](https://github.com/assert6)
- 博客: [https://assert6.com/](https://assert6.com/)


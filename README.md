# 宿舍管理系统（Dorm Management System）

> 软件工程课程设计 / 毕业设计项目：SpringBoot 2.7 + MyBatis-Plus + MySQL 8 + Vue 3 + Vite + Element Plus
> 前后端分离，三种角色（管理员 / 宿管 / 学生），覆盖住宿、报修、访客、卫生、违纪、水电费、公告、调宿全流程。

![登录页](docs/images/login.png)

## 一、功能概览

| 模块 | 管理员 | 宿管 | 学生 |
| --- | :---: | :---: | :---: |
| 数据看板（入住率、报修、访客、欠费） | ✅ | ✅（本楼栋） | — |
| 楼栋管理 | ✅ 增删改查 | 查看 | — |
| 宿舍管理 | ✅ 增删改查 | 本楼栋增改 | — |
| 学生入住 / 调宿 / 退宿 | ✅ | 本楼栋 | 查看自己的宿舍 |
| 报修管理 | ✅ 派单 / 完成 | 本楼栋派单 / 完成 | 提交、查看进度、取消 |
| 访客登记 | ✅ | 本楼栋登记 / 核销 | — |
| 卫生检查 | ✅ | 本楼栋录入 | — |
| 违纪记录 | ✅ | 本楼栋录入 | — |
| 水电费 | ✅ 录入 / 收费 | 本楼栋收费 | 查看自己的账单 |
| 调宿申请 | ✅ 审批 | 涉及本楼栋的审批 | 提交申请 |
| 公告通知 | ✅ 发布 / 编辑 | 查看 | 查看 |
| 账号管理 / 重置密码 | ✅ | — | 修改自己的密码 |
| Excel 导出（学生名单、报修记录） | ✅ | ✅ | — |

## 二、技术栈

| 层次 | 技术 |
| --- | --- |
| 后端 | SpringBoot 2.7.3、MyBatis-Plus 3.5.2、MySQL 8.0、JWT（jjwt）、BCrypt、EasyExcel |
| 前端 | Vue 3.4、Vite 5、Vue Router 4、Pinia、Element Plus 2.4、ECharts 5、Axios |
| 工具 | Maven 3.6、JDK 11、Node 20、Navicat / MySQL CLI、Postman / 内置冒烟脚本 |

## 三、目录结构

```
dorm-management/
├── backend/                     # SpringBoot 后端
│   ├── pom.xml
│   └── src/main/java/com/dorm/
│       ├── common/              # Result、全局异常、JWT、登录上下文、鉴权拦截器
│       ├── config/              # 跨域、拦截器注册、MyBatis-Plus 分页、BCrypt
│       ├── controller/          # 14 个 REST 控制器
│       ├── dto/                 # 登录、派单、审批等请求对象
│       ├── entity/              # 11 张表对应实体
│       ├── mapper/              # MyBatis-Plus Mapper（含联表查询）
│       └── service/             # 业务逻辑（入住、报修流转、调宿审批、导出等）
├── frontend/                    # Vue3 前端
│   └── src/
│       ├── api/                 # 接口封装
│       ├── layout/              # 侧边栏 + 顶栏布局（按角色过滤菜单）
│       ├── router/              # 路由与登录/角色守卫
│       ├── stores/              # Pinia 用户状态
│       ├── utils/request.js     # axios 封装、token 注入、文件下载
│       └── views/               # 14 个页面
├── sql/
│   ├── schema.sql               # 建库建表（含外键、索引、注释）
│   └── data.sql                 # 初始化数据（3 栋楼 / 108 间宿舍 / 180 名学生等）
├── docs/                        # 需求、数据库、接口、测试、部署、课程设计报告
└── scripts/
    ├── api-smoke.ps1            # 接口冒烟测试（PowerShell）
    └── serve-dist.cjs           # 无 Nginx 时的静态托管 + /api 反向代理
```

## 四、快速开始

### 1. 准备环境

- JDK 11+、Maven 3.6+、Node.js 18+、MySQL 8.0+
- 数据库默认连接：`127.0.0.1:3306/dorm_db`，账号 `root`，密码 `1234`

如需改成自己的数据库账号，修改 `backend/src/main/resources/application.yml`，或使用环境变量覆盖：

```bash
DB_HOST=127.0.0.1 DB_PORT=3306 DB_NAME=dorm_db DB_USERNAME=root DB_PASSWORD=你的密码
```

### 2. 初始化数据库

```bash
mysql -uroot -p < sql/schema.sql
mysql -uroot -p < sql/data.sql
```

> 注意：`schema.sql` 会先 `DROP DATABASE IF EXISTS dorm_db`，请勿在有数据的库上直接执行。

### 3. 启动后端

```bash
cd backend
mvn clean package -DskipTests
java -jar target/dorm-management.jar
```

后端地址：<http://127.0.0.1:8080>，健康检查：<http://127.0.0.1:8080/api/health>

### 4. 启动前端

开发模式（推荐）：

```bash
cd frontend
npm install          # 首次运行
npm run dev
```

访问 <http://127.0.0.1:5173>，Vite 已配置 `/api` 代理到 8080。

生产构建 + 本地预览：

```bash
cd frontend
npm run build
cd ..
node scripts/serve-dist.cjs 5173 http://127.0.0.1:8080
```

### 5. 演示账号

| 角色 | 账号 | 密码 | 说明 |
| --- | --- | --- | --- |
| 管理员 | `admin` | `123456` | 全部功能 |
| 宿管 | `manager01` | `123456` | 负责 A 栋（听雨楼） |
| 宿管 | `manager02` | `123456` | 负责 B 栋（疏影楼） |
| 宿管 | `manager03` | `123456` | 负责 C 栋（凌云楼） |
| 学生 | `20230001` ~ `20230180` | `123456` | 账号即学号 |

## 五、页面预览

| 数据看板 | 学生入住 |
| --- | --- |
| ![看板](docs/images/dashboard.png) | ![学生入住](docs/images/students.png) |

学生端「我的宿舍」：

![我的宿舍](docs/images/my-dorm.png)

## 六、接口冒烟测试

后端启动后，在项目根目录执行：

```powershell
powershell -ExecutionPolicy Bypass -File scripts\api-smoke.ps1
```

脚本会以三种角色登录，依次验证看板、各模块列表、学生提交报修/调宿、权限拦截（403）与未登录拦截（401）。

> 脚本会写入一条测试报修和一条测试调宿申请，测试后可在数据库中按 `冒烟测试` 关键字清理。

## 七、文档

| 文档 | 内容 |
| --- | --- |
| [需求分析](docs/01-需求分析.md) | 角色、用例、功能清单、业务规则、非功能需求 |
| [数据库设计](docs/02-数据库设计.md) | ER 关系、11 张表结构、索引与外键、状态字典 |
| [接口文档](docs/03-接口文档.md) | 统一约定、40+ 接口、请求/响应示例 |
| [测试文档](docs/04-测试.md) | 测试策略、用例表、缺陷记录、测试结论 |
| [部署说明](docs/05-部署说明.md) | 本地运行、jar 部署、Nginx 配置、常见问题 |
| [课程设计报告](docs/06-课程设计报告.md) | 可直接改写为论文/报告的完整章节 |
| [课程设计报告（Word）](docs/宿舍管理系统-课程设计报告.docx) | 已排版好的 Word 版，含封面信息、表格与页面截图，可直接提交 |

## 八、已知边界与可扩展点

- 图片上传：报修表预留 `images` 字段，前端以图片链接方式录入；接入对象存储后可替换为上传组件。
- 微信小程序端：后端接口与角色模型可直接复用，使用 UniApp 实现学生端即可。
- 缓存与验证码：可引入 Redis 做 token 黑名单、登录验证码、看板缓存。
- 消息通知：报修派单、调宿审批可接入站内信 / 邮件 / 企业微信推送。

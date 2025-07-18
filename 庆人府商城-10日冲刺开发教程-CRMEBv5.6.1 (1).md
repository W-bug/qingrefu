# 庆人府商城 - 10日冲刺开发教程 (CRMEB v5.6.1 精准版)

**欢迎来到《庆人府商城》实战开发教程！**

本教程专为初学者设计，旨在通过10天的强化训练，指导你从零开始，基于经典稳定的 **CRMEB v5.6.1** 框架，完整地二次开发出具备合伙人、分销、礼品卡、先用后付等核心功能的“庆人府商城”。

**学习前提:**
*   了解基本的PHP和MySQL知识。
*   有耐心，有毅力，准备好迎接挑战。
*   严格按照每天的步骤操作。

**技术栈 (CRMEB v5.6.1 标准):**
*   **后端:** ThinkPHP 6.0 + **PHP 7.4** + MySQL 5.7+ (推荐8.0) + Redis 6.0+
*   **前端:** Vue.js + ElementUI (后台) / UniApp (小程序)
*   **开发环境:** 宝塔面板 (BT.cn) on Linux (推荐CentOS 7.x)

---

### **Day 1: 环境搭建与项目初始化 (精准版)**

**目标:** 建立一个与CRMEB v5.6.1完全兼容的开发环境，并成功安装和运行。

**任务:**

1.  **服务器准备与宝塔面板安装**
    *   准备一台云服务器 (推荐阿里云/腾讯云，配置至少2核4G内存)。
    *   通过SSH连接服务器，执行宝塔面板安装命令 (请从 [BT.cn](https://www.bt.cn)官网获取最新命令)。
    *   安装完成后，记录下你的宝塔面板地址、用户名和密码。

2.  **安装核心软件 (LNMP)**
    *   登录宝塔面板。
    *   在软件商店中，搜索并安装以下软件（选择编译安装以获得最佳性能）：
        *   **Nginx:** 1.20+
        *   **MySQL:** 8.0+ (或 5.7)
        *   **PHP:** **7.4** (这是CRMEB v5.6.1的最佳兼容版本)
        *   **Redis:** 6.0+
        *   **phpMyAdmin**

3.  **PHP 7.4 环境配置**
    *   在宝塔面板 -> 软件商店 -> PHP-7.4设置中：
        *   **安装扩展:** `fileinfo`, `redis`, `swoole4` (非常重要，用于队列和定时任务)。
        *   **禁用函数:** 检查并**删除**此列表中的 `exec`, `shell_exec`, `proc_open`，这些是CRMEB正常运行所必需的。
        *   **配置修改:** 调整 `max_execution_time` 为 `300`，`memory_limit` 为 `512M`。

4.  **创建网站与数据库**
    *   **网站:** 宝塔面板 -> 网站 -> 添加站点。
        *   域名: 填写你的IP地址或一个测试域名。
        *   根目录: 会自动创建，例如 `/www/wwwroot/your.domain.com`。
        *   数据库: 选择“创建MySQL数据库”，并记下数据库名、用户名和密码。
    *   **网站目录:** 点击网站名，进入设置。在“网站目录”中，将运行目录设置为 `/public` 并保存。
    *   **伪静态:** 在“伪静态”中，选择 `thinkphp` 规则并保存。
    *   **SSL证书:** (可选，但推荐) 在“SSL”中，申请免费的Let's Encrypt证书并开启“强制HTTPS”。

5.  **上传并安装 CRMEB v5.6.1**
    *   从官方渠道获取CRMEB v5.6.1源码。
    *   通过宝塔面板的文件管理器，上传源码压缩包到网站根目录并解压。
    *   **设置权限:** 确保网站根目录的所有文件和文件夹用户组为 `www`，权限为 `755`。`runtime` 目录需要 `777` 权限。
    *   **安装:** 访问 `http://你的域名/install`，按照提示填写数据库信息，完成安装。

6.  **配置队列与定时任务**
    *   **队列:**
        *   修改项目根目录下的 `.env` 文件，确保Redis配置正确。
        *   修改 `config/queue.php`，确保默认驱动为 `redis`。
    *   **定时任务 (Crontab):**
        *   在宝塔面板 -> 计划任务中，添加一个Shell脚本任务。
        *   任务类型: Shell脚本
        *   任务名称: CRMEB队列消费
        *   执行周期: 每分钟 (`N分钟` -> `1`)
        *   脚本内容 (请将`/www/wwwroot/你的网站目录` 替换为实际路径):
          ```bash
          #!/bin/bash
          PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
          export PATH
          cd /www/wwwroot/你的网站目录
          php think queue:listen --queue
          ```

**验收:**
*   可以正常访问后台 `http://你的域名/admin`。
*   可以正常访问前台 `http://你的域名`。
*   计划任务执行日志无明显错误。

---

### **Day 2: 数据库设计与核心模型创建**

**目标:** 根据《庆人府开发方案》，创建新增业务所需的数据库表和对应的模型(Model)、数据访问对象(DAO)。

**任务:**

1.  **执行SQL创建数据表**
    *   登录宝塔面板 -> 数据库 -> phpMyAdmin。
    *   选择你的CRMEB数据库，点击 "SQL" 标签页。
    *   将以下SQL语句分批粘贴并执行，创建所有新表。

    ```sql
    -- 合伙人信息表
    CREATE TABLE `eb_partner_info` (
      `id` int(11) NOT NULL AUTO_INCREMENT, `user_id` int(11) NOT NULL, `partner_level` tinyint(2) NOT NULL DEFAULT '1', `start_time` int(11) NOT NULL, `expire_time` int(11) NOT NULL, `credit_limit` decimal(10,2) DEFAULT '0.00', `total_commission` decimal(10,2) DEFAULT '0.00', `current_commission` decimal(10,2) DEFAULT '0.00', `status` tinyint(2) DEFAULT '1', `create_time` int(11) NOT NULL, `update_time` int(11) NOT NULL, PRIMARY KEY (`id`), UNIQUE KEY `user_id` (`user_id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='合伙人信息表';

    -- 合伙人配置表
    CREATE TABLE `eb_partner_config` (
      `id` int(11) NOT NULL AUTO_INCREMENT, `level` tinyint(2) NOT NULL, `commission_rate` decimal(5,2) NOT NULL DEFAULT '0.00', `discount_rate` decimal(5,2) NOT NULL DEFAULT '100.00', `validity_period` int(11) NOT NULL DEFAULT '365', `credit_limit` decimal(10,2) NOT NULL DEFAULT '0.00', `upgrade_condition` text, `is_enabled` tinyint(1) DEFAULT '1', `create_time` int(11) NOT NULL, `update_time` int(11) NOT NULL, PRIMARY KEY (`id`), UNIQUE KEY `level` (`level`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='合伙人配置表';

    -- 礼品卡管理表
    CREATE TABLE `eb_gift_card` (
      `id` int(11) NOT NULL AUTO_INCREMENT, `card_code` varchar(32) NOT NULL, `card_password` varchar(64) NOT NULL, `card_type` tinyint(2) NOT NULL DEFAULT '1', `product_id` int(11) NOT NULL, `user_id` int(11) DEFAULT NULL, `status` tinyint(2) DEFAULT '1', `expire_time` int(11) DEFAULT NULL, `use_time` int(11) DEFAULT NULL, `create_time` int(11) NOT NULL, `update_time` int(11) NOT NULL, PRIMARY KEY (`id`), UNIQUE KEY `card_code` (`card_code`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='礼品卡管理表';

    -- 用户信用信息表
    CREATE TABLE `eb_user_credit` (
      `id` int(11) NOT NULL AUTO_INCREMENT, `user_id` int(11) NOT NULL, `credit_limit` decimal(10,2) NOT NULL DEFAULT '0.00', `used_credit` decimal(10,2) NOT NULL DEFAULT '0.00', `available_credit` decimal(10,2) NOT NULL DEFAULT '0.00', `status` tinyint(2) DEFAULT '1', `create_time` int(11) NOT NULL, `update_time` int(11) NOT NULL, PRIMARY KEY (`id`), UNIQUE KEY `user_id` (`user_id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户信用信息表';

    -- 信用订单表
    CREATE TABLE `eb_credit_order` (
      `id` int(11) NOT NULL AUTO_INCREMENT, `order_id` varchar(32) NOT NULL, `user_id` int(11) NOT NULL, `credit_amount` decimal(10,2) NOT NULL, `repayment_time` int(11) NOT NULL, `status` tinyint(2) DEFAULT '1', `create_time` int(11) NOT NULL, `update_time` int(11) NOT NULL, PRIMARY KEY (`id`), UNIQUE KEY `order_id` (`order_id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='信用订单表';

    -- 分销关系表
    CREATE TABLE `eb_distribution_relation` (
      `id` int(11) NOT NULL AUTO_INCREMENT, `share_user_id` int(11) NOT NULL, `share_order_id` varchar(32) NOT NULL, `buyer_user_id` int(11) NOT NULL, `buyer_order_id` varchar(32) NOT NULL, `is_valid` tinyint(1) DEFAULT '1', `create_time` int(11) NOT NULL, PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='分销关系表';

    -- 分销奖励记录表
    CREATE TABLE `eb_distribution_reward` (
      `id` int(11) NOT NULL AUTO_INCREMENT, `user_id` int(11) NOT NULL, `share_order_id` varchar(32) NOT NULL, `reward_type` tinyint(2) NOT NULL, `reward_amount` decimal(10,2) DEFAULT '0.00', `gift_card_ids` text, `status` tinyint(2) DEFAULT '1', `create_time` int(11) NOT NULL, `update_time` int(11) NOT NULL, PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='分销奖励记录表';
    ```

2.  **创建模型 (Model)**
    *   在 `app/model` 目录下，为每个新表创建对应的模型文件。
    *   **示例:** 创建 `app/model/partner/PartnerInfo.php`
        ```php
        <?php
        namespace app\model\partner;
        use crmeb\basic\BaseModel;
        class PartnerInfo extends BaseModel {
            protected $pk = 'id';
            protected $name = 'partner_info';
        }
        ```
    *   **今日任务:** 按照示例，依次创建以下模型文件：
        *   `app/model/partner/PartnerConfig.php`
        *   `app/model/activity/GiftCard.php`
        *   `app/model/user/UserCredit.php`
        *   `app/model/order/CreditOrder.php`
        *   `app/model/distribution/DistributionRelation.php`
        *   `app/model/distribution/DistributionReward.php`

3.  **创建数据访问对象 (DAO)**
    *   在 `app/dao` 目录下，为每个模型创建对应的DAO文件。DAO层负责数据的增删改查。
    *   **示例:** 创建 `app/dao/partner/PartnerInfoDao.php`
        ```php
        <?php
        namespace app\dao\partner;
        use app\dao\BaseDao;
        use app\model\partner\PartnerInfo;
        class PartnerInfoDao extends BaseDao {
            protected function setModel(): string {
                return PartnerInfo::class;
            }
        }
        ```
    *   **今日任务:** 按照示例，依次创建以下DAO文件：
        *   `app/dao/partner/PartnerConfigDao.php`
        *   `app/dao/activity/GiftCardDao.php`
        *   `app/dao/user/UserCreditDao.php`
        *   `app/dao/order/CreditOrderDao.php`
        *   `app/dao/distribution/DistributionRelationDao.php`
        *   `app/dao/distribution/DistributionRewardDao.php`

**验收:**
*   所有新数据表均已在数据库中创建成功。
*   所有对应的Model和DAO文件都已在项目中正确创建，且无语法错误。

---

### **Day 3-4: 合伙人系统核心功能开发**

**目标:** 完成合伙人系统的后台管理功能，包括等级配置、合伙人列表查看。

**任务 (Day 3): 后台菜单、控制器与服务**

1.  **创建后台菜单**
    *   登录后台 `http://你的域名/admin`。
    *   进入 `设置` -> `后台菜单` -> `添加规则`。
    *   **父级:** `营销`
    *   **类型:** 菜单
    *   **名称:** `合伙人管理`
    *   **路由:** `partner.partner/index`
    *   **图标:** `el-icon-s-custom`
    *   **排序:** 5
    *   在"合伙人管理"下再添加两个子菜单：
        *   **名称:** `合伙人列表`, **路由:** `partner.partner/index`
        *   **名称:** `等级配置`, **路由:** `partner.partner_config/index`

2.  **创建控制器 (Controller)**
    *   在 `app/adminapi/controller/partner` 目录下创建 `PartnerController.php` 和 `PartnerConfigController.php`。
    *   `PartnerController.php`:
        ```php
        <?php
        namespace app\adminapi\controller\partner;
        use app\adminapi\controller\AuthController;
        use app\services\partner\PartnerServices;
        use think\facade\App;
        class PartnerController extends AuthController {
            public function __construct(App $app, PartnerServices $services) {
                parent::__construct($app);
                $this->services = $services;
            }
            public function index() {
                $where = $this->request->getMore([['level', ''], ['nickname', ''], ['page', 1], ['limit', 20]]);
                return app('json')->success($this->services->getPartnerList($where));
            }
        }
        ```
    *   `PartnerConfigController.php`:
        ```php
        <?php
        namespace app\adminapi\controller\partner;
        use app\adminapi\controller\AuthController;
        use app\services\partner\PartnerConfigServices;
        use think\facade\App;
        class PartnerConfigController extends AuthController {
            public function __construct(App $app, PartnerConfigServices $services) {
                parent::__construct($app);
                $this->services = $services;
            }
            public function index() {
                return app('json')->success($this->services->getConfigList());
            }
            public function save() {
                $data = $this->request->postMore([['level', 0], ['commission_rate', 0], ['discount_rate', 100], ['credit_limit', 0]]);
                if (!$data['level']) return app('json')->fail('等级不能为空');
                $this->services->saveConfig($data['level'], $data);
                return app('json')->success('保存成功');
            }
        }
        ```

3.  **创建服务层 (Service)**
    *   在 `app/services/partner` 目录下创建 `PartnerServices.php` 和 `PartnerConfigServices.php`。
    *   `PartnerServices.php`:
        ```php
        <?php
        namespace app\services\partner;
        use app\services\BaseServices;
        use app\dao\partner\PartnerInfoDao;
        class PartnerServices extends BaseServices {
            public function __construct(PartnerInfoDao $dao) {
                $this->dao = $dao;
            }
            public function getPartnerList(array $where) {
                $query = $this->dao->getModel()->alias('p')->join('user u', 'p.user_id = u.uid')->field('p.*, u.nickname, u.avatar');
                if ($where['level']) $query->where('p.partner_level', $where['level']);
                if ($where['nickname']) $query->where('u.nickname', 'like', "%{$where['nickname']}%");
                $list = $query->order('p.id desc')->page($where['page'], $where['limit'])->select()->toArray();
                $count = $this->dao->getModel()->alias('p')->join('user u', 'p.user_id = u.uid')->where($where)->count();
                return compact('list', 'count');
            }
        }
        ```
    *   `PartnerConfigServices.php`:
        ```php
        <?php
        namespace app\services\partner;
        use app\services\BaseServices;
        use app\dao\partner\PartnerConfigDao;
        class PartnerConfigServices extends BaseServices {
            public function __construct(PartnerConfigDao $dao) {
                $this->dao = $dao;
            }
            public function getConfigList() {
                return $this->dao->selectList([], '*', 0, 0, 'level asc')->toArray();
            }
            public function saveConfig(int $level, array $data) {
                $config = $this->dao->getOne(['level' => $level]);
                if ($config) {
                    return $this->dao->update($config['id'], $data);
                } else {
                    return $this->dao->save($data);
                }
            }
        }
        ```

**任务 (Day 4): 后台前端页面**

1.  **理解后台开发流程:** CRMEB后台是基于Vue+ElementUI的。你需要找到 `template/admin` 目录下的前端源码，修改后编译，再将编译好的 `dist` 目录内容覆盖到项目 `public/admin` 目录。
2.  **合伙人列表页面 (`views/partner/partner/index.vue`)**
    *   在后台源码的 `src/views` 下创建 `partner/partner` 目录，并新建 `index.vue` 文件。
    *   内容（简化版，核心是表格和API请求）:
        ```vue
        <template>
          <div>
            <el-card>
              <el-table :data="tableData">
                <el-table-column prop="user_id" label="用户ID"></el-table-column>
                <el-table-column prop="nickname" label="昵称"></el-table-column>
                <el-table-column prop="partner_level" label="等级"></el-table-column>
                <el-table-column prop="credit_limit" label="信用额度"></el-table-column>
              </el-table>
              <div class="block">
                <el-pagination layout="prev, pager, next" :total="total" :page-size="searchParams.limit" @current-change="pageChange"></el-pagination>
              </div>
            </el-card>
          </div>
        </template>
        <script>
        import { getPartnerListApi } from '@/api/partner'; // 假设API已定义
        export default {
          data() {
            return { tableData: [], total: 0, searchParams: { page: 1, limit: 20 } };
          },
          mounted() { this.getList(); },
          methods: {
            getList() {
              getPartnerListApi(this.searchParams).then(res => {
                this.tableData = res.data.list;
                this.total = res.data.count;
              });
            },
            pageChange(page) {
              this.searchParams.page = page;
              this.getList();
            }
          }
        }
        </script>
        ```
3.  **等级配置页面 (`views/partner/partner_config/index.vue`)**
    *   类似地，创建 `partner/partner_config/index.vue`。
    *   内容（简化版，核心是表单和保存逻辑）:
        ```vue
        <template>
          <div>
            <el-form v-for="(item, index) in form" :key="index">
              <el-form-item :label="'等级 ' + item.level"></el-form-item>
              <el-form-item label="折扣率"> <el-input v-model="item.discount_rate"></el-input> </el-form-item>
              <el-form-item label="佣金率"> <el-input v-model="item.commission_rate"></el-input> </el-form-item>
              <el-form-item label="信用额度"> <el-input v-model="item.credit_limit"></el-input> </el-form-item>
              <el-button type="primary" @click="onSubmit(item)">保存</el-button>
            </el-form>
          </div>
        </template>
        <script>
        import { getConfigListApi, saveConfigApi } from '@/api/partner';
        export default {
          data() { return { form: [] }; },
          mounted() {
            getConfigListApi().then(res => { this.form = res.data; });
          },
          methods: {
            onSubmit(item) {
              saveConfigApi(item).then(() => { this.$message.success('保存成功'); });
            }
          }
        }
        </script>
        ```
4.  **编译和部署后台**
    *   在后台源码目录下运行 `npm run build`。
    *   将生成的 `dist` 文件夹内所有内容，复制到服务器项目 `public/admin/` 目录下。

**验收:**
*   刷新后台，可以在“营销”菜单下看到“合伙人管理”。
*   点击“合伙人列表”和“等级配置”可以正常显示页面（即使没有数据）。
*   在等级配置页面可以尝试保存数据，并能在数据库 `eb_partner_config` 表中看到变化。

---

### **Day 5-6: 礼品卡与先用后付系统后台框架**

**目标:** 搭建礼品卡管理和信用支付管理的后台框架，实现基础的列表查看和配置功能。

**任务 (Day 5): 礼品卡系统**

1.  **后台菜单:** 在`营销`下添加`礼品卡管理`，包含子菜单`卡片列表`。
2.  **控制器 (`GiftCardController.php`):**
    *   创建 `app/adminapi/controller/activity/GiftCardController.php`。
    *   实现 `index()` 方法，用于获取卡片列表。
    *   实现 `save()` 方法，用于后台手动添加/生成礼品卡。
3.  **服务层 (`GiftCardServices.php`):**
    *   创建 `app/services/activity/GiftCardServices.php`。
    *   `getCardList()`: 实现列表查询逻辑。
    *   `createCard()`: 实现单个或批量生成礼品卡逻辑。卡号可以用 `uniqid()` 生成，密码可以用 `password_hash()` 加密。
4.  **后台页面 (`views/activity/gift_card/index.vue`):**
    *   创建Vue页面，展示礼品卡列表。
    *   包含一个表单弹窗，用于调用 `save` 接口生成新卡。
    *   编译并部署。

**任务 (Day 6): 先用后付系统**

1.  **后台菜单:** 在`用户`菜单下添加`信用管理`，包含子菜单`信用订单`和`用户信用`。
2.  **控制器:**
    *   `CreditOrderController.php`: 在 `app/adminapi/controller/order/` 下创建，实现信用订单列表的查看。
    *   `UserCreditController.php`: 在 `app/adminapi/controller/user/` 下创建，实现用户信用列表的查看和额度手动调整功能。
3.  **服务层:**
    *   `CreditOrderServices.php`: 实现信用订单列表查询。
    *   `UserCreditServices.php`: 实现用户信用列表查询和额度调整 `adjustCredit()` 方法。
4.  **后台页面:**
    *   创建 `views/order/credit_order/index.vue` 用于展示信用订单。
    *   创建 `views/user/user_credit/index.vue` 用于展示用户信用，并提供一个“调整额度”的按钮/弹窗。
    *   编译并部署。

**验收:**
*   礼品卡管理后台可以正常查看和生成礼品卡。
*   信用管理后台可以查看信用订单和用户信用列表。
*   可以成功手动调整某个用户的信用额度。

---

### **Day 7-8: 创新分销系统与小程序端开发**

**目标:** 实现分销系统的核心逻辑，并开始开发小程序端的核心页面，让用户能感知到新功能。

**任务 (Day 7): 分销核心逻辑**

1.  **分销关系绑定:**
    *   **核心:** 用户通过分享链接进入时，需要记录分享者(spread)。
    *   修改 `app/api/controller/v1/IndexController.php` 的 `index` 方法或使用中间件，检查请求中是否带有 `spread` 参数，如果有，则存入 `Session` 或 `Cookie`。
    *   用户注册或下单时，检查 `Session` 中是否存在 `spread`，如果存在，则在 `eb_user` 表中更新 `spread_uid` 字段，并可在 `eb_distribution_relation` 表中建立关系。
2.  **分销奖励服务 (`DistributionServices.php`):**
    *   创建 `app/services/distribution/DistributionServices.php`。
    *   **核心方法 `checkAndReward($order)`:** 这是整个分销系统的核心。当一个订单支付成功后，触发此方法。
        *   **步骤1:** 根据订单的 `uid`，查找其 `spread_uid` (分享者)。
        *   **步骤2:** 检查分享条件：查询 `eb_distribution_relation`，看这个 `share_order_id` 是否已经成功分享给 >=2 个人。
        *   **步骤3:** 检查价格条件：购买者的订单金额是否 >= 分享者的订单金额。
        *   **步骤4:** 判断分享者是新用户还是老用户 (通过查询用户的历史订单判断)。
        *   **步骤5:** 如果所有条件满足，调用 `giveReward()` 方法发放奖励，并更新 `eb_distribution_reward` 记录表。
3.  **订单支付成功后回调:**
    *   找到订单支付成功的回调位置（通常在 `app/services/order/StoreOrderServices.php` 的 `paySuccess` 方法中）。
    *   在末尾添加代码，调用分销服务：`app(DistributionServices::class)->checkAndReward($order);`

**任务 (Day 8): 小程序端核心页面**

1.  **准备小程序开发环境:**
    *   安装 HBuilderX。
    *   下载CRMEB小程序前端源码并导入 HBuilderX。
    *   在 `manifest.json` 中配置你的小程序AppID。
    *   在 `utils/request.js` 中配置你的API地址 `http://你的域名/api/`。

2.  **合伙人中心页面 (`pages/partner/center/index.vue`)**
    *   **API:** 创建 `app/api/controller/v1/partner/PartnerController.php`，提供 `getInfo()` 方法返回当前用户的合伙人信息。
    *   **页面:** 创建 `pages/partner/center/index.vue`，调用API，展示用户的合伙人等级、权益、佣金等信息。

3.  **礼品卡中心页面 (`pages/activity/gift_card/index.vue`)**
    *   **API:** 创建 `app/api/controller/v1/activity/GiftCardController.php`，提供 `getMyCards()` 方法返回用户的礼品卡列表，`exchangeCard()` 方法用于兑换礼品卡。
    *   **页面:** 创建 `pages/activity/gift_card/index.vue`，包含一个列表展示我的礼品卡，一个输入框和按钮用于兑换。

4.  **订单分享按钮:**
    *   在订单详情页 `pages/order_details/index.vue` 中，添加一个“分享返现”按钮。
    *   点击按钮，调用 `uni.share` 功能，分享的路径要带上参数 `?spread=当前用户ID&share_order_id=当前订单ID`。

**验收:**
*   通过分享链接注册新用户，数据库中能看到分销关系记录。
*   小程序可以运行，并能看到合伙人中心和礼品卡中心页面（即使没有数据）。
*   订单详情页出现分享按钮。

---

### **Day 9: 功能串联与流程测试**

**目标:** 将前端和后端功能串联起来，进行核心业务流程的模拟测试，发现并修复问题。

**任务:**

1.  **合伙人升级流程:**
    *   **后台:** 在`商品管理`中，为某个商品添加一个特殊标识（例如，在商品名称中加`[合伙人升级]`）。
    *   **逻辑:** 修改订单支付成功回调 `paySuccess`。
    *   **代码:**
        ```php
        // 在 StoreOrderServices.php 的 paySuccess 方法中
        $containsUpgradeProduct = false;
        foreach($order->orderProduct as $product) {
            if (strpos($product['productInfo']['store_name'], '[合伙人升级]') !== false) {
                $containsUpgradeProduct = true;
                break;
            }
        }
        if ($containsUpgradeProduct) {
            // 调用合伙人服务，将会员升级为合伙人
            // app(PartnerServices::class)->upgradeUserToPartner($order->uid, 1); // 假设升级为1级
        }
        ```
    *   **测试:** 购买该商品，检查用户是否自动成为合伙人。

2.  **礼品卡兑换流程:**
    *   **后台:** 手动生成一张礼品卡。
    *   **小程序:** 在礼品卡中心页面，输入卡号卡密，点击兑换。
    *   **逻辑:** `GiftCardController` 的 `exchangeCard` 方法接收到请求，验证卡密，如果成功，将 `eb_gift_card` 表中的 `user_id` 更新为当前用户ID。
    *   **测试:** 兑换后，在“我的礼品卡”中能看到这张卡。

3.  **信用支付流程 (模拟):**
    *   **后台:** 手动给测试用户设置一个信用额度，如500元。
    *   **小程序:** 在订单支付页面 `pages/order_confirm/index.vue`，添加一个“信用支付”的选项。
        *   **条件:** `v-if="userInfo.is_partner && userCredit.available_credit > 0"`。
    *   **逻辑:** 用户选择信用支付后，调用一个新的API `order/credit_pay`。该API会：
        1. 检查用户可用额度是否足够。
        2. 创建普通订单，但标记为“待支付”。
        3. 在 `eb_credit_order` 表中创建一条信用订单记录。
        4. 更新用户已用额度和可用额度。
        5. 调用 `paySuccess` 逻辑，使订单状态变为“待发货”。
    *   **测试:** 合伙人用户在额度内可以使用信用支付成功下单。

4.  **分销奖励流程 (完整测试):**
    *   **用户A** (老用户) 下单，获得订单A。
    *   **用户A** 分享订单A的链接。
    *   **用户B** 通过链接进入并下单，生成订单B。
    *   **用户C** 通过链接进入并下单，生成订单C。
    *   **检查:** 订单C支付成功后，系统是否自动给用户A发放了奖励（老用户奖励：发放商品礼品券）。检查 `eb_distribution_reward` 表是否有记录。

**验收:**
*   所有核心流程（合伙人升级、礼品卡兑换、信用支付、分销奖励）都能跑通。
*   数据库中的数据状态符合预期。

---

### **Day 10: 优化、安全加固与部署**

**目标:** 对项目进行最终的优化和安全检查，准备上线。

**任务:**

1.  **性能优化:**
    *   **缓存:** 在服务层(Service)的关键查询方法（如获取合伙人配置、获取商品详情等）中加入Redis缓存。
        ```php
        // 示例: PartnerConfigServices.php
        use think\facade\Cache;
        public function getConfigList() {
            return Cache::remember('PARTNER_CONFIG_LIST', function() {
                return $this->dao->selectList([], '*', 0, 0, 'level asc')->toArray();
            }, 3600); // 缓存1小时
        }
        ```
    *   **索引:** 使用phpMyAdmin检查所有新表的查询字段，确保关键字段（如`user_id`, `status`, `level`）都已建立索引。

2.  **安全加固:**
    *   **输入验证:** 检查所有后台保存接口，确保使用了 `request->postMore()` 并对关键参数做了类型和存在性校验。
    *   **权限检查:** 确保所有后台控制器都继承了 `AuthController`。
    *   **敏感数据:** 确认礼品卡密码等信息在存入数据库前已经过哈希处理。
    *   **防刷:** 在关键API（如礼品卡兑换、信用支付）的控制器方法开头加入频率限制。
        ```php
        // 示例: GiftCardController.php 的 exchangeCard 方法
        $uid = $this->request->uid();
        $lockKey = 'exchange_card_lock_' . $uid;
        if (Cache::has($lockKey)) {
            return app('json')->fail('操作过于频繁，请稍后再试');
        }
        Cache::set($lockKey, 1, 10); // 锁定10秒
        ```

3.  **清理与部署:**
    *   删除安装目录 `install`。
    *   在 `.env` 文件中设置 `APP_DEBUG = false`。
    *   重新编译并上传小程序和后台前端代码。

4.  **最终测试:**
    *   作为一个真实用户，完整地体验所有流程。
    *   注册 -> 成为合伙人 -> 使用信用支付 -> 分享订单 -> 获得奖励 -> 兑换礼品卡。

**验收:**
*   项目运行流畅，关键操作响应迅速。
*   安全措施已部署，无法进行恶意高频操作。
*   项目已达到可上线状态。

---

**恭喜你！**

经过10天的努力，你已经成功完成了《庆人府商城》的开发。这不仅是一个功能强大的电商项目，更是你技术能力的一次飞跃。继续探索，不断优化，你的商城将会更加出色！
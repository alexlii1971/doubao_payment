# WordPress Multisite 支付插件部署文档

## 一、概述
本部署文档旨在指导用户在已安装好的 WordPress Multisite 环境中部署支付插件。该插件支持与 WooCommerce 集成，提供多种支付方式，如微信支付和支付宝支付，同时具备分账、退款、争议处理和审计日志等功能。此文档以网站域名 `tao.ooo` 为例进行说明。

## 二、环境要求
### 服务器环境
- **操作系统**：Linux（推荐 Ubuntu 18.04 及以上）、Windows Server 2016 及以上
- **Web 服务器**：Apache 2.4 及以上 或 Nginx 1.14 及以上
- **数据库**：MySQL 5.6 及以上 或 MariaDB 10.1 及以上
- **PHP**：7.2 及以上

### WordPress 环境
- **WordPress 版本**：5.0 及以上
- **WooCommerce 插件**：需提前在 Multisite 中网络激活

## 三、部署步骤

### 1. 下载插件
从代码仓库或插件市场下载 `wp - multisite - payment - plugin` 插件压缩包。

### 2. 上传插件到 Multisite
#### 2.1 访问网络管理后台
使用具有网络管理员权限的账号登录 WordPress Multisite 的网络管理后台。访问地址为 `https://tao.ooo/wp-admin/network/`。

#### 2.2 上传插件
- 导航到“网络管理” -> “插件” -> “添加新插件”。
- 点击“上传插件”按钮，选择之前下载的插件压缩包，然后点击“安装现在”。

### 3. 网络激活插件
安装完成后，在插件列表中找到 `wp - multisite - payment - plugin`，点击“网络激活”。这将使该插件在整个 Multisite 网络中可用。

### 4. 配置插件

#### 4.1 基础配置
- 导航到“网络管理” -> “设置” -> “支付设置”。
- 配置微信支付和支付宝支付的相关信息，如商户 ID、证书、应用 ID 和私钥等。这些信息可在相应支付平台的商户后台获取。

#### 4.2 缓存配置
插件使用文件系统作为缓存存储，缓存目录默认为插件目录下的 `cache` 文件夹。此缓存机制主要用于存储一些频繁访问且相对稳定的数据，例如支付 SDK 实例、配置信息等，以提高插件的响应速度和减少服务器负载。

##### 缓存目录权限设置
确保 `cache` 目录具有读写权限，不同操作系统的设置方式有所不同：

**Linux 系统**
可以使用以下命令为 `cache` 目录赋予读写权限：
```bash
chmod -R 755 /path/to/your/plugin/wp - multisite - payment - plugin/cache
```
如果需要修改目录的所有者和所属组，可以使用以下命令：
```bash
chown -R www - data:www - data /path/to/your/plugin/wp - multisite - payment - plugin/cache
```
其中 `www - data` 是常见的 Web 服务器用户和组，具体可能因服务器配置而异。

**Windows 系统**
- 右键点击 `cache` 文件夹，选择“属性”。
- 在“安全”选项卡中，确保当前运行 Web 服务器的用户（如 `IIS_IUSRS` 或 `Network Service`）具有“读取”和“写入”权限。

##### 缓存文件清理
在某些情况下，可能需要手动清理缓存文件，例如缓存数据过期或出现异常时。可以通过以下步骤进行清理：
- 进入 `cache` 目录，删除其中的所有文件。
```bash
rm -rf /path/to/your/plugin/wp - multisite - payment - plugin/cache/*
```
注意：清理缓存文件后，插件在下次访问相关数据时会重新生成缓存。

#### 4.3 数据库配置
插件依赖 WordPress 的 `$wpdb` 进行数据库操作，无需额外配置。但需确保数据库正常运行且具有足够的权限。

#### 4.4 `config.json` 配置
`config.json` 文件位于插件的 `config` 目录下，主要用于存储各子站点的支付相关配置信息，为插件的正常运行提供必要的参数。

##### 作用
- **存储支付配置信息**：包含不同支付方式（如微信支付、支付宝支付）的关键配置参数，如商户 ID、应用 ID、密钥、支付网关 URL、回调地址等。这些信息是插件与支付平台进行交互的基础，插件在处理支付、退款等操作时，会根据这些配置信息与支付平台进行通信。
- **多站点配置管理**：在 WordPress Multisite 环境中，不同子站点可能有不同的支付需求。`config.json` 可以为每个子站点单独配置支付信息，通过子站点的唯一标识（如站点 ID 或域名）进行区分，实现多站点的差异化管理。
- **配置灵活性和可扩展性**：使用 `config.json` 存储配置信息，方便动态修改配置，无需修改插件代码。当支付平台的 API 地址发生变化或需要添加新的支付方式时，只需在 `config.json` 中更新或添加相应的配置项即可。

以下是一个以 `tao.ooo` 相关子站点为例的 `config.json` 文件示例：
```json
{
    "sites": {
        "tao.ooo": {
            "payment_methods": {
                "wechat_pay": {
                    "merchant_id": "1234567890",
                    "api_key": "abcdef1234567890",
                    "gateway_url": "https://api.mch.weixin.qq.com/pay/unifiedorder",
                    "notify_url": "https://tao.ooo/wechat_pay_notify"
                },
                "alipay": {
                    "app_id": "0987654321",
                    "private_key": "-----BEGIN RSA PRIVATE KEY-----\n...\n-----END RSA PRIVATE KEY-----",
                    "gateway_url": "https://openapi.alipay.com/gateway.do",
                    "notify_url": "https://tao.ooo/alipay_notify"
                }
            },
            "split_rules": {
                "platform_ratio": 0.1,
                "merchant_ratio": 0.9
            }
        },
        "subsite.tao.ooo": {
            "payment_methods": {
                "wechat_pay": {
                    "merchant_id": "0987654321",
                    "api_key": "0987654321abcdef",
                    "gateway_url": "https://api.mch.weixin.qq.com/pay/unifiedorder",
                    "notify_url": "https://subsite.tao.ooo/wechat_pay_notify"
                }
            },
            "split_rules": {
                "platform_ratio": 0.2,
                "merchant_ratio": 0.8
            }
        }
    }
}
```

##### 注意事项
- **安全性**
    - **敏感信息保护**：`config.json` 中包含大量敏感信息，如商户密钥、私钥等，需严格保护。设置文件权限，避免公共访问。例如，在 Linux 系统中，可将文件权限设置为 `600`，仅允许文件所有者读写。还可考虑对敏感信息进行加密存储，使用对称加密算法（如 AES）加密密钥，在插件使用时再解密。
    - **防止篡改**：使用哈希算法（如 MD5、SHA - 256）计算文件的哈希值，并在插件启动时验证哈希值是否匹配，以防止文件被恶意篡改。
- **格式正确性**
    - **JSON 语法规范**：`config.json` 必须遵循 JSON 语法规范，如 JSON 对象的键和字符串值使用双引号，数组元素用逗号分隔等。
    - **验证机制**：在插件中添加对 `config.json` 文件的验证机制，使用 JSON 解析库（如 PHP 的 `json_decode` 函数）解析文件，若解析失败，提示用户检查文件格式。
- **版本兼容性**
    - **配置项更新**：随着插件更新，`config.json` 中的配置项可能变化。更新插件时，确保 `config.json` 文件版本与插件版本兼容。
    - **配置迁移**：若插件升级需添加或修改配置项，提供配置迁移脚本或工具，帮助用户将旧版本的 `config.json` 文件迁移到新版本。
    - **版本检查**：插件启动时，检查 `config.json` 文件的版本信息，若版本不兼容，提示用户进行相应处理。
- **备份与恢复**
    - **定期备份**：由于 `config.json` 包含重要配置信息，需定期备份。可使用自动化脚本或工具（如 cron 任务）定期备份到安全位置。
    - **恢复机制**：若 `config.json` 文件丢失或损坏，提供恢复机制。可使用备份文件恢复，或提供默认配置模板，让用户重新配置。

### 5. 数据库表创建
插件会在激活时自动尝试创建所需的数据库表，如审计日志表、争议处理表等。若未自动创建，可手动执行 SQL 脚本创建。以下是可能需要创建的表结构示例：

#### 审计日志表
```sql
CREATE TABLE IF NOT EXISTS `wp_wmp_audit_log` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `order_id` int(11) NOT NULL,
    `action_type` varchar(50) NOT NULL,
    `operator` varchar(100) NOT NULL,
    `action_time` datetime NOT NULL,
    `details` text,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

#### 争议处理表
```sql
CREATE TABLE IF NOT EXISTS `wp_wmp_dispute` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `order_id` int(11) NOT NULL,
    `dispute_reason` text NOT NULL,
    `submit_time` datetime NOT NULL,
    `status` varchar(20) NOT NULL,
    `handling_result` text,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

#### 资金流水表
```sql
CREATE TABLE IF NOT EXISTS `wp_wmp_fund_flow` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `order_id` int(11) NOT NULL,
    `amount` decimal(10, 2) NOT NULL,
    `flow_type` varchar(20) NOT NULL,
    `transaction_time` datetime NOT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

#### 分账回滚表
```sql
CREATE TABLE IF NOT EXISTS `wp_wmp_split_rollback` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `order_id` int(11) NOT NULL,
    `split_amount` decimal(10, 2) NOT NULL,
    `rollback_time` datetime NOT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 6. 语言配置
- 插件支持多语言，语言文件位于 `languages` 文件夹中。
- 若需要使用其他语言，可将对应的 `.po` 和 `.mo` 文件上传到 WordPress Multisite 的网络语言文件夹（通常为 `wp - content/languages/plugins`）中。

### 7. 子站点配置（可选）
如果需要为不同的子站点设置不同的支付配置，可以在每个子站点的管理后台进行单独配置。步骤如下：
- 登录子站点的管理后台（如 `https://subsite.tao.ooo/wp-admin/`）。
- 导航到“设置” -> “支付设置”，根据子站点的需求修改支付配置信息。

## 四、测试
### 1. 支付功能测试
#### 1.1 微信支付测试
- 在子站点创建一个测试订单，选择微信支付作为支付方式。
- 点击“支付”按钮，检查是否能正确跳转到微信支付页面。
- 使用微信支付测试账号完成支付，检查订单状态是否更新为“已支付”。

#### 1.2 支付宝支付测试
- 重复微信支付测试步骤，选择支付宝支付。
- 检查是否能正确跳转到支付宝支付页面，完成支付后检查订单状态。

### 2. 分账功能测试
- 配置分账规则，创建一个支付订单并完成支付。
- 检查审计日志和资金流水表，确认分账操作是否正确执行。

### 3. 退款功能测试
- 选择一个已支付的订单，发起退款申请。
- 检查退款流程是否正常，订单状态是否更新为“已退款”，分账金额是否正确回滚。

### 4. 争议处理功能测试
- 以用户身份提交一个争议申请。
- 以管理员身份登录网络管理后台，处理该争议，检查争议处理记录是否正确。

## 五、常见问题及解决方法

### 1. 支付失败
- **检查支付配置**：确保微信支付和支付宝支付的配置信息正确，如商户 ID、证书、应用 ID 和私钥等。
- **检查网络连接**：确保服务器能够正常访问支付平台的 API。
- **查看支付平台报错信息**：根据支付平台返回的报错信息，排查具体问题，如签名错误、账户余额不足等。

### 2. 分账失败
- **检查分账规则**：确保分账规则配置正确，如比例是否合理，分账账户信息是否正确。
- **检查资金余额**：确保平台和商户账户有足够的资金进行分账。
- **查看支付平台分账接口文档**：根据支付平台分账接口返回的错误信息，排查分账失败的原因，如分账金额超出限制、分账账户状态异常等。

### 3. 退款失败
- **检查订单状态**：确保订单状态为已支付，且未进行过退款操作。
- **检查支付平台限制**：某些支付平台可能对退款有时间或金额限制，需检查相关规定。
- **查看支付平台退款接口文档**：根据支付平台退款接口返回的错误信息，排查退款失败的原因，如退款金额超出支付金额、退款时间已过等。

### 4. 插件无法激活
- **检查 PHP 版本**：确保服务器的 PHP 版本符合插件要求（7.2 及以上）。
- **检查文件权限**：确保插件文件和文件夹具有正确的读写权限。
- **查看错误日志**：查看 WordPress 的错误日志，根据日志中的错误信息排查插件无法激活的原因，如代码语法错误、依赖插件未安装等。

### 5. 缓存相关问题
#### 5.1 缓存目录权限问题
- **问题描述**：插件无法读写缓存文件，可能会导致部分功能异常，如支付 SDK 实例无法缓存、配置信息无法读取等。
- **解决方法**：按照上述缓存目录权限设置的方法，确保 `cache` 目录具有正确的读写权限。可以使用 `ls -l` 命令查看目录权限，使用 `chmod` 和 `chown` 命令进行修改。

#### 5.2 缓存数据过期或异常问题
- **问题描述**：缓存数据可能会因为各种原因过期或出现异常，导致插件使用到错误的数据，影响功能正常运行。
- **解决方法**：手动清理缓存文件，让插件重新生成缓存。可以通过删除 `cache` 目录下的所有文件来清理缓存。

### 6. `config.json` 相关问题
#### 6.1 配置文件格式错误
- **问题描述**：若 `config.json` 文件格式不符合 JSON 规范，插件在解析配置时会出错，导致支付、分账等功能无法正常使用。
- **解决方法**：使用在线 JSON 验证工具或代码中的验证逻辑检查文件格式，修正语法错误，如添加缺失的引号、逗号等。

#### 6.2 敏感信息泄露
- **问题描述**：`config.json` 包含商户密钥等敏感信息，若文件权限设置不当或被恶意获取，可能导致信息泄露。
- **解决方法**：确保文件权限为仅所有者可读写，对敏感信息加密存储，并定期更换密钥。

#### 6.3 版本不兼容
- **问题描述**：插件升级后 `config.json` 配置项变化，旧配置文件可能与新版本插件不兼容。
- **解决方法**：参考插件更新说明，手动更新配置文件或使用插件提供的迁移工具进行配置迁移。

## 六、维护与监控

### 1. 日志监控
定期查看插件的日志文件，及时发现和处理异常情况。日志文件可在 `includes/utils/logger.php` 中进行配置。重点关注支付失败、分账失败、退款失败等异常操作的日志记录，分析原因并及时处理。

### 2. 数据库备份
定期备份数据库，以防数据丢失。可使用 WordPress 插件或数据库管理工具进行备份。建议每周进行一次全量备份，并在重要操作（如支付、退款等）后进行增量备份。

### 3. 插件更新
关注插件的更新信息，及时更新插件以获取最新功能和修复已知问题。在更新插件之前，建议先备份数据库和插件文件，以防更新过程中出现问题。

### 4. 安全检查
定期进行安全检查，如检查服务器的安全设置、插件的漏洞情况等，确保系统安全稳定运行。可使用安全扫描工具对服务器和插件进行安全扫描，及时发现并修复安全漏洞。

## 七、附录
### 数据库表结构
| 表名 | 描述 |
| ---- | ---- |
| `wp_wmp_audit_log` | 审计日志表，记录所有支付相关操作的


### 数据库表结构（续）
| 表名 | 描述 |
| ---- | ---- |
| `wp_wmp_dispute` | 争议处理表，记录用户提交的争议信息及处理结果 |
| `wp_wmp_fund_flow` | 资金流水表，记录资金的流向和操作类型，如支付、分账、退款等 |
| `wp_wmp_split_rollback` | 分账回滚表，记录分账回滚的相关信息，如订单号、分账金额、回滚时间等 |

### 配置文件示例（config.json）
```json
{
    "sites": {
        "tao.ooo": {
            "payment_methods": {
                "wechat_pay": {
                    "merchant_id": "your_wechat_merchant_id_for_tao_ooo",
                    "api_key": "your_wechat_api_key_for_tao_ooo",
                    "cert_path": "/path/to/wechat/cert/tao_ooo.pem",
                    "key_path": "/path/to/wechat/key/tao_ooo.pem",
                    "notify_url": "https://tao.ooo/wechat_pay_notify",
                    "return_url": "https://tao.ooo/wechat_pay_return"
                },
                "alipay": {
                    "app_id": "your_alipay_app_id_for_tao_ooo",
                    "private_key": "your_alipay_private_key_for_tao_ooo",
                    "alipay_public_key": "your_alipay_public_key_for_tao_ooo",
                    "notify_url": "https://tao.ooo/alipay_notify",
                    "return_url": "https://tao.ooo/alipay_return"
                }
            },
            "split_rules": {
                "enable_split": true,
                "platform_ratio": 0.1,
                "merchant_ratio": 0.9,
                "split_accounts": [
                    {
                        "account_type": "merchant",
                        "account_id": "merchant_account_tao_ooo",
                        "ratio": 0.9
                    },
                    {
                        "account_type": "platform",
                        "account_id": "platform_account_tao_ooo",
                        "ratio": 0.1
                    }
                ]
            }
        },
        "subsite.tao.ooo": {
            "payment_methods": {
                "wechat_pay": {
                    "merchant_id": "your_wechat_merchant_id_for_subsite",
                    "api_key": "your_wechat_api_key_for_subsite",
                    "cert_path": "/path/to/wechat/cert/subsite.pem",
                    "key_path": "/path/to/wechat/key/subsite.pem",
                    "notify_url": "https://subsite.tao.ooo/wechat_pay_notify",
                    "return_url": "https://subsite.tao.ooo/wechat_pay_return"
                },
                "alipay": {
                    "app_id": "your_alipay_app_id_for_subsite",
                    "private_key": "your_alipay_private_key_for_subsite",
                    "alipay_public_key": "your_alipay_public_key_for_subsite",
                    "notify_url": "https://subsite.tao.ooo/alipay_notify",
                    "return_url": "https://subsite.tao.ooo/alipay_return"
                }
            },
            "split_rules": {
                "enable_split": false
            }
        }
    }
}
```

### 配置文件字段说明
#### 全局配置
- `sites`：一个对象，包含各个子站点的配置信息，键为子站点的域名。

#### 子站点配置
- `payment_methods`：一个对象，包含该子站点支持的支付方式的配置信息。
  - `wechat_pay`：微信支付配置。
    - `merchant_id`：微信支付商户 ID。
    - `api_key`：微信支付 API 密钥。
    - `cert_path`：微信支付证书路径。
    - `key_path`：微信支付私钥路径。
    - `notify_url`：微信支付回调通知 URL。
    - `return_url`：微信支付支付完成后返回的 URL。
  - `alipay`：支付宝支付配置。
    - `app_id`：支付宝应用 ID。
    - `private_key`：支付宝应用私钥。
    - `alipay_public_key`：支付宝公钥。
    - `notify_url`：支付宝回调通知 URL。
    - `return_url`：支付宝支付完成后返回的 URL。
- `split_rules`：分账规则配置。
  - `enable_split`：是否启用分账功能，布尔值。
  - `platform_ratio`：平台分账比例，小数。
  - `merchant_ratio`：商户分账比例，小数。
  - `split_accounts`：分账账户列表，数组。
    - `account_type`：账户类型，如 "merchant" 或 "platform"。
    - `account_id`：账户 ID。
    - `ratio`：该账户的分账比例，小数。

### 支付回调处理示例代码（PHP）
#### 微信支付回调处理
```php
<?php
// 引入必要的文件
require_once('path/to/wechat_pay_sdk.php');

// 获取微信支付回调数据
$xml = file_get_contents('php://input');

// 验证签名
$wechatPay = new WechatPay();
if ($wechatPay->verifySignature($xml)) {
    // 解析 XML 数据
    $data = $wechatPay->parseXml($xml);

    // 处理支付成功逻辑
    if ($data['result_code'] === 'SUCCESS' && $data['return_code'] === 'SUCCESS') {
        $order_id = $data['out_trade_no'];
        // 更新订单状态为已支付
        updateOrderStatus($order_id, 'paid');

        // 进行分账操作
        if ($wechatPay->isSplitEnabled()) {
            $wechatPay->doSplit($order_id);
        }

        // 返回成功响应
        $response = '<xml><return_code><![CDATA[SUCCESS]]></return_code><return_msg><![CDATA[OK]]></return_msg></xml>';
        echo $response;
    }
} else {
    // 签名验证失败，返回错误响应
    $response = '<xml><return_code><![CDATA[FAIL]]></return_code><return_msg><![CDATA[Signature verification failed]]></return_msg></xml>';
    echo $response;
}

function updateOrderStatus($order_id, $status) {
    global $wpdb;
    $table_name = $wpdb->prefix . 'woocommerce_orders';
    $wpdb->update(
        $table_name,
        array('status' => $status),
        array('id' => $order_id)
    );
}
?>
```

#### 支付宝支付回调处理
```php
<?php
// 引入必要的文件
require_once('path/to/alipay_sdk.php');

// 获取支付宝回调数据
$data = $_POST;

// 验证签名
$alipay = new Alipay();
if ($alipay->verifySignature($data)) {
    // 处理支付成功逻辑
    if ($data['trade_status'] === 'TRADE_SUCCESS') {
        $order_id = $data['out_trade_no'];
        // 更新订单状态为已支付
        updateOrderStatus($order_id, 'paid');

        // 进行分账操作
        if ($alipay->isSplitEnabled()) {
            $alipay->doSplit($order_id);
        }

        // 返回成功响应
        echo 'success';
    }
} else {
    // 签名验证失败，返回错误响应
    echo 'fail';
}

function updateOrderStatus($order_id, $status) {
    global $wpdb;
    $table_name = $wpdb->prefix . 'woocommerce_orders';
    $wpdb->update(
        $table_name,
        array('status' => $status),
        array('id' => $order_id)
    );
}
?>
```

### 分账操作示例代码（PHP）
```php
<?php
// 引入必要的文件
require_once('path/to/payment_plugin.php');

// 获取订单 ID
$order_id = $_GET['order_id'];

// 创建支付插件实例
$paymentPlugin = new PaymentPlugin();

// 执行分账操作
if ($paymentPlugin->doSplit($order_id)) {
    echo 'Split operation succeeded.';
} else {
    echo 'Split operation failed.';
}
?>
```

### 退款操作示例代码（PHP）
```php
<?php
// 引入必要的文件
require_once('path/to/payment_plugin.php');

// 获取订单 ID 和退款金额
$order_id = $_GET['order_id'];
$refund_amount = $_GET['refund_amount'];

// 创建支付插件实例
$paymentPlugin = new PaymentPlugin();

// 执行退款操作
if ($paymentPlugin->doRefund($order_id, $refund_amount)) {
    echo 'Refund operation succeeded.';
} else {
    echo 'Refund operation failed.';
}
?>
```

### 争议处理示例代码（PHP）
```php
<?php
// 引入必要的文件
require_once('path/to/payment_plugin.php');

// 获取订单 ID 和争议原因
$order_id = $_GET['order_id'];
$dispute_reason = $_GET['dispute_reason'];

// 创建支付插件实例
$paymentPlugin = new PaymentPlugin();

// 提交争议
if ($paymentPlugin->submitDispute($order_id, $dispute_reason)) {
    echo 'Dispute submitted successfully.';
} else {
    echo 'Failed to submit dispute.';
}
?>
```

通过以上文档，你可以以 `tao.ooo` 域名为例，在 WordPress Multisite 环境中完成支付插件的部署、配置、测试和维护工作。同时，附录中的内容能帮助你进一步理解数据库表结构、配置文件以及相关功能的代码实现，可根据实际情况进行调整和扩展。 

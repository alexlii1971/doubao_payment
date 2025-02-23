=== WordPress Multisite Payment Plugin ===
Contributors: Your Name
Tags: payment, multisite, woocommerce
Requires at least: 5.0
Tested up to: 6.0
Stable tag: 1.0
License: GPLv2 or later
License URI: http://www.gnu.org/licenses/gpl-2.0.html

A payment plugin for WordPress Multisite with WooCommerce integration, supporting multiple payment modes and advanced features.

== Description ==
This plugin enables seamless payment processing on WordPress Multisite environments integrated with WooCommerce. It supports multiple payment gateways such as WeChat Pay and Alipay, and includes features like split payments, refunds, dispute handling, and audit logging.

== Installation ==
1. Upload the `wp-multisite-payment-plugin` folder to the `/wp-content/plugins/` directory.
2. Activate the plugin through the 'Plugins' menu in WordPress.
3. Navigate to the 'Payment Settings' page in the admin menu to configure the payment gateways.

== Usage ==
- **Payment Processing**: Customers can choose between WeChat Pay and Alipay during the checkout process.
- **Split Payments**: Automatically split payments between the platform and merchants based on predefined rules.
- **Refunds**: Admins can initiate refunds for orders, with support for splitting refunds and rollback of split payments.
- **Dispute Handling**: Customers can submit disputes for orders, and admins can resolve or reject them.
- **Audit Logging**: All payment-related actions are logged for auditing purposes.

== Frequently Asked Questions ==
Q: Does this plugin support other payment gateways?
A: Currently, the plugin only supports WeChat Pay and Alipay. However, additional payment gateways can be added in future updates.

Q: Can I use this plugin without WooCommerce?
A: No, this plugin is designed to work with WooCommerce on WordPress Multisite.

== Changelog ==
= 1.0 =
- Initial release.

== Upgrade Notice ==
- Always backup your database before upgrading the plugin.

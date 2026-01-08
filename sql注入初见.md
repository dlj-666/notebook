---
[[sql]] [[sql injection]]
---
在dvwa的sql注入这里输入1'后报错
Fatal error: Uncaught mysqli_sql_exception: You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''1''' at line 1 in /var/www/html/DVWA/vulnerabilities/sqli/source/low.php:11 Stack trace: #0 /var/www/html/DVWA/vulnerabilities/sqli/source/low.php(11): mysqli_query() #1 /var/www/html/DVWA/vulnerabilities/sqli/index.php(34): require_once('...') #2 {main} thrown in /var/www/html/DVWA/vulnerabilities/sqli/source/low.php on line 11
# 前置知识
- SELECT * FROM [TABLE_NAME] WHERE [条件] ORDER BY []



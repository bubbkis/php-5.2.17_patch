# php-5.2.17_patch
## CentOS7 & php5.2.17 & apache2.4 でのbuild用Patch

```
sudo yum -y httpd httpd-devel
cd /usr/local/src
wget http://museum.php.net/php5/php-5.2.17.tar.gz
tar zxvf php-5.2.17.tar.gz
```

### buildするとext/dom/node.c 以下のエラーが発生する。

```zsh
/usr/local/src/php-5.2.17/ext/dom/node.c: In function 'dom_canonicalization':
/usr/local/src/php-5.2.17/ext/dom/node.c:1953:21: error: dereferencing pointer to incomplete type
    ret = buf->buffer->use;
                     ^
In file included from /usr/local/src/php-5.2.17/main/php.h:38:0,
                 from /usr/local/src/php-5.2.17/ext/dom/node.c:26:
/usr/local/src/php-5.2.17/ext/dom/node.c:1955:40: error: dereferencing pointer to incomplete type
     RETVAL_STRINGL((char *) buf->buffer->content, ret, 1);
                                        ^
/usr/local/src/php-5.2.17/Zend/zend_API.h:472:14: note: in definition of macro 'ZVAL_STRINGL'
   char *__s=(s); int __l=l;  \
              ^
/usr/local/src/php-5.2.17/ext/dom/node.c:1955:5: note: in expansion of macro 'RETVAL_STRINGL'
     RETVAL_STRINGL((char *) buf->buffer->content, ret, 1);
     ^
make: *** [ext/dom/node.lo] Error 1
```

## php-5.2.17_patch をクローンして持ってくる
```zsh
git clone https://github.com/bubbkis/php-5.2.17_patch.git ./
```

## php-5.2.17_patch を当てる
```zsh
cd php-5.2.17
sudo patch -p0 < ../php-5.2.17_patch/php-5.2.17.patch 
```
```
patching file ext/dom/node.c
Hunk #1 succeeded at 1950 (offset 55 lines).
patching file ext/dom/documenttype.c
Hunk #1 succeeded at 215 (offset 10 lines).
patching file ext/simplexml/simplexml.c
Hunk #1 succeeded at 1343 (offset -74 lines).
```

### php5.2.17とapache2.4.*でコンパイルリンクしてhttpdを動作させるとエラーになる
```zsh
httpd: Syntax error on line 55 of /etc/httpd/conf/httpd.conf: Cannot load /usr/lib64/httpd/modules/libphp5.so into server: /usr/lib64/httpd/modules/libphp5.so: undefined symbol: unixd_config
```

## php_functions.c.patch を当てる
sourceファイルの中の
./sapi/apache2hander/php_functions.c
にpatchを当てる。
```zsh
sudo patch -u ./sapi/apache2handler/php_functions.c < ../php-5.2.17_patch/php_functions.c.patch
```

#### configure時に、"/usr/bin/ld: cannot find -lltdl" というエラーが発生した場合、libtool-ltdl-develをインストールする
```zsh
sudo yum install libtool-ltdl-devel
```

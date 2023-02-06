## 打包与解压
## 1）.zip
zip <xxx.zip> <file>											## 压缩⾄zip包
zip -r <xxx.zip> <file1> <file2> <dir1>		## 将多个⽂件+⽬录压成zip包
unzip <xxx.zip>														## 解压zip包

## 2）.tar
tar -cvf <xxx.tar> <file>									## 创建⾮压缩tar包
tar -cvf <xxx.tar> <file1> <file2> <dir1>	## 将多个⽂件+⽬录打tar包
tar -tf <xxx.tar>													## 查看tar包的内容
tar -xvf <xxx.tar>												## 解压tar包
tar -xvf <xxx.tar> -C /dir								## 将tar包解压⾄指定⽬录

## 3）.bz2
tar -cvfj <xxx.tar.bz2> <dir>	## 创建bz2压缩包
tar -jxvf <xxx.tar.bz2>				## 解压bz2压缩包
bunzip2 <xxx.bz2>							## 解压bz2压缩包

## 4）.gz
tar -cvfz <xxx.tar.gz> <dir>	## 创建gzip压缩包
tar -zxvf <xxx.tar.gz>				## 解压gzip压缩包
gunzip <xxx.gz>								## 解压gzip压缩包
gzip -d <xxx.gz> 							## 解压gzip压缩包

## 压缩文件
bzip2 <file_name>			## 压缩⽂件
gzip <file_name>			## 基础压缩（不保留源文件）压缩文件为 源文件.gz
gzip -9 <file_name>		## 最⼤程度压缩
## e.g. 保留源文件压缩
gzip -c file_name.txt > file_name.txt.gz ## -c压缩重定向（保留源文件）压缩文件为 源文件.gz

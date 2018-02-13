## 将文本文件提取后写入至csv文本中

### 文本文件格式如下
```
99900036    中行悉尼                       16 621394       1 1                        
99900124    Bank of China,Canada           16 623423       1 1                        
99900250    Bank of China Paris Branch     16 629153       1 1                         
99900348    Bank of China (Hungary) Close  16 623485       1 1                      
99900360    中行印尼分行                   16 621248       1 1                        
99900392    中行日本分行                   16 621215       1 1                       
99900410    中国银行首尔分行               16 621249       1 1                     
99900458    Bank of China(Malaysia)        16 625829       1 2                       
99900458    Bank of China(Malaysia)        16 625943       1 2                        
99900608    中行菲律宾分行                 16 621231       1 1                        
99900702    中行新加坡分行                 16 6227890      1 2                           
99900702    中行新加坡分行                 16 6227891      1 2
```

```
#!/usr/bin/env python
 
import re
import csv
 
def csv_writer(data, filename):
    # csv是文本格式的文件不支持二进制的写入，所以不要用'b'模式打开文件数据也不必转成bytes
    # newline=''可以防止写入csv每行后面多一个空白行
    with open(filename, "w", newline='') as csv_file:
        writer = csv.writer(csv_file)
        for line in data:
            writer.writerow(line)
 
if __name__ == "__main__":
    data = []
    with open('d:/CARDBIN', mode='r', encoding='GB2312') as f:
        for line in f:
            # data.append(re.split(r''[;,\s]\s*', line)  # 不确定空格的数量
            # data.append(re.split(r'\s{2,}', line))     # 两个及以上空格
            data.append(re.split(r'\s*', line))          # 1个及以上空格
            
    print(data)
    filename = "output.csv"
    csv_writer(data, filename)
```
输出的文件其中有个小问题英文字符串的空格也会被分割

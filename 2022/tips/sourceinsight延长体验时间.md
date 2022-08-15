## 修改sourceinsight4.exe

用16进制编辑器（sublime text）打开sourceinsight4.exe文件，找到**c800 0000 742a 83bc 2408** 这一段，修改74 为 eb。

## 修改license文件

打开 C:\ProgramData\Source Insight\4.0\si4.lic
将Expiration=”2022-XX-XX”中的2022修改为2050。

注意：过一段时间提示过期后，把Date="2024-01-07 00:00:00"，改成前一天的，又能继续使用。




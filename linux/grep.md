# grep 사용법

filter 를 하는 사용법이다.

```text
cat aaa.txt | grep bbb
cat aaa.txt | grep -i -B 4 -A 2 bbb
# -i 대소문자 구분안함. 
# -B bbb를 포함한 줄의 before 4줄 출력
# -A bbb를 포함한 줄의 after 2줄 출력
# -v <keywords to be excluded>
```

grep  옵션 

```text
grep "([A-Za-z ]*)" - (blah Blah Blah) 
```






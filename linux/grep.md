# grep 사용법

## 기본 사용법

```bash
cat aaa.txt | grep bbb
```

## 옵션

```bash
cat aaa.txt | grep -i -B 4 -A 2 -e "([A-Za-z]*)" bbb
```

- bbb 검색할 문자.
- -i : 대소문자 구분안함.
- -B : bbb를 포함한 줄의 before 4줄 출력
- -A : bbb를 포함한 줄의 after 2줄 출력
- -v : keywords to be excluded
- -e --regexp: regexp 사용

여러개의 단어를 동시에 필터하고 싶은경우

`grep -e gtid -e delete`

gtid나 delete가 있는줄을 보여준다.

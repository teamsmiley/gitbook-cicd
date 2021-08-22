# grep 사용법

## 기본 사용법

```sh
cat aaa.txt | grep bbb
```

## 옵션

```sh
cat aaa.txt | grep -i -B 4 -A 2 -e "([A-Za-z]*)"  bbb
```

- -i : 대소문자 구분안함.
- -B : bbb를 포함한 줄의 before 4줄 출력
- -A : bbb를 포함한 줄의 after 2줄 출력
- -v : keywords to be excluded
- -e --regexp: regexp 사용

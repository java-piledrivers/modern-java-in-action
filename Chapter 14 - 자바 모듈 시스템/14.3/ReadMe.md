## 14.3 자바 모듈 : 큰 그림

자바 8은 **모듈** 이라는 새로운 자바 프로그램 구조 단위를 제공한다.

`module` 이라는 키워드에 이름과 바디를 추가해서 정의한다.

모듈 디스크립터는 module-info.java라는 파일에 저장되고, 보통 패키지와 같은 폴더에 위치한다.

```
module 모듈명
exports 패키지명
requires 모듈명
```

메이븐 같은 도구에서 모듈의 많은 세부사항을 IDE가 처리한다.

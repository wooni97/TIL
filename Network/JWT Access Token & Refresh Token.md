<aside>
💡 Refresh Token, Access Token 둘다 JWT. 단, 역할이 다르다.

</aside>

- Access Token : 접근에 관여하는 토큰
- Refresh Token : 재발급에 관여하는 토큰

### Refresh Token이 필요한 이유

Access Token만을 통한 인증 방식은 제 3자에게 탈취당할 경우 보안이 취약하다는 문제가 있다. 

Access Token이 발급된 이후, 서버에 저장되지 않고 토큰 자체만으로 검증하기 때문에 토큰을 획득한 사람은 누구나 접근이 가능하게 된다. 

JWT는 발급 이후 삭제가 불가능 하기 때문에 유효기간을 부여하는 식으로 탈취 문제에 대응할 수 있다. 

그러나 유효기간이 짧은 Token의 경우 그만큼 사용자는 로그인을 자주 해야 하기에 불편하다는 단점이 있고, 유효기간을 너무 늘리면 보안에 취약해지게 된다.

이에 대한 해결책이 **Refresh Token**이다.

Access Token이 접근에 관여하는 토큰이라면 Refresh Token은 재발급에 관여하는 토큰이다. 

처음 로그인을 했을 때, 서버는 로그인에 성공하면 Access Token, Refresh Token을 동시에 발급한다. 서버는 데이터베이스에 Refresh Token을 저장한다. 

이 Refresh Token은 Access Token보다 상대적으로 긴 유효시간을 가지면서, Access Token이 만료됐을 때 새로 발급해주는 열쇠가 된다. 

### 동작 방식

1. 기본적으로 로그인 성공 시 Access Token, Refresh Token을 모두 발급한다.
2. 사용자가 인증이 필요한 AP에 접근하고자 하면 토큰을 검사한다.
    - case 1 : access token과 refresh token 모두 만료된 경우 → 에러 발생 (재로그인하여 둘다 발급)
    - case 2 : access token은 만료됐지만, refresh token은 유효한 경우 → refresh token을 검증하여 access token 재발급
    - case 3 : access token은 유효하지만, refresh token 만료된 경우 → access token을 검증하여 refresh token 재발급
    - case 4 : 모두 유효한 경우 → 정상 처리
3. 로그아웃을 하면 Access token, Refresh token 모두 만료

[🌐 Access Token & Refresh Token 원리](https://inpa.tistory.com/entry/WEB-📚-Access-Token-Refresh-Token-원리-feat-JWT)

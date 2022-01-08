# Jwt

[공식문서](https://github.com/jwtk/jjwt#quickstart)

**build.gradle**

```java
dependencies {
    implementation 'io.jsonwebtoken:jjwt-api:0.11.2'
    runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.2',
            // Uncomment the next line if you want to use RSASSA-PSS (PS256, PS384, PS512) algorithms:
            //'org.bouncycastle:bcprov-jdk15on:1.60',
            'io.jsonwebtoken:jjwt-jackson:0.11.2' // or 'io.jsonwebtoken:jjwt-gson:0.11.2' for gson
}
```

### Creating a JWS

- JWT는 json header와 body 그리고 signature를 갖는다.

```java
@PostConstruct
protected void init() {
    secretKey = Base64.getEncoder().encodeToString(secretKey.getBytes());
}

public String createToken(String userPk, List<String> roles) {
        Claims claims = Jwts.claims().setSubject(userPk);
        claims.put("roles", roles);
        Date date = new Date();
        return Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(date)
                .setExpiration(new Date(date.getTime() + tokenValidMilisecond))
                .signWith(SignatureAlgorithm.HS256, secretKey)
                .compact();
    }
						.
```

- Claims은 JWT의 body를 가리키며 생성자가 원하고자 하는 정보를 포함한다.
- `setIssuer`: sets the `[iss` (Issuer) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.1)
- `setSubject`: sets the `[sub` (Subject) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.2)
- `setAudience`: sets the `[aud` (Audience) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.3)
- `setExpiration`: sets the `[exp` (Expiration Time) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.4)
- `setNotBefore`: sets the `[nbf` (Not Before) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.5)
- `setIssuedAt`: sets the `[iat` (Issued At) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.6)
- `setId`: sets the `[jti` (JWT ID) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.7)

### Reading JWs

```java
Jws<Claims> claims = Jwts.parser()
.setSigningKey(secretKey)
.parseClaimsJws(jwtToken);
```

**검증**

```java
public String resolveToken(HttpServletRequest request) {
        return request.getHeader("X-AUTH-TOKEN");
    }

public boolean validateToken(String jwtToken) {
        try {
            Jws<Claims> claims = Jwts.parser().setSigningKey(secretKey).parseClaimsJws(jwtToken);
            return !claims.getBody().getExpiration().before(new Date());
        } catch (Exception e) {
            return false;
        }
    }
```
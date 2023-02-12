---
title: 基于jwt实现token登录验证
date: 2020-07-20 09:27:09
tags:
- token
categories:
- 框架
---

**引入jwt依赖**

```java
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.2.0</version>
</dependency>
```

**jwt工具类，生成和解析token**

```java
public class JwtUtil {
    //过期时间
    private static final long EXPIRE_TIME = 15 * 60 * 1000;
    //私钥
    private static final String TOKEN_SECRET = "privateKey";

    /**
     * 生成签名，15分钟过期
     */
    public static String sign(Long userId) {
        try {
            // 设置过期时间
            Date date = new Date(System.currentTimeMillis() + EXPIRE_TIME);
            // 私钥和加密算法
            Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);
            // 设置头部信息
            Map<String, Object> header = new HashMap<>(2);
            header.put("Type", "Jwt");
            header.put("alg", "HS256");
            // 返回token字符串
            return JWT.create()
                    .withHeader(header)
                    .withClaim("userId", userId)
                    .withExpiresAt(date)
                    .sign(algorithm);
        } catch (Exception e) {
            // e.printStackTrace();
            return null;
        }
    }

    /**
     * 检验token是否正确
     */
    public static Long verify(String token){
        try {
            Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);
            JWTVerifier verifier = JWT.require(algorithm).build();
            DecodedJWT jwt = verifier.verify(token);
            Long userId = jwt.getClaim("userId").asLong();
            return userId;
        } catch (Exception e){
            // e.printStackTrace();
            return 0L;
        }
    }
}
```

**返回结果实体类**

```java
public class Result implements Serializable {

    private boolean success;
    private String message;

    public Result(boolean success, String message) {
        this.success = success;
        this.message = message;
    }

    public boolean isSuccess() {
        return success;
    }

    public void setSuccess(boolean success) {
        this.success = success;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}

```

**控制器类**

```java
@RequestMapping("/user")
@RestController
@CrossOrigin
public class UserController {

    @RequestMapping(value = "/login", method = RequestMethod.POST)
    public Result login(String username, String password) {
        if (username.equals("admin") && password.equals("123456")) {
            Long userId = 1328L;
            return new Result(true, JwtUtil.sign(userId));
        } else {
            return new Result(false, "用户名密码错误");
        }
    }

    @RequestMapping(value = "/find", method = RequestMethod.GET)
    public Result find(HttpServletRequest request) {
        String token = request.getHeader("token");
        if (token == null) {
            return new Result(false, "用户未登录");
        }
        Long userId = JwtUtil.verify(token);
        if (userId != 0L) {
            return new Result(true, String.valueOf(userId));
        } else {
            return new Result(false, "登录失效，请重新登录");
        }
    }
}
```

**主页**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Index</title>
    <script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.js"></script>
    <script src="https://cdn.bootcss.com/jquery-cookie/1.4.1/jquery.cookie.min.js"></script>
</head>
<body>
<h3>首页</h3>
<a href="login.html">去登录</a><br><br>
<button>个人中心</button>
<script>
    $(function () {
        let btn = $("button");
        btn.click(function () {
            let token = $.cookie("token");
            $.ajax({
                url: "http://localhost:8080/user/find",
                type: "get",
                headers: {
                    'token': token
                },
                success: function (res) {
                    if (res.success) {
                        alert("用户Id为：" + res.message)
                    } else {
                        alert(res.message);
                    }
                }
            })
        })
    })
</script>
</body>
</html>
```

**登录页面**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
    <script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.js"></script>
    <script src="https://cdn.bootcss.com/jquery-cookie/1.4.1/jquery.cookie.min.js"></script>
</head>
<body>
<h3>请登录</h3>
用户名：<input type="text"><br>
密码：<input type="password"><br>
<input type="submit" value="登录">
<script>
    $(function () {
        let btn = $("input[type=submit]");
        btn.click(function () {
            let username = $("input[type=text]").val();
            let password = $("input[type=password]").val();
            $.ajax({
                url: "http://localhost:8080/user/login",
                type: "post",
                data: {
                    "username": username,
                    "password": password
                },
                success: function (res) {
                    if (res.success) {
                        alert("登录成功");
                        $.cookie("token", res.message);
                        window.location.href = "index.html";
                    } else {
                        alert("用户名密码错误");
                    }
                }
            })
        })
    })
</script>
</body>
</html>
```

**基本流程**

1. 登录页面输入用户名和密码，进行登录验证。
2. 验证失败，弹出用户名密码错误提示。
3. 验证成功，弹出登录成功提示，将服务器端返回的token保存在cookie中，跳转到首页。
4. 每次请求在headers中携带token，服务端验证成功处理请求，验证失败返回错误信息。
5. token设置有过期时间，过期后需要重新登录。

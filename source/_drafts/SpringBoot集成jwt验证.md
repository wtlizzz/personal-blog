title: SpringBoot集成jwt验证
author: Wtli
tags:
  - SpringBoot
  - Jwt
  - SpringSecurity
categories:
  - 后端
date: 2020-09-04 13:45:00
---
本文介绍
- SpringBoot集成Jwt的过程
- Token生成与有效时间的刷新与使用
- 随项目启动加载用户权限数据
- 数据库建立相应数据表
<!-- more -->

### Token使用
#### Token生成
```
public String generateToken(AircasBaseUser userDetails) {
    Map<String, Object> claims = new HashMap<>();
    return doGenerateToken(claims, userDetails.getUsername());
}

private String doGenerateToken(Map<String, Object> claims, String subject) {
    final Date createdDate = clock.now();
    final Date expirationDate = calculateExpirationDate(createdDate);
    return Jwts.builder()
            .setHeaderParam("type", "JWT")
            .setClaims(claims)//声明，作为荷载
            .setSubject(subject) //代表这个JWT的主体，即它的所有人，这个是一个json格式的字符串，可以存放什么userid，roldid之类的，作为什么用户的唯一标志
            .setIssuedAt(createdDate)
            .setExpiration(expirationDate)
            .signWith(SignatureAlgorithm.HS512, secret)//signWith() 签名方法。两个参数分别是签名算法和自定义的签名Key（盐）。签名key可以byte[] 、String及Key的形式传入。前两种形式均存入builder的keyBytes属性，后一种形式存入builder的key属性。如果是第二种（及String类型）的key，则将其进行base64解码获得byte[] 。
            .compact();
    }
```


### SpringBoot集成jwt

pom引入
```
<!--jwt-->
<dependency>
   <groupId>io.jsonwebtoken</groupId>
   <artifactId>jjwt</artifactId>
   <version>0.9.1</version>
</dependency>
```

WebSecurityConfigurerAdapter添加配置，添加过滤器
```
private void configureForDefault(HttpSecurity httpSecurity) throws Exception {
    httpSecurity
           // 禁用 CSRF
           .csrf().disable()
           // 授权异常
           .exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
           // 不创建会话
           .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
           // 过滤请求
           .authorizeRequests().antMatchers(HttpMethod.GET, "/*.html", "/**/*.html", "/**/*.css", "/**/*.js", "/*.gif", "/*.gif*", "/*.ico").anonymous()
           .antMatchers(HttpMethod.POST, "/auth/login").anonymous()
           .antMatchers(HttpMethod.POST, "/auth/userRegister").anonymous()
           .antMatchers("/auth/getVCode").anonymous()
           // swagger start
           .antMatchers("/swagger-ui.html").anonymous().antMatchers("/swagger-resources/**").anonymous()
           .antMatchers("/webjars/**").anonymous().antMatchers("/*/api-docs").anonymous()
           // 接口限流测试
           .antMatchers("/test/**").anonymous().antMatchers(HttpMethod.OPTIONS, "/**").anonymous()
           .antMatchers("/static/**").permitAll()
           .antMatchers("/favicon.ico").permitAll()
           // 所有请求都需要认证
           .anyRequest().authenticated()
           // 防止iframe 造成跨域
           .and().headers().frameOptions().disable();
        httpSecurity.addFilterBefore(jwtAuthorizationTokenFilter, UsernamePasswordAuthenticationFilter.class);
    }
```
请求拦截器处理。
- 所有请求都会到这个拦截器中，验证头部数据是否有token。
- 如果没有token则会没有权限访问其他请求接口。
- 如果有token数据，验证Token的有效性，如果有效就在SecurityContextHolder添加权限，就能够访问其他请求。
- 在拦截器中需要获取用户权限信息，所以在项目中添加用户信息的缓存，伴随服务的启动而加载。

```
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
    String requestHeader = request.getHeader(this.tokenHeader);
    if (requestHeader != null && requestHeader.startsWith("Bearer ")) {
        String authToken = requestHeader.substring(7);
        if (StringUtils.isNotBlank(authToken) && jwtTokenUtil.validateToken(authToken)) {
            //如果token有效则会添加请求的权限
            String userName = jwtTokenUtil.getUsernameFromToken(authToken);
            logger.info("checking authentication {}" + userName);
            if (userName != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(userName, null, jwtUserDetailsService.getAuthorities(userName));
                authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                logger.info("authenticated user {}, setting security context" + userName);
                SecurityContextHolder.getContext().setAuthentication(authenticationToken);
            }
        }
    }
    chain.doFilter(request, response);
}
```
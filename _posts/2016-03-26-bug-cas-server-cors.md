---
layout: post
title: 修复bug CAS 整合 CORS
tags: [其他]
---

在进行CAS整合项目过程中，碰到了一个bug，修复了很长时间。
CAS整合项目，是类似SSO的单一登录系统，整合 邮件客户端 Rainloop 和 odoo ERP系统统一登录的问题。


###  bug 描述

bug具体表现为，邮件客户端登出时，根据cas协议返回CAS系统登出时失败。
Chrome console 调试后，发现具体原因是
    > XMLHttpRequest cannot load https://192.168.31.173:8443/logout. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://rainloop.io' is therefore not allowed access.

典型的JS异步调用跨域 CORS问题 - 跨域XMLHttpRequest。


### 解决办法。
一 js 使用 json 调用异步
类似 $.ajax({
  type: "POST",
  dataType: 'jsonp',

由于 rainloop JS 具体实现，为尝试。

PS. 另外, 尝试，在rainloop php段，增加<?php header('Access-Control-Allow-Origin: *'); ?>, 失败

二 CAS 服务段 允许CORS

CAS 服务器为 tomcat 服务器。 具体操作是,
1. 服务器安装依赖jar
在 web application (webapps/<your-web-app>/WEB-INF/lib/) 路径下

cors-filter-2.4.jar , http://search.maven.org/remotecontent?filepath=com/thetransactioncompany/cors-filter/2.4/cors-filter-2.4.jar

java-property-utils-1.9.1.jar, http://search.maven.org/remotecontent?filepath=com/thetransactioncompany/java-property-utils/1.9.1/java-property-utils-1.9.1.jar

2. 配置web.xml
在 web application 路径(webapps/<your-web-app>/WEB-INF/) 下 增加以下代码

      <filter>
        <filter-name>CORS</filter-name>
        <filter-class>com.thetransactioncompany.cors.CORSFilter</filter-class>

        <init-param>
            <param-name>cors.allowOrigin</param-name>
            <param-value>*</param-value>
        </init-param>
        <init-param>
            <param-name>cors.supportsCredentials</param-name>
            <param-value>false</param-value>
        </init-param>
        <init-param>
            <param-name>cors.supportedHeaders</param-name>
            <param-value>accept, authorization, origin</param-value>
        </init-param>
        <init-param>
            <param-name>cors.supportedMethods</param-name>
            <param-value>GET, POST, HEAD, OPTIONS</param-value>
        </init-param>
      </filter>

  <filter-mapping>
    <filter-name>CORS</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

3. 重启tomcat


### 参考文章
1. CORS 问题: http://stackoverflow.com/questions/20035101/no-access-control-allow-origin-header-is-present-on-the-requested-resource?rq=1
2. CAS CORS 配置:  http://software.dzhuvinov.com/cors-filter-installation.html

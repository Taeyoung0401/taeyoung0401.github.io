---
layout: post
title: Spring Boot Gradle Troubleshootings
subtitle: 불편한 환경에서 Gradle..
gh-repo: daattali/beautiful-jekyll
gh-badge: [star, fork, follow]
tags: [gradle, spring-boot]
comments: true
---

# Gradle 초기화 시 발생하는 여러 문제에 대한 

## No candidates found for method call plugins

`Reload All Gradle Projects` 후 재시도 하라는데 내 경우에는 별 개선이 없다.

<br>

## Plugin [id: 'org.springframework.boot', version: '2.2.5.RELEASE'] was not found in any of the following sources:

### 첫번째 솔루션

- `JAVA_HOME`, `Path` 등에 시스템변수가 정상적으로 설정되어 있는지 확인
- JVM이 정상적으로 설정되어 있는지 확인
- `build.gradle` 수정  
방화벽 때문에 `gradle-wrapper.properties`의 `distributionUrl`을 정상적으로 가져오지 못하는 문제가 있을 수 있다.  
[여기](https://da2uns2.tistory.com/entry/Spring-Boot-Plugin-id-orgspringframeworkboot-version-256-was-not-found-%EC%98%A4%EB%A5%98-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0)를 참고해서 수정하자.

<br>

### 두번째 솔루션

`settings.gradle`에 `pluginManagement`를 정의해 준다.

```gradle
// settings.gradle
pluginManagement {
    repositories {
        maven { url 'https://repo.spring.io/release' }
        mavenCentral()
        gradlePluginPortal()
    }
}

rootProject.name = ...
```

> https://stackoverflow.com/questions/70302863/plugin-id-io-spring-dependency-management-version-1-0-11-release-was-no

<br>    

## Cause: unable to find valid certification path to requested target

인증서 문제. 아래 커맨드 참고해서 사용하는 JDK에 인증서 추가

```
keytool -import -trustcacerts -keystore /opt/java/jre/lib/security/cacerts \
   -storepass changeit -noprompt -alias mycert -file /tmp/examplecert.crt
```

<br>

## Could not initialize class org.codehaus.groovy.runtime.InvokerHelper

이건 Gradle과 JDK의 버전이 맞지 않아서 나오는 문제

<br>

# 결론은...

네트워크에 제약이 너무 많은 관계로 문제가 끝나지 않는다.  
그냥 의존성을 로컬에서 관리하기로 하자. 추가할 때 좀 불편하겠지만, 이 상황보다 불편해질 수는 없다.

```gradle
// jar 파일을 `libs`에 저장해 놓는다고 하면..

repositories {
    flatDir {
        dirs 'libs'
    }
    mavenCentral()
}
```

jar파일을 하나하나 받는게 너무 힘들다고 생각이 되면, 네트워크에 제약이 없는 곳에서 아래 task를 통해 한번에 수집하는 것도 가능하다.

```gradle
task getDeps(type:Copy) {
    from sourceSets.main.runtimeClasspath
    into 'runtime/'

    println 'get-dependencies'
}
```

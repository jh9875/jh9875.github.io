---
layout: post
title:  "[Spring] Component scan"
date:   2020-08-10
categories: SPRING
permalink: /archivers/Spring_ComponentScan
tags: [spring, java]
---

## 머릿말

이번 포스팅에서는 컴포넌트 스캔에 대해 알아보겠습니다.   

(이번 포스팅에서 사용하는 코드는 최범균님의 "초보 웹 개발자를 위한 스프링5 프로그래밍 입문" 코드를 활용했습니다.)   
<br/>
<br/>

## 컴포넌트 스캔

지난 포스팅에선 스프링이 제공하는 자동 의존 주입에 대해 알아보았습니다.

자동 의존 주입은 스프링에서 객체를 자동으로 의존 주입시켜주는 기능이었습니다.

이번에 알아볼 컴포넌트 스캔은 지정한 클래스를 자동으로 빈으로 등록해 주는 기능입니다.

사용 방법은 빈으로 지정하려는 클래스에 **@Component** 어노테이션을 사용해야 합니다.

다음 코드를 보겠습니다.

~~~java
...

@Component
public class MemberDao {
    ...
}
~~~
<br/>

위 코드처럼 빈으로 사용할 클래스 위에 @Component 어노테이션을 붙여 주시면 됩니다.

@Component 어노테이션을 붙임으로써 스프링 설정 클래스에서 Bean으로 등록을 하지 않고 스프링이 알아서 Bean으로 찾을 수 있습니다.

그러면 다음과 같이 설정 코드가 줄어 듭니다.

~~~java
...

@Configuration
@ComponentScan(basePackages ={"spring"})
public class AppCtx {

    /*
    @Bean
	public MemberDao memberDao() {
		return new MemberDao();
	}
    */

    @Bean
	public MemberRegisterService memberRegSvc() {
		return new MemberRegisterService(memberDao());
	}

    ...

}
~~~

<br/>
<br/>

## 스캔 대상 설정

위 예제 코드를 보시면 이전 AppCtx와 다른점이 있습니다.

@Configuration 어노테이션 밑에 다른 어노테이션이 있습니다.

@ComponentScan 어노테이션입니다.

**@ComponentScan** 어노테이션은 @Component 어노테이션으로 스캔 대상으로 지정한 클래스들을 빈 객체로 등록합니다.

그리고 속성값을 지정해 줄 수 있는데, 위 코드에서 basePackages를 spring으로 설정해 주었습니다.

그 의미는 spring패키지와 그 하위 패키지에 속한 클래스를 대상으로 스캔하라는 의미입니다.

<br/>

@ComponentScan 어노테이션은 스캔 대상을 제외하는 기능도 제공합니다.

*excludeFilters* 속성을 이용하면 됩니다.

~~~java
...

@Configuration
@ComponentScan(basePackages ={"spring"},
    excludeFilters =@Filter(type =FilterType.REGEX, pattern ="spring\\..*Dao"))
public class AppCtx {
    ...
}
~~~
<br/>

위 코드는 excludeFilters 속성을 지정해준 코드입니다.

위 코드에선 @Filter 어노테이션의 type 속성값으로 FilterType.REGEX를 사용 함으로써 정규표현식을 사용해서 스캔 대상을 제외시켰습니다.

위 정규표현식 의미는 "spring." 으로 시작하고 Dao로 끝나는 것을 의미하므로, spring.~Dao 클래스를 제외한다는 의미입니다.(spring은 패키지 이름, ~Dao는 클래스 이름.)

<br/>


컴포넌트 스캔 대상에서 제외하는 방법은 몇가지 더 있습니다.

1. 정규표현식 사용(위 예제)
   
   위 예제에서 사용한 것처럼 FilterType.REGEX를 이용한 방법입니다.
2. AjpectJ 패턴 사용
   
   AspectJ를 이용한 코드를 보겠습니다.

   ~~~java
    @ComponentScan(basePackage ={"spring"},
        excludeFilters =@Filter(type = FilterType.ASPECTJ, patter ="spring.*Dao"))
    public class ... {
        ...
    }
   ~~~
    <br/>

   위 코드는 AspectJ를 이용한 코드인데 AspectJ에 대해서는 추후에 알아보도록 하겠습니다.

   의미는 spring패키지에서 이름이 Dao로 끝나는 타입을 제외한다는 의미입니다.

3. 특정 어노테이션 제외
   
   특정 어노테이션이 붙은 클래스를 제외하려면 FilterType.ANNOTATION을 사용하면 됩니다.

   > type =FilterType.ANNOTATION, classes = {NoProduct.class, ManualBean.class}

   위 코드를 @Filter에 넣으면 classes 안에 있는 어노테이션(여기선 NoProduct, ManualBean)을 적용한 클래스를 스캔 대상에서 제외합니다.


4. 특정 타입 제외
   
   특정 타입을 제외하려면 FilterType.ASSIGNABLE_TYPE을 설정해 주면 됩니다.

   > type =FilterType.ASSIGNABLE_TYPE, classes = {MemberDao.class}


<br/>
<br/>

## 기본 스캔 대상

@Component 어노테이션 외에도 다른 어노테이션을 사용 함으로써 스캔 대상에 포함될 수 있습니다.

- @Component
- @Controller
- @Service
- Repository

각각의 의미는 다음과 같습니다.

@Component : 해당 클래스를 빈으로 알려줌.
@Controller : spring의 MVC에서 Controller로 쓰일것을 알려줌.
@Service : 해당 클래스가 비지니스 로직을 수행할것을 알림.
@Repository : DAO 클래스로 쓰일것을 알림.

위 어노테이션을 사용하면 해당 타입 또한 스캔 대상으로 등록됩니다.

<br/>
<br/>

## 정리

이번 포스팅에서는 @Component 어노테이션에 대해서 알아봤습니다.

@Component 어노테이션을 사용 함으로써 설정 클래스에 빈으로 등록하지 않아도 사용할 수 있었습니다.

또한 @Component 어노테이션을 붙인 클래스를 스프링 빈으로 등록하기 위해  @ComponentScan 어노테이션을 사용했습니다.

여기서 basePackages 속성값과 excludeFilters 속성값으로 스캔 대상을 정하거나 제한할 수 있었습니다.

<br/>
<br/>

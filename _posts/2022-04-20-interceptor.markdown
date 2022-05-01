---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: id_Examples
title: "[스프링]interceptor 작성"

# post specific
# if not specified, .name will be used from _data/owner.yml
author: Mr. Green's Workshop
# multiple category is not supported
category: 스프링
# multiple tag entries are possible
tags: []
# thumbnail image for post
img: ":Spring_Framework_logo.png"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-04-20 11:30:06 +0900

# seo
# if not specified, date will be used.
#meta_modify_date: 2022-02-10 08:11:06 +0900
# check the meta_common_description in _data/lang/[language].yml
#meta_description: ""

# optional
# if you enabled image_viewer_posts you don't need to enable this. This is only if image_viewer_posts = false
#image_viewer_on: true
# if you enabled image_lazy_loader_posts you don't need to enable this. This is only if image_lazy_loader_posts = false
#image_lazy_loader_on: true
# exclude from on site search
#on_site_search_exclude: true
# exclude from search engines
#search_engine_exclude: true
# to disable this page, simply set published: false or delete this file
#published: false
---

<!-- outline-start -->

<!-- outline-end -->
# 인터셉터란?
컨트롤러(Controller)의 '핸들러(Handler)'를 호출하기 전과 후에 요청과 응답을   
참조하거나 가공할수 있는 일종의 필터   


# 사용하는 이유?
작성해주어야 할 핸들러수가 적다면 문제가 되지않는다   
하지만 적용해야할 핸들러가 수천개가 된다면 크게 두가지 문제가 생기는데   
1)메모리 낭비, 서버의 부하가 늘어난다   
2)코드 누락, 사람이 작성하는것이므로 실수가 발생할 가능성이 높다   
그러므로 특정 Controller의 핸들러가 실행되기 전이나 후에 추가적인 작업을 원할때   
Interceptor를 사용한다 (추가적인 작업으로는 로그인체크, 권한 체크 등이 있다.)   

# 인터셉터 클래스 추가

```
public class BaseHandlerInterceptor implements HandlerInterceptor {

    Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        logger.info("preHandle requestURI : {}", request.getRequestURI());
        if (handler instanceof HandlerMethod) {
            HandlerMethod handlerMethod = (HandlerMethod) handler;
            logger.info("handlerMethod : {}", handlerMethod);
            RequestConfig requestConfig = handlerMethod.getMethodAnnotation(RequestConfig.class);
            if (requestConfig != null) { // 로그인 체크가 필수인 경우
                if (requestConfig.loginCheck()) {
                    throw new BaseException(BaseResponseCode.LOGIN_REQUIRED, new String[]{request.getRequestURI()});
                }
            }
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        logger.info("postHandle requestURI : {}", request.getRequestURI());

    }
}
```
# 인터셉터를 스프링빈으로 등록
```
@Configuration
public class WebConfiguration implements WebMvcConfigurer {
    ...
    @Bean
    public BaseHandlerInterceptor baseHandlerInterceptor(){
        return new BaseHandlerInterceptor();
    }
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(baseHandlerInterceptor());
    }
}
```

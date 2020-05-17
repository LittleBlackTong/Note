# zuul 网关实现鉴权和限流

### 一、zuul 的核心功能
> zuul 的主要就是过滤器把不符合要求的请求过滤出去,常用的过滤器类型 前置过滤 Filterconstants.PRE_TYPE FilterConstants.POST_TYPE

### 二、通过 zuul 实现过滤器

1. post 请求过滤,并且在响应体中添加请求头

代码实现:
```
/**
 * @author littleblack TongTong
 * @create 2020-05-15 12:17 上午
 **/
@Component
public class ResponseHeaderFilter extends ZuulFilter { // zuul 的过滤器需要继承 zuulfilter 类
    @Override
    public String filterType() {  // 声明过滤器类型 POST 请求类型的过滤
        return FilterConstants.POST_TYPE;
    }

    @Override
    public int filterOrder() { // 声明过滤器的顺序,过滤器是顺序执行的,数值越小的过滤器越先执行 POST 类型的请求 需要在 SEND_RESPONSE_FILTER_ORDER 之前 官方推荐-1的这种写法
        return FilterConstants.SEND_RESPONSE_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() { //返回是否需要进行过滤
        return true;
    }

    @Override
    public Object run() throws ZuulException { //过滤执行的方法 这里是在请求之后在响应头中添加 XF-FOO 请求头
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletResponse httpServletResponse = requestContext.getResponse();
        httpServletResponse.addHeader("XF-FOO", UUID.randomUUID().toString());
        return null;
    }
}
```

2. token 拦截,token 拦截需要在所有请求之前,所以要使用前置过滤器

```
@Component
public class TokerFilter extends ZuulFilter {

    @Override
    public String filterType() { //声明请求类型为前置过滤
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.PRE_DECORATION_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() { //在请求的 param 中拦截 token 实际token 会放在 cookie 里或者 header 里面 拦截不到返回401
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletRequest request = requestContext.getRequest();
        String token = request.getParameter("token");
        if (StringUtils.isEmpty(token)) {
            requestContext.setSendZuulResponse(false);
            requestContext.setResponseStatusCode(HttpStatus.UNAUTHORIZED.value());
        }
        return null;
    }
}
```

3. 服务限流 采用 google 算法,算法机制 是令牌限流, 每秒中往桶里设置多少个令牌,每个请求会使用 一个令牌 一秒内请求超过令牌数量 那么其他的请求就会被拦截

```
import com.google.common.util.concurrent.RateLimiter;

@Component
public class RateLimiterFilter extends ZuulFilter {

    private static final RateLimiter RATE_LIMITER = RateLimiter.create(100); //google 的限流算法 每秒在请求中添加 100 个令牌

    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.SERVLET_DETECTION_FILTER_ORDER - 1;  //限流在所有 filter 前边
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        if (RATE_LIMITER.tryAcquire()) {
            return null;
        }
        throw new RuntimeException();
    }
}
```

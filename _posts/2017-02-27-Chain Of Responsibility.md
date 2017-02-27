---
layout: post
title:  "责任链(Chain Of Responsibility)"
categories: 设计模式
tags:  设计模式 责任链 filter 过滤器
excerpt: 责任链模式：使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。
---

* content
{:toc}

![](http://7xvdkv.com1.z0.glb.clouddn.com/image/design_pattern/inbetweening/domino.jpg)

## 什么是责任链模式？
> 责任链模式：使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。
> 将这个请求连成一个链，并沿着这条链传递该请求，直到有一个对象处理它为止。




## 优点和缺点

### 优点：

1. 降低耦合度。它将请求的发送者和接收者解耦。
2. 简化了对象。使得对象不需要知道链的结构。
3. 增强给对象指派职责的灵活性。通过改变链内的成员或者调动它们的次序，允许动态地新增或者删除责任。
4. 增加新的请求处理类很方便。

### 缺点：
1. 不能保证请求一定被接收。
2. 系统性能将受到一定影响，而且在进行代码调试时不太方便，可能会造成循环调用。
3. 可能不容易观察运行时的特征，有碍于除错。


## 什么情况下使用？

1. 在处理客户端请求时可能要根据不同的情况对请求添加不同的处理逻辑。
2. 有多个对象可以处理同一个请求，具体哪个对象处理该请求由运行时刻自动确定。


## 实现例子

#### 下面的例子是servlet过滤器类似的实现

servlet过滤器处理顺序：

![](http://7xvdkv.com1.z0.glb.clouddn.com/image/design_pattern/chainOfResponsibility_filterChain_explain.png)


例子的类结构：

![](http://7xvdkv.com1.z0.glb.clouddn.com/image/design_pattern/chainOfResponsibility_filterChain.jpg)


```
public class Request {
	String requestStr;

	public String getRequestStr() {
		return requestStr;
	}

	public void setRequestStr(String requestStr) {
		this.requestStr = requestStr;
	}

}
```

```
public class Response {
	String responseStr;

	public String getResponseStr() {
		return responseStr;
	}

	public void setResponseStr(String responseStr) {
		this.responseStr = responseStr;
	}

}
```

```

public interface Filter {
	public void doFilter(Request request, Response response,FilterChain chain);

}

```

```
import java.util.ArrayList;
import java.util.List;

public class FilterChain implements Filter {
	List<Filter> filters = new ArrayList<>();
    int index = 0;

    public FilterChain addFilter(Filter f) {
        this.filters.add(f);
        return this;
    }

    @Override
    public void doFilter(Request request, Response response, FilterChain chain) {
        if (index == filters.size())
            return;
        Filter f = filters.get(index);
        index++;
        f.doFilter(request, response, chain);
    }
}
```

```
/**
 * 过滤HTML中的脚本元素
 */
public class HTMLFilter implements Filter {

	@Override
	public void doFilter(Request request, Response response, FilterChain chain) {
		System.out.println("===> start HTMLFilter");
		request.requestStr = request.getRequestStr().replace("<", "[")
				.replace(">", "]");
		chain.doFilter(request, response, chain);
		System.out.println("===> end HTMLFilter");
		response.responseStr += " -> HTMLFilter";

	}

}
```

```
public class FaceFilter implements Filter {

	@Override
	public void doFilter(Request request, Response response, FilterChain chain) {
		System.out.println("===> start FaceFilter");
		request.requestStr = request.getRequestStr().replace(":)",
				"^V^");
		chain.doFilter(request, response, chain);

		System.out.println("===> end FaceFilter");
		response.responseStr += " -> FaceFilter";

	}

}
```

```
public class SesitiveFilter implements Filter {

	@Override
	public void doFilter(Request request, Response response, FilterChain chain) {
		System.out.println("===> start SesitiveFilter");
		request.requestStr = request.getRequestStr().replace("敏感", "**");
		chain.doFilter(request, response, chain);
		System.out.println("===> end SesitiveFilter");
		response.responseStr += " -> SesitiveFilter";

	}

}
```

```
public class Main {
	public static void main(String[] args) {
		String message = "敏感词，<script> 哈哈哈 :)";
		Request request = new Request();
		request.setRequestStr(message);
		Response response = new Response();
		response.setResponseStr("response");

		FilterChain fc = new FilterChain();
		fc.addFilter(new HTMLFilter()).addFilter(new SesitiveFilter());

		FilterChain fc2 = new FilterChain();
		fc2.addFilter(new FaceFilter());
		fc.addFilter(fc2);
		fc.doFilter(request, response, fc);

		System.out.println("request = " + request.getRequestStr());
		System.out.println("response = " + response.getResponseStr());
	}

}

// 结果输出：
// ===> start HTMLFilter
// ===> start SesitiveFilter
// ===> start FaceFilter
// ===> end FaceFilter
// ===> end SesitiveFilter
// ===> end HTMLFilter
// request = **词，[script] 哈哈哈 ^V^
// response = null -> FaceFilter -> SesitiveFilter -> HTMLFilter

```


#### 经典的责任链模式是链节点中记录下一个节点，然后向下传递，一旦处理成功，就直接返回。

![](http://7xvdkv.com1.z0.glb.clouddn.com/image/design_pattern/chainOfResponsibility_classic.jpg)

```

public abstract  class Handler {
	protected Handler successor;

	public void setSuccessor(Handler successor) {
		this.successor = successor;
	}

	public abstract void handlerRequest(Request request);
}

```

```
public class ConcreteHandler1 extends Handler{

	@Override
	public void handlerRequest(Request request) {
		if (request.getRequestStr().startsWith("handler-1")) {

			request.setRequestStr(request.getRequestStr().replace("handler-1", "handler-1111"));
			System.out.println("handler-1 process");
		} else if (successor != null) {
			successor.handlerRequest(request);
		}
	}
}

```

```
public class ConcreteHandler2 extends Handler{

	@Override
	public void handlerRequest(Request request) {
		if (request.getRequestStr().startsWith("handler-2")) {

			request.setRequestStr(request.getRequestStr().replace("handler-2", "handler-2222"));
			System.out.println("handler-2 process");
		} else if (successor != null) {
			successor.handlerRequest(request);
		}
	}
}
```

```
public class Main {
	public static void main(String[] args) {
		Handler h1 = new ConcreteHandler1();
		Handler h2 = new ConcreteHandler2();

		h1.setSuccessor(h2);

		Request request = new Request();
		request.setRequestStr("handler-2:哈哈");
		h1.handlerRequest(request);
		System.out.println("request : " + request.getRequestStr());
	}
}

// 结果输出：
// handler-2 process
// request : handler-2222:哈哈
```


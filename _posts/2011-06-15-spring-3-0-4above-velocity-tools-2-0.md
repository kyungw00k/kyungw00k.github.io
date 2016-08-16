---
layout: post
title: Spring 3.0.4(above) + Velocity Tools 2.0
date: 2011-06-15 18:07:37.000000000 +09:00
type: post
published: true
status: publish
categories:
- Programming
- Java
tags:
- "Spring 3"
- Velocity
---
Velocity Tools 2.0부터 XML형식이 새롭게 바뀌었다.

그런데 이상하게 Spring 3.0.4(above)에서 제대로 불러오지 못하는 버그가 있는것 같다.

다시말해 Request Scope와 Session Scope에 정의된 Tool을 쓸 수가 없는것이다.

열심히 구글링해 본 결과!

[http://www.elepha.info/elog/archives/305-configuring-velocity-tools-20-in-spring.html](http://www.elepha.info/elog/archives/305-configuring-velocity-tools-20-in-spring.html)

누군가 해결책을 내놓았다 `-_-;;`

```java
import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.velocity.context.Context;
import org.apache.velocity.tools.Scope;
import org.apache.velocity.tools.ToolboxFactory;
import org.apache.velocity.tools.config.XmlFactoryConfiguration;
import org.apache.velocity.tools.view.ViewToolContext;
import org.springframework.web.context.support.ServletContextResource;

/**
 * Velocity Tools 2.0부터 적용된 새로운 형태의 XML 형식을 Spring 3에서 제대로 지원하지 않는점을 수정
 * (Configures Velocity Toolbox View to use new XML Configuration in Spring)
 *
 * @author Gregor "hrax" Magdolen
 * @see http://www.elepha.info/elog/archives/305-configuring-velocity-tools-20-in-spring.html
 */
public class VelocityToolboxView extends
		org.springframework.web.servlet.view.velocity.VelocityToolboxView {

    @Override
	protected Context createVelocityContext(Map<String, Object> model,
			HttpServletRequest request, HttpServletResponse response)
			throws Exception {
		// Create a ViewToolContext instance since ChainedContext is deprecated
		// in Velocity Tools 2.0.
		ViewToolContext velocityContext = new ViewToolContext(
				getVelocityEngine(), request, response, getServletContext());
		velocityContext.putAll(model);

        // Load a Configuration and publish toolboxes to the context when
		// necessary
		if (getToolboxConfigLocation() != null) {
			XmlFactoryConfiguration cfg = new XmlFactoryConfiguration();
			cfg.read(new ServletContextResource(getServletContext(),
					getToolboxConfigLocation()).getURL());
			ToolboxFactory factory = cfg.createFactory();

            velocityContext
					.addToolbox(factory.createToolbox(Scope.APPLICATION));
			velocityContext.addToolbox(factory.createToolbox(Scope.REQUEST));
			velocityContext.addToolbox(factory.createToolbox(Scope.SESSION));
		}
		return velocityContext;
	}
}
```

위의 코드를 적절하게 넣고, 위 클래스를 `VelocityViewResolver` 설정에 `ViewClass`로 지정해주면 되겠다.

```xml

	<bean id="viewResolver"
		class="org.springframework.web.servlet.view.velocity.VelocityViewResolver"
		p:suffix=".vm" p:contentType="text/html; charset=UTF-8"
		p:toolboxConfigLocation="/WEB-INF/velocity-toolbox.xml"
		p:dateToolAttribute="dateTool" p:numberToolAttribute="numberTool"
		p:exposeRequestAttributes="true" p:exposeSessionAttributes="true"
		p:exposeSpringMacroHelpers="true"
		p:viewClass="kr.kyungw00k.velocity.VelocityToolboxView" />
```

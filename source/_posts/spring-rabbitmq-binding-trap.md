---
title: Spring rabbitMQ binding配置的“陷阱”
date: 2022-06-14 20:31:59
categories: 
    - 基础姿势
    - 框架和组件
tags:
	- RabbitMQ
	- Spring
---

## Spring RabbitMQ bindings

> exchange的`auto-declare`配置同时作用于binding，如果声明新的queue，但是绑定的exchange的auto-declare为false，则不会在既有的exchange上进行绑定

```xml
  <rabbit:queue id="queue0.id" name="queue.0.name"/>
  <rabbit:fanout-exchange name="exchange.0.name">
    <rabbit:bindings>
      <rabbit:binding queue="queue0.id"/>
    </rabbit:bindings>
  </rabbit:fanout-exchange>
```

解析binding的入口：
```java
    protected void doParseBindings(Element element, ParserContext parserContext,
			String exchangeName, Element bindings, AbstractExchangeParser parser) {
		if (bindings != null) {
			for (Element binding : DomUtils.getChildElementsByTagName(bindings, BINDING_ELE)) {
                // 解析rabbit:binding
				BeanDefinitionBuilder bindingBuilder = parser.parseBinding(exchangeName, binding,
						parserContext);
                // 这里是重点，解析bindings的时候，传入的是element，也就是rabbit:fanout-exchange对应的元素
				NamespaceUtils.parseDeclarationControls(element, bindingBuilder);
				BeanDefinition beanDefinition = bindingBuilder.getBeanDefinition();
				registerBeanDefinition(new BeanDefinitionHolder(beanDefinition, parserContext.getReaderContext()
						.generateBeanName(beanDefinition)), parserContext.getRegistry());
			}
		}
	}
```

每个binding的自动声明`shouldDeclare`属性的设置：
```java
	/**
	 * Parses 'auto-declare' and 'declared-by' attributes.
	 *
	 * @param element The element.
	 * @param builder The builder.
	 */
	public static void parseDeclarationControls(Element element, BeanDefinitionBuilder builder) {
        // 这里用exchange的auto-declare属性为每个binding设置shouldDeclare值
		NamespaceUtils.setValueIfAttributeDefined(builder, element, "auto-declare", "shouldDeclare");
		String admins = element.getAttribute("declared-by");
		if (StringUtils.hasText(admins)) {
			String[] adminBeanNames = admins.split(",");
			ManagedList<BeanReference> adminBeanRefs = new ManagedList<BeanReference>();
			for (String adminBeanName : adminBeanNames) {
				adminBeanRefs.add(new RuntimeBeanReference(adminBeanName.trim()));
			}
			builder.addPropertyValue("adminsThatShouldDeclare", adminBeanRefs);
		}
		NamespaceUtils.setValueIfAttributeDefined(builder, element, "ignore-declaration-exceptions");
	}
```

`shouldDeclare`在声明中的作用:
```java

	/**
	 * Remove any instances that should not be declared by this admin.
	 * @param declarables the collection of {@link Declarable}s.
	 * @return a new collection containing {@link Declarable}s that should be declared by this
	 * admin.
	 */
	private <T extends Declarable> Collection<T> filterDeclarables(Collection<T> declarables) {
		Collection<T> filtered = new ArrayList<T>();
		for (T declarable : declarables) {
			Collection<?> adminsWithWhichToDeclare = declarable.getDeclaringAdmins();
            // 此处只保留shouldDeclare=true的bean
			if (declarable.shouldDeclare() &&
					(adminsWithWhichToDeclare.isEmpty() || adminsWithWhichToDeclare.contains(this))) {
				filtered.add(declarable);
			}
		}
		return filtered;
	}
```

初始化的部分：
```java

	/**
	 * Declares all the exchanges, queues and bindings in the enclosing application context, if any. It should be safe
	 * (but unnecessary) to call this method more than once.
	 */
	public void initialize() {

		...

		final Collection<Exchange> exchanges = filterDeclarables(contextExchanges);
		final Collection<Queue> queues = filterDeclarables(contextQueues);
        // 这里过滤掉了shouldDeclare=false的binding
		final Collection<Binding> bindings = filterDeclarables(contextBindings);

		...

		this.rabbitTemplate.execute(new ChannelCallback<Object>() {

			@Override
			public Object doInRabbit(Channel channel) throws Exception {
				declareExchanges(channel, exchanges.toArray(new Exchange[exchanges.size()]));
				declareQueues(channel, queues.toArray(new Queue[queues.size()]));
                // 向rabbit声明binding的部分
				declareBindings(channel, bindings.toArray(new Binding[bindings.size()]));
				return null;
			}
		});
		this.logger.debug("Declarations finished");

	}
```
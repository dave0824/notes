## 问题描述
本地调用联通限速接口无误，部署到服务器上调用刚开始也无误，随着时间的推移，调用次数的增加，再次调用时报 400 Bad Request 错误。

## 问题代码
```java
private String sendRequest(String s) {
		try {
			// 请求头
			HttpHeaders headers = new HttpHeaders();
			// headers.set("Content-Type","text/html;charset=UTF-8");
			headers.add("Content-Type","text/html;charset=UTF-8");
			String xmlmsg = DesUtil.encryption(bssProperties.getPwd(), s);
			JSONObject encyptionRequest = new JSONObject();
			encyptionRequest.put("xmlmsg",xmlmsg);
			encyptionRequest.put("channelcode",bssProperties.getChannelCode());
			HttpEntity<Map<String, Object>> request = new HttpEntity<>(encyptionRequest, headers);
			
			MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter = new MappingJackson2HttpMessageConverter();
			mappingJackson2HttpMessageConverter.setSupportedMediaTypes(Arrays.asList(MediaType.TEXT_HTML));
			restTemplate.getMessageConverters().add(mappingJackson2HttpMessageConverter);
			
			ResponseEntity<String> entity = restTemplate.postForEntity(bssProperties.getUrl(), request,String.class); 
			String body = entity.getBody();
			JSONObject bodyJson = JSONObject.parseObject(body);
			String xmlmsgRes = bodyJson.getString("xmlmsg");
			String str = DesUtil.decrypt(xmlmsgRes, bssProperties.getPwd());
			logger.info("响应结果-----> " + str);
			return str;
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}	
		
	}
```

## 问题产生原因

由于限速接口的请求头 Content-Type 要设置成 text/html;charset=UTF-8 类型，所以就在方法上添加了转换器，由于这个转换器是 List 集合，导致每次请求都会向 messageConverters 中添加转换器，随着请求次数的增多，messageConverters 越来越大，这会导致 Accept 标头的内容不断增长，导致了 400 Bad Request 错误的产生。  

```java
private final List<HttpMessageConverter<?>> messageConverters = new ArrayList<>();
```

## 问题解决

将添加转化器的代码放到注册 bean 对象中

```java
@Bean
	public RestTemplate restTemplate(){
		RestTemplate restTemplate = new RestTemplate();
		MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter = new MappingJackson2HttpMessageConverter();
		mappingJackson2HttpMessageConverter.setSupportedMediaTypes(Arrays.asList(MediaType.TEXT_HTML));
		restTemplate.getMessageConverters().add(mappingJackson2HttpMessageConverter);
		return restTemplate;
	}
```

## 参考资料

- https://stackoverflow.com/questions/29606808/over-time-of-period-got-400-bad-request-for-resttemplate/35830569
- https://www.jianshu.com/p/95bf08696cd7
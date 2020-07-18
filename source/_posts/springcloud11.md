---
title: SpringCloud系列之--Eureka注册探秘
date: 2020-07-18 14:11:00
updated: 2020-07-18 14:11:00
tags: SpringCloud
categories: SpringCloud
keywords: Java, SpringCloud, 微服务
type: 
description: Eureka注册探秘。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img11.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img11.jpg
aside: true
toc: true
toc_number: true
auto_open: true
copyright: false
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
> 转载自：[Eureka中RetryableClientQuarantineRefreshPercentage参数探秘](http://www.spring4all.com/article/180 "Eureka中RetryableClientQuarantineRefreshPercentage参数探秘")

# 前言

我们知道Eureka分为两部分，`Eureka Server`和`Eureka Client`。Eureka Server充当注册中心的角色，Eureka Client相对于Eureka Server来说是客户端，需要将自身信息注册到注册中心。本文主要介绍的就是在Eureka Client注册到Eureka Server时RetryableClientQuarantineRefreshPercentage参数的使用技巧。

# Eureka Client注册过程分析

Eureka Client注册到Eureka Server时，首先遇到第一个问题就是Eureka Client端要知道Server的地址，这个参数对应的是`eureka.client.service-url.defaultZone`，举个例子，在Eureka Client的properties文件中配置如下：
```Properties
	eureka.client.service-url.defaultZone=http://localhost:8761/eureka,http://localhost:8762/eureka,http://localhost:8763/eureka,http://localhost:8764/eureka
```
如上所示，Eureka Client配置对应的Eureka Server地址分别是8761、8762、8763、8764。这里存在两个问题：

1. Eureka Client会将自身信息分别注册到这四个地址吗？
2. Eureka Clinent注册机制是怎样的？

源码面前一目了然，带着这两个问题我们通过源码来解答这两个问题。Eureka Client在启动的时候注册源码如下：
RetryableEurekaHttpClient中的execut方法
```Java
	 @Override
	  protected <R> EurekaHttpResponse<R> execute(RequestExecutor<R> requestExecutor) {
	      List<EurekaEndpoint> candidateHosts = null;
	      int endpointIdx = 0;
	      for (int retry = 0; retry < numberOfRetries; retry++) {
	          EurekaHttpClient currentHttpClient = delegate.get();
	          EurekaEndpoint currentEndpoint = null;
	          if (currentHttpClient == null) {
	              if (candidateHosts == null) {
	                  candidateHosts = getHostCandidates();
	                  if (candidateHosts.isEmpty()) {
	                      throw new TransportException("There is no known eureka server; cluster server list is empty");
	                  }
	              }
	              if (endpointIdx >= candidateHosts.size()) {
	                  throw new TransportException("Cannot execute request on any known server");
	              }
	 
	              currentEndpoint = candidateHosts.get(endpointIdx++);
	              currentHttpClient = clientFactory.newClient(currentEndpoint);
	          }
	 
	          try {
	              EurekaHttpResponse<R> response = requestExecutor.execute(currentHttpClient);
	              if (serverStatusEvaluator.accept(response.getStatusCode(), requestExecutor.getRequestType())) {
	                  delegate.set(currentHttpClient);
	                  if (retry > 0) {
	                      logger.info("Request execution succeeded on retry #{}", retry);
	                  }
	                  return response;
	              }
	              logger.warn("Request execution failure with status code {}; retrying on another server if available", response.getStatusCode());
	          } catch (Exception e) {
	              logger.warn("Request execution failed with message: {}", e.getMessage());  // just log message as the underlying client should log the stacktrace
	          }
	 
	          // Connection error or 5xx from the server that must be retried on another server
	          delegate.compareAndSet(currentHttpClient, null);
	          if (currentEndpoint != null) {
	              quarantineSet.add(currentEndpoint);
	          }
	      }
	      throw new TransportException("Retry limit reached; giving up on completing the request");
	  }
```
按照我的理解，代码精简后内容如下：
```Java
	int endpointIdx = 0;
	//用来保存所有Eureka Server信息(8761、8762、8763、8764)
	List<EurekaEndpoint> candidateHosts = null;
	//numberOfRetries的值代码写死默认为3次
	for (int retry = 0; retry < numberOfRetries; retry++) {
	    /**
	     *首次进入循环时，获取全量的Eureka Server信息(8761、8762、8763、8764)
	     */
	    if (candidateHosts == null) {
	        candidateHosts = getHostCandidates();
	    }
	    /**
	     *通过endpointIdx自增，依次获取Eureka Server信息，然后发送
	     *注册的Post请求.
	     */
	    currentEndpoint = candidateHosts.get(endpointIdx++);
	    currentHttpClient = clientFactory.newClient(currentEndpoint);
	    try {
	       /**
	         *发送注册的Post请求动作，注意如果成功，则跳出循环，如果失败则
	         *根据endpointIdx依次获取下一个Eureka Server.
	         */
	        response = requestExecutor.execute(currentHttpClient);
	        return respones;
	    } catch (Exception e) {
	        //向注册中心(Eureka Server)发起注册的post出现异常时，打印日志...
	    }
	    //如果此次注册动作失败，将当前的信息保存到quarantineSet中(一个Set集合)
	    if (currentEndpoint != null) {
	        quarantineSet.add(currentEndpoint);
	    }
	}
	//如果都失败,则以异常形式抛出...
	throw new TransportException("Retry limit reached; giving up on completing the request");
```
上面代码中还有一个方法很重要就是 List<EurekaEndpoint> candidateHosts = getHostCandidates();接下来看下getHostCandidates()方法源码。
```Java
    private List<EurekaEndpoint> getHostCandidates() {
        List<EurekaEndpoint> candidateHosts = clusterResolver.getClusterEndpoints();
        quarantineSet.retainAll(candidateHosts);
 
        // If enough hosts are bad, we have no choice but start over again
        int threshold = (int) (candidateHosts.size() * transportConfig.getRetryableClientQuarantineRefreshPercentage());
        if (quarantineSet.isEmpty()) {
            // no-op
        } else if (quarantineSet.size() >= threshold) {
            logger.debug("Clearing quarantined list of size {}", quarantineSet.size());
            quarantineSet.clear();
        } else {
            List<EurekaEndpoint> remainingHosts = new ArrayList<>(candidateHosts.size());
            for (EurekaEndpoint endpoint : candidateHosts) {
                if (!quarantineSet.contains(endpoint)) {
                    remainingHosts.add(endpoint);
                }
            }
            candidateHosts = remainingHosts;
        }
 
        return candidateHosts;
    }
```
按照我的理解，将代码精简下，只包括关键逻辑，内容如下：
```Java
	private List<EurekaEndpoint> getHostCandidates() {
	    /**
	     * 获取所有defaultZone配置的注册中心信息(Eureka Server)，
	     * 在本文例子中代表4个(8761、8762、8763、8764)Eureka Server
	     */
	    List candidateHosts = clusterResolver.getClusterEndpoints();
	    /**
	     * quarantineSet这个Set集合中保存的是不可用的Eureka Server
	     * 此处是拿不可用的Eureka Server与全量的Eureka Server取交集
	     */
	    quarantineSet.retainAll(candidateHosts);
	    /**
	     * 根据RetryableClientQuarantineRefreshPercentage参数计算阈值
	     * 该阈值后续会和quarantineSet中保存的不可用的Eureka Server个数
	     * 作比较，从而判断是否返回全量的Eureka Server还是过滤掉不可用的
	     * Eureka Server。
	     */
	    int threshold = 
	       (int) (
	        candidateHosts.size()
	              *
	        transportConfig.getRetryableClientQuarantineRefreshPercentage()
	        );
	    if (quarantineSet.isEmpty()) {
	        /**
	         * 首次进入的时候，此时quarantineSet为空，直接返回全量的
	         * Eureka Server列表
	         */
	    } else if (quarantineSet.size() >= threshold) {
	        /**
	         * 将不可用的Eureka Server与threshold值相比较，如果不可
	         * 用的Eureka Server个数大于阈值，则将之前保存的Eureka
	         * Server内容直接清空，并返回全量的Eureka Server列表。
	         */
	        quarantineSet.clear();
	    } else {
	        /**
	         * 通过quarantineSet集合保存不可用的Eureka Server来过滤
	         * 全量的EurekaServer，从而获取此次Eureka Client要注册要
	         * 注册的Eureka Server实例地址。
	         */
	        List<EurekaEndpoint> remainingHosts = new ArrayList<>(candidateHosts.size());
	        for (EurekaEndpoint endpoint : candidateHosts) {
	            if (!quarantineSet.contains(endpoint)) {
	                remainingHosts.add(endpoint);
	            }
	        }
	        candidateHosts = remainingHosts;
	    }
	    return candidateHosts;
	}
```
通过源码分析，我们现在初步知道，当Eureka Client向Eureka Server发起注册请求的时候(根据defaultZone寻找Eureka Server列表)，如果有一次请求注册成功，那么后续就不会在向其他Eureka Server发起注册请求。以本文为例，注册中心有四个(8761、8762、8763、8764)。如果8761对应的Eureka Server服务的状态是UP，那么Eureka Client向该注册中心注册成功后，不会再向(8762、8763、8764)对应的Eureka Server发起注册请求(对应程序是在for循环中直接return respones)。

**说到这里又引出来另外一个问题，如果8761这个Eureka Server是down掉的呢？**

根据源码我们可知Eureka Client首次会向8761这个Server发起注册请求，如果该Server的状态是down，那么它会将该Server保存到quarantineSet这个Set集合中，然后再次访问8762这个Eureka Server，如果8762这个Server的状态依旧是down，它也会把这个Server保存到quarantineSet这个Set集合中，然后继续访问8763这个Server，如果8763这个Server的状态依旧是down，此时除了会将其保存到quarantineSet这个Set集合中之外，还会跳出本次循环。从而结束此次注册过程。

说道这里有人要问接下来会不会向8764这个Server发起注册，答案是否定的，因为循环的次数默认是3次。所以即使8764这个Server的状态是UP，它也不会接收到来自Eureka Client发起的注册信息。

Eureka Client向Eureka Server发起注册信息的过程除了在Eureka Client启动的时候触发，还有另外一种方式，就是后台定时任务。

假设我们上面描述的场景是在Eureka Client启动的时候，因为在启动的时候注册这个过程全部失败了，当后台定时任务执行时，还会进入该注册流程。注意此时quarantineSet的值为3(8761、8762、8763之前注册失败的Eureka Server)。

所以当程序再次进入getHostCandidates()方法时，if (quarantineSet.isEmpty())这个方法是不满足的，接下来会走else if (quarantineSet.size() >= threshold)这个判断，如果这个判断成立，那么会将quarantineSet集合清空，同时返回全量的Eureka Server列表，如果这个判断不成立，会拿quarantineSet集合中保存的内容去过滤Eureka Server的全量列表。以本文为例：quarantineSet中保存的是(8761、8762、8763)三个Eureka Server，Eureka Server全量列表的内容是(8761、8762、8763、8764)四个Eureka Server，过滤后返回的结果为8764这个Eureka Server。

在本文的例子中8761、8762、8763这三个Eureka Server的状态是down而8764这个Eureka Server的状态是UP，我们其实是想走到最后的else分支，从而完成过滤操作，并最终得到8764这个Server，遗憾的是它并不会走到这个分支，而是被上面的else if (quarantineSet.size() >= threshold)这个分支所拦截，返回的依旧是全量的Eureka Server列表。这样造成的后果就是Eureka Client依旧会依次向(8761、8762、8763)这三个down的Eureka Server发起注册请求。

那么问题的关键在哪里呢？问题的关键就是threshold这个值的由来，因为此时quarantineSet.size()的值为3，而3这个值大于threshold，从而导致，会将quarantineSet集合清空，返回全量的Server列表。

我们知道threshold这个值是根据全量的Eureka Server列表乘以一个可配置的参数计算出来的，在本文的例子当中，我的properties文件中除了defaultZone之外并没有配置这个参数，那么也就是说这个参数是有默认值的，通过源码我们了解到，这个默认值是0.66。具体源码如下：
```Java
	final class PropertyBasedTransportConfigConstants {
	    /**
	     *省略部分源码
	     */
	    static class Values {
	        static final int SESSION_RECONNECT_INTERVAL = 20*60;
	        //默认值为0.66
	        static final double QUARANTINE_REFRESH_PERCENTAGE = 0.66;
	        static final int DATA_STALENESS_TRHESHOLD = 5*60;
	        static final int ASYNC_RESOLVER_REFRESH_INTERVAL = 5*60*1000;
	        static final int ASYNC_RESOLVER_WARMUP_TIMEOUT = 5000;
	        static final int ASYNC_EXECUTOR_THREADPOOL_SIZE = 5;
	    }
	}
	/**
	 *@return the percentage of the full endpoints set above which the   
	 *quarantine set is cleared in the range [0, 1.0]
	 */
	double getRetryableClientQuarantineRefreshPercentage();
```
看到这里就不难理解了，因为这个值是0.66而此时全量的Eureka Server值为4。计算之后的值为2，而由于注册的for循环为3次，所以当第二次发起注册流程的时候quarantineSet的值始终大于threshold。这样就会导致一个问题，就是如果8761、8762、8763一直是down即使8764一直是好的，那么Eureka Client也不会注册成功。而且这个参数值的区间为0到1.

既然通过源码分析我们找到了问题根源，其实对应的我们也找到了解决这个问题的办法，就是对应把这个参数值调大些。
这个值在properties中对应的写法如下：
```Properties
eureka.client.transport.retryableClientQuarantineRefreshPercentage = xxx 
```
接下来我们修改下properties文件，修改后的内容如下：
```Properties
eureka.client.service-url.defaultZone=http://localhost:8761/eureka,http://localhost:8762/eureka,http://localhost:8763/eureka,http://localhost:8764/eureka
eureka.client.transport.retryableClientQuarantineRefreshPercentage=1
```
接下来按照这个配置再次回顾下上面的流程：

- Eureka Client启动时进行注册(8761、8762、8763的状态是down)，所以此时quarantineSet的值为3.
- 接下来在定时任务中又触发注册事件，此时因为参数的值从0.66调整为1。所以计算出的threshold的值为4。而此时quarantineSet的值为3。所以不会进入到else if (quarantineSet.size() >= threshold)分支，而是会进入最后的esle分支。
- 在else分支中会完成过滤功能，最终返回的list中的结果只有一个就是8764这个Eureka Server。
- Eureka Client向8764这个Eureka Server发起注册请求，得到成功相应，并返回。

# 遗留问题

说道这里我们感觉好像是解决了这个问题，那么问一个问题，这个参数值可以设置的无限大吗？

比如我将这个参数值设置为10，虽然javaDoc中说明这个参数值的范围在0-1之间，但是并没有说明如果将这个参数调整大于1会出现什么情况。接下来按照上面的流程我们分析下：

之前我们分析的流程中的前提是8761、8762、8763这三台Server的状态是down而8764这个server的状态是up，现在我们修改下这个前提。

假设一开始8761、8762、8763、8764这四台Eureka Server的状态都是down。Eureka Client启动时进行注册(8761、8762、8763的状态是down)，所以此时quarantineSet的值为3.

- 接下来在定时任务中又触发注册事件，此时因为参数的值从0.66调整为10。所以计算出的threshold的值为40。而此时quarantineSet的值为3。所以不会进入到else if (quarantineSet.size() >= threshold)分支，而是会进入最后的esle分支。
- 在else分支中会完成过滤功能，最终返回的list中的结果只有一个就是8764这个Eureka Server。
- Eureka Client向8764这个Eureka Server发起注册请求，因为此时8764的状态也是down导致注册失败，此时quarantineSet中的内容是(8761、8762、8763、8764)
- 当定时任务再次触发时if (quarantineSet.isEmpty())这个分支不会进入，因为此时quarantineSet的值为4
- else if (quarantineSet.size() >= threshold)这分支也不会进入因为threshold的值为40
- 最终会进入else分支，这个分支原本的含义是想通过quarantineSet来充当过滤器，从全量的Eureka Server中过滤掉之前状态为down的Eureka Server，但是由于quarantineSet的值现在已经是全量，导致过滤后的结果返回的是一个空的list。即使此时Eureka Server列表(8761、8762、8763、8764)任何一个Server的状态变为UP，该Eureka Client也不可能完成注册事件。


# 解决办法

上面出现的那个问题，根本原因个人认为是由于eureka.client.transport.retryableClientQuarantineRefreshPercentage 参数过大而源码中没有校验，从而导致没有进入else if (quarantineSet.size() >= threshold)的逻辑分支，因为此时如果quarantineSet中的值已经达到了所有Eureka Server列表，那么此时我们希望的是将这个Set集合清空，从而再次返回全量的Eureka Server列表，也就是说再重新来一次注册流程。

所以基于上面的分析，个人认为在源码的getHostCandidates增加下校验，具体代码如下：
```Java
	private List<EurekaEndpoint> getHostCandidates() {
	    List<EurekaEndpoint> candidateHosts = clusterResolver.getClusterEndpoints();
	    quarantineSet.retainAll(candidateHosts);
	 
	    // If enough hosts are bad, we have no choice but start over again
	    int threshold = (int) (candidateHosts.size() * transportConfig.getRetryableClientQuarantineRefreshPercentage());
	 
	    /**
	     * 增加判断如果threshold的值过大，即超过Eureka Server
	     * 列表的数量，那么将其再次赋值，赋值的内容为Eureka Server
	     * 列表的数量。
	     */
	    if (threshold > candidateHosts.size()) {
	          threshold = candidateHosts.size();
	    }
	 
	    if (quarantineSet.isEmpty()) {
	        // no-op
	    } else if (quarantineSet.size() >= threshold) {
	        logger.debug("Clearing quarantined list of size {}", quarantineSet.size());
	        quarantineSet.clear();
	    } else {
	        List<EurekaEndpoint> remainingHosts = new ArrayList<>(candidateHosts.size());
	        for (EurekaEndpoint endpoint : candidateHosts) {
	            if (!quarantineSet.contains(endpoint)) {
	                remainingHosts.add(endpoint);
	            }
	        }
	        candidateHosts = remainingHosts;
	    }
	 
	    return candidateHosts;
	}
```
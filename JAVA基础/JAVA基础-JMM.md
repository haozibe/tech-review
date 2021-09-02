### 线程通信

- 共享内存
  - 在共享内存的并发模型中线程之间共享线程的公共数据状态，线程之前通过读写内存中的公共内存区域来进行消息的传递，典型的共享内存通信方式就是通过共享对象进行通信
- 消息传递
  - 通过socket，信号量，管道，信号，消息队列这几种方式，在消息传递的并发模型中，线程之间是没有共享状态的，线程之间必须通过明确发送消息来显式的进行通信。





``` 4     public ChangeRequestWrapper(HttpServletRequest request) { 5         super(request); 6         parameterMap = request.getParameterMap(); 7     }
      public HeaderMapRequestWrapper(HttpServletRequest request) {
          super(request);
         parameterMap = request.getParameterMap();
         }
 
 7     }
```
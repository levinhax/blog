在 Web 应用中，浏览器和服务器之间使用的是请求 / 响应的交互模式。浏览器发出请求，服务器根据收到的请求来生成相应的响应。浏览器再对收到的响应进行处理，展现给用户。响应的格式可能是 HTML、XML 或 JSON 等。

随着 REST 架构风格和 AJAX 的流行，服务器更多地使用 JSON 作为响应的数据格式。Web 应用使用 XMLHttpRequest 对象来发送请求，并根据服务器端返回的数据，对页面的内容进行动态更新。通常来说，用户在页面上的操作，比如点击或移动鼠标，会触发相应的事件。由 XMLHttpRequest 对象来发出请求，得到服务器响应之后进行页面的局部更新。

但是，服务器端产生的数据变化不能及时地通知浏览器，而是需要等到下次请求发出时才能被浏览器获取。对于某些对数据实时性要求很高的应用来说，这种延迟是不能接受的。

为了满足这类应用的需求，就需要有某种方式能够从服务器端推送数据给浏览器，以保证服务器端的数据变化可以在第一时间通知给用户。目前常见的解决办法有不少，主要可以分成两类。这两类方法的区别在于是否基于 HTTP 协议来实现。不使用 HTTP 协议的做法是使用 HTML 5 新增的 WebSocket 规范，而使用 HTTP 协议的做法则包括简易轮询、COMET 技术和本文中要介绍的 HTML 5 服务器推送事件。

### 背景

WebSocket 使用的是套接字连接，基于 TCP 协议。使用 WebSocket 之后，实际上在服务器端和浏览器之间建立一个套接字连接，可以进行双向的数据传输。WebSocket 的功能是很强大的，使用起来也灵活，可以适用于不同的场景。不过 WebSocket 技术也比较复杂，包括服务器端和浏览器端的实现都不同于一般的 Web 应用。

除了 WebSocket 之外，其他的实现方式是基于 HTTP 协议来达到实时推送的效果。第一种做法是简易轮询，即浏览器端定时向服务器端发出请求，来查询是否有数据更新。这种做法比较简单，可以在一定程度上解决问题。不过对于轮询的时间间隔需要进行仔细考虑。轮询的间隔过长，会导致用户不能及时接收到更新的数据；轮询的间隔过短，会导致查询请求过多，增加服务器端的负担。

COMET 技术改进了简易轮询的缺点，使用的是长轮询。长轮询的方式在每次请求时，服务器端会保持该连接在一段时间内处于打开状态，而不是在响应完成之后就立即关闭。这样做的好处是在连接处于打开状态的时间段内，服务器端产生的数据更新可以被及时地返回给浏览器。当上一个长连接关闭之后，浏览器会立即打开一个新的长连接来继续请求。不过 COMET 技术的实现在服务器端和浏览器端都需要第三方库的支持。

综合比较上面提到的 4 种不同的技术，简易轮询由于其本身的缺陷，并不推荐使用。COMET 技术并不是 HTML 5 标准的一部分，从兼容标准的角度出发，也不推荐使用。WebSocket 规范和服务器推送技术都是 HTML 5 标准的组成部分，在主流浏览器上都提供了原生的支持，是推荐使用的。不过 WebSocket 规范更加复杂一些，适用于需要进行复杂双向数据通讯的场景。对于简单的服务器数据推送的场景，使用服务器推送事件就足够了。

### 规范

Server-sent Events 规范是 HTML 5 规范的一个组成部分，主要由两个部分组成：第一个部分是服务器端与浏览器端之间的通讯协议，第二部分则是在浏览器端可供 JavaScript 使用的 EventSource 对象。

通讯协议是基于纯文本的简单协议。服务器端的响应的内容类型是“text/event-stream”。响应文本的内容可以看成是一个事件流，由不同的事件所组成。每个事件由类型和数据两部分组成，同时每个事件可以有一个可选的标识符。不同事件的内容之间通过仅包含回车符和换行符的空行（“\r\n”）来分隔。每个事件的数据可能由多行组成。

服务器端响应的示例：
```
data: first event

data: second event
id: 100

event: myevent
data: third event
id: 101

: this is a comment
data: fourth event
data: fourth event continue
```

对于服务器端返回的响应，浏览器端需要在 JavaScript 中使用 EventSource 对象来进行处理。EventSource 使用的是标准的事件监听器方式，只需要在对象上添加相应的事件处理方法即可。
名称 | 说明 | 事件处理方法
--|--|--
open | 当成功与服务器建立连接时产生 | onopen
message | 当收到服务器发送的事件时产生 | onmessage
error | 当出现错误时产生 | onerror

```
es.onmessage = function(e) {
    console.log(e.data);
};

es.addEventListener('myevent', function(e) {
    console.log(e.data);
});
```

### 服务器端和浏览器端实现

从对通讯协议的描述可以看出，服务器端推送事件是一个比较简单的协议。服务器端的实现也相对比较简单，只需要按照协议规定的格式，返回响应内容即可。

#### 服务器端实现

服务器端的实现由两部分组成：一部分是用来产生数据的 org.eclipse.jetty.servlets.EventSource 接口的实现，另一部分是作为浏览器访问端点的继承自 org.eclipse.jetty.servlets.EventSourceServlet 类的 servlet 实现。
```
public class MovementEventSource implements EventSource {
     
    private int width = 800;
    private int height = 600;
    private int stepMax = 5;
    private int x = 0;
    private int y = 0;
    private Random random = new Random();
    private Logger logger = Logger.getLogger(getClass().getName());
     
    public MovementEventSource(int width, int height, int stepMax) {
        this.width = width;
        this.height = height;
        this.stepMax = stepMax;
        this.x = random.nextInt(width);
        this.y = random.nextInt(height);
    }
 
    @Override
    public void onOpen(Emitter emitter) throws IOException {
        query(emitter); //开始生成位置信息
    }
 
    @Override
    public void onResume(Emitter emitter, String lastEventId)
            throws IOException {
        updatePosition(lastEventId); //更新起始位置
        query(emitter);  //开始生成位置信息
    }
     
    //根据Last-Event-Id来更新起始位置
    private void updatePosition(String id) {
        if (id != null) {
            String[] pos = id.split(",");
            if (pos.length > 1) {
                int xPos = -1, yPos = -1;
                try {
                    xPos = Integer.parseInt(pos[0], 10);
                    yPos = Integer.parseInt(pos[1], 10);
                } catch (NumberFormatException e) {
                     
                }
                if (isValidMove(xPos, yPos)) {
                    x = xPos;
                    y = yPos;
                }
            }
        }
    }
     
    private void query(Emitter emitter) throws IOException {
        emitter.comment("Start sending movement information.");
        while(true) {
            emitter.comment("");
            move(); //移动位置
            String id = String.format("%s,%s", x, y);
            emitter.id(id); //根据位置生成事件标识符
            emitter.data(id); //发送位置信息数据
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                logger.log(Level.WARNING, \
               "Movement query thread interrupted. Close the connection.", e);
                break;
            }
        }
        emitter.close(); //当循环终止时，关闭连接
    }
 
    @Override
    public void onClose() {
         
    }
     
    //获取下一个合法的移动位置
    private void move() {
        while (true) {
            int[] move = getMove();
            int xNext = x + move[0];
            int yNext = y + move[1];
            if (isValidMove(xNext, yNext)) {
                x = xNext;
                y = yNext;
                break;
            }
        }
    }
 
    //判断当前的移动位置是否合法
    private boolean isValidMove(int x, int y) {
        return x >= 0 && x <= width && y >=0 && y <= height;
    }
     
    //随机生成下一个移动位置
    private int[] getMove() {
        int[] xDir = new int[] {-1, 0, 1, 0};
        int[] yDir = new int[] {0, -1, 0, 1};
        int dir = random.nextInt(4);
        return new int[] {xDir[dir] * random.nextInt(stepMax), \
           yDir[dir] * random.nextInt(stepMax)};
    }
}
```

#### 浏览器端实现

浏览器端的实现也比较简单，只需要创建出 EventSource 对象，并添加相应的事件处理方法即可。

```
var es = new EventSource('sse/movement'); 
es.addEventListener('message', function(e) { 
    var pos = e.data.split(','), x = pos[0], y = pos[1]; 
    $('#box').css({ 
        left : x + 'px', 
        top : y + 'px' 
        }); 
    });
```

### IE 支持

使用浏览器原生的 EventSource 对象的一个比较大的问题是 IE 并不提供支持。为了在 IE 上提供同样的支持，一般有两种办法。第一种办法是在其他浏览器上使用原生 EventSource 对象，而在 IE 上则使用简易轮询或 COMET 技术来实现；另外一种做法是使用 polyfill 技术，即使用第三方提供的 JavaScript 库来屏蔽浏览器的不同。本文使用的是 polyfill 技术，只需要在页面中加载第三方 JavaScript 库即可。

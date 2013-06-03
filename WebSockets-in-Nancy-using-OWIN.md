**Note: Requires OWIN host. More information on it can be found at [Hosting Nancy with OWIN](Hosting-nancy-with-owin)**

This is based on [Owin WebSocket Extensions v0.4.0](http://owin.org/extensions/owin-WebSocket-Extension-v0.4.0.htm).

It is recommended to use [SignalR](http://www.asp.net/signalr) if possible.

**Create an index.html file in views folder**
```html
<!DOCTYPE html>
<head>
    <meta charset="utf-8" />
    <title>Nancy + Owin WebSocket Test</title>
</head>
<body>
    <script>
        //var wsUri = "ws://echo.websocket.org/";
        var wsUri = "ws://localhost:1901/websocket";
        var output, websocket;
        function init() {
            output = document.getElementById("output");
            testWebSocket();
        }
        function testWebSocket() {
            websocket = new WebSocket(wsUri, ['echo', 'chat']);
            websocket.onopen = function (evt) { onOpen(evt); };
            websocket.onclose = function (evt) { onClose(evt); };
            websocket.onmessage = function (evt) { onMessage(evt); };
            websocket.onerror = function (evt) { onError(evt); };
        }
        function onOpen(evt) {
            writeToScreen("CONNECTED");
            doSend("WebSocket rocks");
        }
        function onClose(evt) {
            writeToScreen("DISCONNECTED");
        }
        function onMessage(evt) {
            writeToScreen('<span style="color: blue;">RESPONSE: ' + evt.data + '</span>');
            websocket.close();
        }
        function onError(evt) {
            writeToScreen('<span style="color: red;">ERROR:</span> ' + evt.data);
        }
        function doSend(message) {
            writeToScreen("SENT: " + message);
            websocket.send(message);
        }
        function writeToScreen(message) {
            var pre = document.createElement("p");
            pre.style.wordWrap = "break-word";
            pre.innerHTML = message;
            output.appendChild(pre);
        }
        window.addEventListener("load", init, false);
    </script>
    <h2>WebSockets on Nancy using OWIN WebSocket Extension</h2>
    <p>Requires ASP.NET and IIS 8+ on Win8+</p>
    <div id="output"></div>

</body>
</html>
```

**Create a module with route that renders the view and handles the actual web socket connection**

```c#
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Net.WebSockets;
using System.Text;
using System.Threading;
using Nancy;
using Nancy.Owin;

using WebSocketAccept = System.Action<
           System.Collections.Generic.IDictionary<string, object>, // WebSocket Accept parameters
           System.Func< // WebSocketFunc callback
               System.Collections.Generic.IDictionary<string, object>, // WebSocket environment
               System.Threading.Tasks.Task>>;

using WebSocketSendAsync = System.Func<
               System.ArraySegment<byte>, // data
               int, // message type
               bool, // end of message
               System.Threading.CancellationToken, // cancel
               System.Threading.Tasks.Task>;
    // closeStatusDescription

using WebSocketReceiveAsync = System.Func<
            System.ArraySegment<byte>, // data
            System.Threading.CancellationToken, // cancel
            System.Threading.Tasks.Task<
                System.Tuple< // WebSocketReceiveTuple
                    int, // messageType
                    bool, // endOfMessage
                    int?, // count
                    int?, // closeStatus
                    string>>>; // closeStatusDescription

using WebSocketCloseAsync = System.Func<
            int, // closeStatus
            string, // closeDescription
            System.Threading.CancellationToken, // cancel
            System.Threading.Tasks.Task>;

namespace WebApplication6
{
    public class WebSocketModule : NancyModule
    {
        public WebSocketModule()
        {
            Get["/"] = x => View["index"];

            Get["/websocket"] = x => {

                var env = (IDictionary<string, object>)Context.Items[NancyOwinHost.RequestEnvironmentKey];

                object temp;
                if (env.TryGetValue("websocket.Accept", out temp) && temp != null) // check if the owin host supports web sockets
                {
                    var wsAccept = (WebSocketAccept)temp;
                    var requestHeaders = GetValue<IDictionary<string, string[]>>(env, "owin.RequestHeaders");

                    Dictionary<string, object> acceptOptions = null;
                    string[] subProtocols;
                    if (requestHeaders.TryGetValue("Sec-WebSocket-Protocol", out subProtocols) && subProtocols.Length > 0)
                    {
                        acceptOptions = new Dictionary<string, object>();
                        // Select the first one from the client
                        acceptOptions.Add("websocket.SubProtocol", subProtocols[0].Split(',').First().Trim());
                    }

                    wsAccept(acceptOptions, async wsEnv => {
                        var wsSendAsync = (WebSocketSendAsync)wsEnv["websocket.SendAsync"];
                        var wsRecieveAsync = (WebSocketReceiveAsync)wsEnv["websocket.ReceiveAsync"];
                        var wsCloseAsync = (WebSocketCloseAsync)wsEnv["websocket.CloseAsync"];
                        var wsVersion = (string)wsEnv["websocket.Version"];
                        var wsCallCancelled = (CancellationToken)wsEnv["websocket.CallCancelled"];

                        // note: make sure to catch errors when calling sendAsync, receiveAsync and closeAsync
                        // for simiplicity this code does not handle errors
                        var buffer = new ArraySegment<byte>(new byte[6]);
                        while (true)
                        {
                            var webSocketResultTuple = await wsRecieveAsync(buffer, wsCallCancelled);
                            int wsMessageType = webSocketResultTuple.Item1;
                            bool wsEndOfMessge = webSocketResultTuple.Item2;
                            int? count = webSocketResultTuple.Item3;
                            int? closeStatus = webSocketResultTuple.Item4;
                            string closeStatusDescription = webSocketResultTuple.Item5;

                            Debug.Write(Encoding.UTF8.GetString(buffer.Array, 0, count.Value));

                            await wsSendAsync(new ArraySegment<byte>(buffer.ToArray(), 0, count.Value), 1, wsEndOfMessge, wsCallCancelled);

                            if (wsEndOfMessge)
                                break;
                        }

                        await wsCloseAsync((int)WebSocketCloseStatus.NormalClosure, "Closing", CancellationToken.None);
                    });

                    return 200;
                }
                else
                {
                    return 404;
                }
            };
        }

        private static T GetValue<T>(IDictionary<string, object> env, string key)
        {
            object value;
            return env.TryGetValue(key, out value) && value is T ? (T)value : default(T);
        }

        private static string GetHeader(IDictionary<string, string[]> headers, string key)
        {
            string[] value;
            return headers.TryGetValue(key, out value) && value != null ? string.Join(",", value.ToArray()) : null;
        }

    }
}
```
# WebSockets — Middle

## Вопросы

- [Как реализовать heartbeat и reconnect логику?](#как-реализовать-heartbeat-и-reconnect-логику)
- [Как использовать Socket.io?](#как-использовать-socketio)
- [Как интегрировать WebSocket с React?](#как-интегрировать-websocket-с-react)
- [Как обрабатывать бинарные данные в WebSocket?](#как-обрабатывать-бинарные-данные-в-websocket)
- [Как реализовать стриминг ответов AI (SSE)?](#как-реализовать-стриминг-ответов-ai-sse)
- [Как тестировать WebSocket соединения?](#как-тестировать-websocket-соединения)
- [Что такое STOMP поверх WebSocket?](#что-такое-stomp-поверх-websocket)

---

## Как реализовать heartbeat и reconnect логику?

Heartbeat: клиент периодически отправляет ping, сервер отвечает pong. Если pong не пришёл — reconnect. Reconnect через exponential backoff.

```typescript
class WsClient {
  private ws: WebSocket | null = null;
  private reconnectDelay = 1000;

  connect(url: string) {
    this.ws = new WebSocket(url);
    this.ws.onclose = () => {
      setTimeout(() => {
        this.reconnectDelay = Math.min(this.reconnectDelay * 2, 30000);
        this.connect(url);
      }, this.reconnectDelay);
    };
    this.ws.onopen = () => { this.reconnectDelay = 1000; };
  }
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как использовать Socket.io?

Socket.io — библиотека поверх WebSocket с: автоматическим reconnect, комнатами (rooms), namespace, fallback на long-polling, событийная модель.

```typescript
import { io } from "socket.io-client";
const socket = io("https://api.example.com", { auth: { token: getToken() } });
socket.emit("join-room", { roomId: "chat-1" });
socket.on("message", (data) => addMessage(data));
socket.on("disconnect", () => console.log("Disconnected"));
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как интегрировать WebSocket с React?

Через кастомный хук: создавать соединение в useEffect, очищать при размонтировании. useRef для хранения WebSocket объекта.

```typescript
function useWebSocket(url: string) {
  const [messages, setMessages] = useState<string[]>([]);
  const wsRef = useRef<WebSocket | null>(null);

  useEffect(() => {
    wsRef.current = new WebSocket(url);
    wsRef.current.onmessage = (e) => setMessages(m => [...m, e.data]);
    return () => wsRef.current?.close();
  }, [url]);

  const send = useCallback((msg: string) => wsRef.current?.send(msg), []);
  return { messages, send };
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как обрабатывать бинарные данные в WebSocket?

Установить `ws.binaryType = "arraybuffer"` или `"blob"`. ArrayBuffer для обработки бинарных протоколов (MessagePack, protobuf). Blob для файлов.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как реализовать стриминг ответов AI (SSE)?

AI API (OpenAI, Anthropic) стримит ответы через SSE. `ReadableStream` + `TextDecoder` для обработки chunked ответов в fetch.

```typescript
const response = await fetch("/api/chat", { method: "POST", body: JSON.stringify(prompt) });
const reader = response.body!.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  const chunk = decoder.decode(value);
  // парсим SSE формат: "data: {...}\n\n"
  setContent(prev => prev + parseSSEChunk(chunk));
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как тестировать WebSocket соединения?

Мокировать WS в unit тестах через `jest-websocket-mock`. E2E тесты: Playwright поддерживает `page.routeWebSocket()` для перехвата WS.

```typescript
import WS from "jest-websocket-mock";
const server = new WS("ws://localhost:1234");
render(<ChatComponent />);
await server.connected;
server.send(JSON.stringify({ type: "message", text: "Hi" }));
expect(screen.getByText("Hi")).toBeInTheDocument();
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое STOMP поверх WebSocket?

**STOMP (Simple Text Oriented Messaging Protocol)** — протокол обмена сообщениями поверх WebSocket. Добавляет семантику: топики, подписки, подтверждения, заголовки — то чего нет в чистом WebSocket.

**Зачем STOMP:**
- Чистый WebSocket — просто канал передачи байт, нет структуры сообщений
- STOMP добавляет: `CONNECT`, `SUBSCRIBE`, `SEND`, `ACK` фреймы
- Используется в Java Spring Boot (Spring WebSocket + STOMP)

**Структура фрейма STOMP:**
```
SUBSCRIBE
destination:/topic/chat
id:sub-1

^@ (нулевой байт — конец фрейма)
```

**Интеграция в React (библиотека @stomp/stompjs):**
```typescript
import { Client } from '@stomp/stompjs';

const client = new Client({
  brokerURL: 'ws://localhost:8080/ws',
  onConnect: () => {
    // Подписка на топик:
    client.subscribe('/topic/messages', (message) => {
      const data = JSON.parse(message.body);
      setMessages(prev => [...prev, data]);
    });
  },
  reconnectDelay: 5000, // авто-переподключение
});

client.activate();

// Отправка:
client.publish({ destination: '/app/chat', body: JSON.stringify({ text: 'Hello' }) });

// Очистка:
return () => client.deactivate();
```

**STOMP vs чистый WebSocket:**
| | WebSocket | STOMP |
|---|---|---|
| Протокол | Низкоуровневый | Высокоуровневый |
| Топики/routing | Вручную | Встроен |
| ACK | Нет | Есть |
| Совместимость | Любой бэкенд | Java/Spring чаще |

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

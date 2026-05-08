# WebSockets — Expert

## Вопросы

- [Как реализовать кастомный бинарный протокол поверх WebSocket?](#как-реализовать-кастомный-бинарный-протокол-поверх-websocket)
- [Как строить real-time коллаборативный редактор?](#как-строить-real-time-коллаборативный-редактор)
- [Как обеспечить производительность при 10k+ одновременных соединений?](#как-обеспечить-производительность-при-10k-одновременных-соединений)
- [Как интегрировать WebRTC сигналинг через WebSocket?](#как-интегрировать-webrtc-сигналинг-через-websocket)

---

## Как реализовать кастомный бинарный протокол поверх WebSocket?

Используй ArrayBuffer + DataView или Protobuf. Определи wire format: magic bytes + message type + length + payload. Encode/decode через TypedArray.

```typescript
// Кодирование: type (1 byte) + id (4 bytes) + payload
function encodeMessage(type: number, id: number, payload: Uint8Array): ArrayBuffer {
  const buf = new ArrayBuffer(5 + payload.length);
  const view = new DataView(buf);
  view.setUint8(0, type);
  view.setUint32(1, id, true);
  new Uint8Array(buf, 5).set(payload);
  return buf;
}
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как строить real-time коллаборативный редактор?

CRDT (Yjs/Automerge) — структуры данных с детерминированным слиянием конфликтов. Каждая операция коммутативна и идемпотентна. Yjs: `Y.Text`, `Y.Map`, `Y.Array` + WebSocket provider для синхронизации.

Операционная трансформация (OT) — альтернатива (используется в Google Docs): сложнее, но детерминированный порядок. CRDT проще масштабировать — нет центрального сервера трансформаций.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как обеспечить производительность при 10k+ одновременных соединений?

Node.js: `uWebSockets.js` — 10-15x быстрее `ws` библиотеки. Elixir Phoenix Channels — миллионы соединений на один инстанс. Разделять: WebSocket gateway (много соединений) + бизнес-логика сервис. Memory per connection: буферы, event listeners — минимизировать. Горизонтальное масштабирование через Redis Pub/Sub.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как интегрировать WebRTC сигналинг через WebSocket?

WebRTC требует сигнального канала для обмена SDP offer/answer и ICE кандидатами — WebSocket идеален.

```typescript
// Инициатор
const pc = new RTCPeerConnection({ iceServers: [{ urls: "stun:stun.google.com:19302" }] });
const offer = await pc.createOffer();
await pc.setLocalDescription(offer);
ws.send(JSON.stringify({ type: "offer", sdp: offer.sdp, to: targetId }));

// Получатель
ws.onmessage = async (e) => {
  const msg = JSON.parse(e.data);
  if (msg.type === "offer") {
    await pc.setRemoteDescription(msg);
    const answer = await pc.createAnswer();
    await pc.setLocalDescription(answer);
    ws.send(JSON.stringify({ type: "answer", sdp: answer.sdp }));
  }
};
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

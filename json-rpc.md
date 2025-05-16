# JSON‑RPC 2.0 치트시트

> **JSON‑RPC**는 *JSON(JavaScript Object Notation) 문자열*만으로 원격 프로시저를 호출하는, 매우 가벼운 **RPC(Remote Procedure Call)** 프로토콜입니다. 전송 계층(HTTP·WebSocket·TCP·STDIO 등)에 구애받지 않고, 상태를 남기지 않는(stateless) 구조여서 MCP 같은 메시지 버스에서 널리 쓰입니다.

---

## 1. 핵심 특성

| 특성                  | 설명                                                                           |
| --------------------- | ------------------------------------------------------------------------------ |
| **형식(JSON)**        | 순수 JSON 텍스트. 추가 헤더나 래퍼가 없다.                                     |
| **무상태(stateless)** | 요청마다 `id·method·params` 정보를 모두 담아 서버가 세션을 기억할 필요가 없다. |
| **전송 독립**         | HTTP·WebSocket·STDIO 등 문자만 보내면 OK.                                      |
| **양방향 지원**       | 응답이 없는 _Notification_ / 여러 호출을 한 번에 묶는 _Batch_ 가능.            |
| **초소형 스펙**       | 2.0 전문이 A4 두어 장. 구현이 쉽고 가독성이 좋다.                              |

---

## 2. 메시지 객체 구조

### 2.1 요청(Request) 객체

| 필드      | 타입                       | 필수 | 의미                                             |
| --------- | -------------------------- | ---- | ------------------------------------------------ |
| `jsonrpc` | `"2.0"`                    | ✅   | 프로토콜 버전(고정)                              |
| `method`  | `string`                   | ✅   | 호출할 원격 함수 이름                            |
| `params`  | `array` 또는 `object`      | ❌   | 위치(positional)·이름(named) 인자                |
| `id`      | `string \| number \| null` | ❌\* | 응답 매칭용 식별자. 없으면 **Notification** 취급 |

> **Notification** = `id` 를 생략한 요청. 서버는 **반드시** 응답을 보내지 않는다.

#### 예시

```json
{ "jsonrpc": "2.0", "id": 1, "method": "subtract", "params": [42, 23] }
```

### 2.2 응답(Response) 객체

| 필드      | 타입                       | 필수 | 비고                           |
| --------- | -------------------------- | ---- | ------------------------------ |
| `jsonrpc` | `"2.0"`                    | ✅   | 요청과 동일 버전               |
| `result`  | _임의_                     | ✅\* | 성공 시에만 포함               |
| `error`   | **Error Object**           | ✅\* | 실패 시에만 포함               |
| `id`      | `string \| number \| null` | ✅   | 요청 `id` 와 동일(또는 `null`) |

> `result` **와** `error` 는 **동시에 존재할 수 없다**.

#### 성공 예시

```json
{ "jsonrpc": "2.0", "id": 1, "result": 19 }
```

#### 실패 예시

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32602,
    "message": "Invalid params"
  }
}
```

### 2.3 Error Object

| 필드      | 타입     | 필수 | 설명                             |
| --------- | -------- | ---- | -------------------------------- |
| `code`    | `number` | ✅   | 오류 코드(−32768 \~ −32000 예약) |
| `message` | `string` | ✅   | 짧은 오류 메시지                 |
| `data`    | _임의_   | ❌   | 추가 정보(스택 트레이스 등)      |

---

## 3. Batch 호출

여러 **Request** 객체를 배열로 묶어 한 번에 전송하고, 서버는 비‑Notification 요청에 대해 **Response** 배열을 돌려준다.

```jsonc
[
  { "jsonrpc": "2.0", "id": 1, "method": "add", "params": [1, 2] },
  { "jsonrpc": "2.0", "method": "ping" } // notification
]
```

---

## 4. MCP에서의 의의

1. **언어 독립성** → MCP Server가 TypeScript·Python·Rust 등 어떤 언어로 작성돼도 동일 규격.
2. **전송 유연성** → 로컬은 STDIO, 운영은 HTTP+SSE로 손쉽게 전환 가능.
3. **함수 중심 모델** → `method` = MCP **Tool** 이름, `params` = Tool 입력.

덕분에 MCP의 “소개 → 선택 → 실행 → 답변” 흐름을 최소 오버헤드로 주고받을 수 있다.

---

### 빠른 참고용

```text
Request  = {jsonrpc:"2.0", method, /*params?*/, /*id?*/}
Response = {jsonrpc:"2.0", id, result | error}
Error    = {code:int, message:str, /*data?*/}
```

> **암기 팁**: *버전·메서드·변수(인자)·(있으면) ID* → *버전·ID·값(성공) 또는 위반(오류)*.

---

_마지막 업데이트 : 2025‑05‑16_

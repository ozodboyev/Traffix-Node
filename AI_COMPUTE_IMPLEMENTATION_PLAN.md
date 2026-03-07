# Traffix Node — AI Compute Module v2 (Yangilangan Reja)

Minglab Android telefonlarni bitta markazlashmagan superkompyuterga aylantiruvchi modul. Proksi tarmog'iga ta'sir qilmaydi — parallel va izolyatsiya qilingan holda ishlaydi.

> [!CAUTION]
> **Honeypot (Tuzoq) Anti-Cheat** — 3-node konsensus O'RNIGA. Bitta paketga 1 ta oldindan javobi ma'lum "tuzoq" vazifa qo'shiladi. Serverga faqat o'sha 1 ta natijani tekshirish yetarli.

---

## Umumiy Arxitektura

```mermaid
graph TB
    subgraph "AI Client (Mijoz)"
        C[AI Task<br/>100K rasm]
    end

    subgraph "Backend (Go + RabbitMQ)"
        API[AI API]
        SPL["Splitter<br/>100K → 10K paket<br/>(9 real + 1 honeypot)"]
        Q[("RabbitMQ<br/>ai.subtasks.queue")]
        RCV[Result Receiver]
        HP["Honeypot Checker<br/>|X_tel - X_srv| < ε"]
        AGG[Aggregator]

        API --> SPL
        SPL --> Q
        RCV --> HP
        HP -->|✅ Trap to'g'ri| AGG
        HP -->|❌ Trap xato| BL[Blacklist + Retry]
    end

    subgraph "Android Nodes (TFLite)"
        N1["📱 Node 1<br/>Charging ✅ WiFi ✅"]
        N2["📱 Node 2<br/>Charging ✅ WiFi ✅"]
    end

    subgraph "Database"
        DB[("PostgreSQL<br/>ai_tasks<br/>ai_subtasks<br/>node_reputation")]
    end

    Q -->|WebSocket push| N1
    Q -->|WebSocket push| N2
    N1 -->|10 ta natija| RCV
    N2 -->|10 ta natija| RCV
    AGG --> DB
```

---

## Honeypot (Tuzoq) Mexanizmi — Batafsil

```
┌─────────────────────────────────────────────────────────┐
│ 1-QADAM: Paket tayyorlash (Go Backend)                  │
│   • Mijozdan 9 ta haqiqiy vazifa                        │
│   • Server o'z DB'dan 1 ta tuzoq qo'shadi               │
│   • Jami: 10 ta — aralashtirilgan, farq qilib bo'lmaydi │
├─────────────────────────────────────────────────────────┤
│ 2-QADAM: Hisoblash (Android TFLite)                     │
│   • Telefon 10 ta vazifani bajaradi                     │
│   • Hacker qaysi biri tuzoq ekanini bilmaydi           │
│   • Hammasini halol ishlashga majbur                    │
├─────────────────────────────────────────────────────────┤
│ 3-QADAM: Tekshirish (Go Backend)                        │
│   • Faqat tuzoq ID topib, natijani solishtiriladi       │
│   • Epsilon tolerance: |X_tel - X_srv| < 0.001         │
│   • Float32 protsessor farqlari hisobga olinadi         │
├─────────────────────────────────────────────────────────┤
│ 4-QADAM: Qaror                                          │
│   ✅ Tuzoq to'g'ri → 9 ta qabul + token to'lash        │
│   ❌ Tuzoq xato → 9 ta ham rad + blacklist + qayta      │
└─────────────────────────────────────────────────────────┘
```

---

## Proposed Changes

### 1. PostgreSQL Migration

#### [NEW] [026_ai_compute_tables.sql](file:///home/ozod/Downloads/opt/Traffix_Node/backend/migrations/026_ai_compute_tables.sql)

3 ta yangi jadval + honeypot reference jadval:

**`ai_tasks`** — Mijozdan kelgan katta vazifalar
**`ai_subtasks`** — 10 tadan iborat paketlar (9 real + 1 honeypot)
**`ai_honeypots`** — Oldindan hisoblab qo'yilgan tuzoq vazifalari
**`node_reputation`** — Node ishonchlilik reytingi

> [!NOTE]
> `ai_subtasks` da yangi ustun: `is_honeypot BOOLEAN DEFAULT false` va `honeypot_expected_hash VARCHAR(64)` — server faqat shu ustunli qatorni tekshiradi.

---

### 2. Backend — Go Models

#### [NEW] [ai_models.go](file:///home/ozod/Downloads/opt/Traffix_Node/backend/internal/models/ai_models.go)

Struct'lar: `AITask`, `AISubtask`, `AIHoneypot`, `NodeReputation`, `AIBatchPayload`

---

### 3. Backend — Repository

#### [NEW] [ai_repository.go](file:///home/ozod/Downloads/opt/Traffix_Node/backend/internal/repository/ai_repository.go)

CRUD + maxsus so'rovlar:
- `GetRandomHoneypot(taskType)` — tasodifiy tuzoq olish
- `GetHoneypotFromBatch(batchID)` — paketdagi tuzoqni aniqlash
- `UpdateReputationOnCheat(userID)` — cheat aniqlanganda reyting tushirish
- `GetBlacklistedNodes()` — qora ro'yxat

---

### 4. Backend — AI Task Service (Asosiy Logika)

#### [NEW] [ai_task_service.go](file:///home/ozod/Downloads/opt/Traffix_Node/backend/internal/services/ai_task_service.go)

**Asosiy funksiyalar:**

| Funksiya | Vazifasi |
|:---|:---|
| `SubmitTask()` | Yangi katta task qabul qilish |
| `SplitIntoBatches()` | 9 real + 1 honeypot = 10 tali paketlar yaratish |
| `InjectHoneypot()` | Tasodifiy joyga tuzoq qo'yish |
| `PublishToRabbitMQ()` | Paketni navbatga yuborish |
| `ReceiveBatchResult()` | 10 ta natijani qabul qilish |
| `ValidateHoneypot()` | `\|X_tel - X_srv\| < ε` tekshiruv |
| `AcceptBatch()` | ✅ 9 ta qabul + token to'lash |
| `RejectBatch()` | ❌ Hammasini rad + blacklist |
| `RetryBatch()` | Rad qilingan paketni boshqa node'ga yuborish |

**Epsilon tekshiruv (Go pseudocode):**
```go
const HoneypotEpsilon = 0.001

func (s *AITaskService) ValidateHoneypot(expected, actual []float32) bool {
    if len(expected) != len(actual) { return false }
    for i := range expected {
        if math.Abs(float64(expected[i]-actual[i])) > HoneypotEpsilon {
            return false
        }
    }
    return true
}
```

---

### 5. Backend — RabbitMQ Integration

#### [NEW] [rabbitmq.go](file:///home/ozod/Downloads/opt/Traffix_Node/backend/internal/core/rabbitmq.go)

RabbitMQ ulanish va queue boshqaruv. Ikki queue:
- `ai.subtasks.pending` — node'larga yuboriladigan paketlar
- `ai.results.completed` — node'lardan kelgan natijalar

#### [MODIFY] [config.yaml](file:///home/ozod/Downloads/opt/Traffix_Node/backend/config.yaml)

```diff
+rabbitmq:
+  url: "amqp://guest:guest@localhost:5672/"
+  subtask_queue: "ai.subtasks.pending"
+  result_queue: "ai.results.completed"
+  prefetch_count: 10

 modules:
   enabled:
     ...
     - traffic_billing
+    - ai_compute

+ai_compute:
+  enabled: true
+  dispatch_interval: 5
+  batch_size: 10
+  honeypot_count: 1
+  honeypot_epsilon: 0.001
+  subtask_timeout: 300
+  reward_per_subtask: 0.001
+  reputation_penalty: 25
+  reputation_reward: 1
+  min_reputation: 50
+  blacklist_threshold: 3
```

---

### 6. Backend — Module

#### [NEW] [ai_compute_module.go](file:///home/ozod/Downloads/opt/Traffix_Node/backend/internal/modules/ai_compute/ai_compute_module.go)

`core.Module` interface — `Init()` da RabbitMQ consumer va dispatcher ishga tushadi.

---

### 7. Backend — API Endpoints

#### [MODIFY] API module — yangi route'lar

| Method | Endpoint | Tavsif |
|:---|:---|:---|
| `POST` | `/api/v1/ai/tasks` | Yangi AI task |
| `GET` | `/api/v1/ai/tasks/:id` | Task status |
| `POST` | `/api/v1/ai/batch/result` | Node 10 ta natija yuboradi |
| `GET` | `/api/v1/ai/node/next-batch` | Keyingi paketni olish |
| `GET` | `/api/v1/ai/node/eligibility` | Node AI-ga tayyor mi? |
| `GET` | `/api/v1/admin/ai/dashboard` | Dashboard statistika |
| `GET` | `/api/v1/admin/ai/reputation` | Node reytinglari |

---

### 8. Backend — AppContext Update

#### [MODIFY] [context.go](file:///home/ozod/Downloads/opt/Traffix_Node/backend/internal/core/context.go)

```diff
+type AIComputeInterface interface {
+    SubmitTask(ctx context.Context, task interface{}) (string, error)
+    GetNextBatch(nodeUserID, nodeDeviceID int) (interface{}, error)
+    ReceiveBatchResult(batchID string, results []byte) error
+}

 type AppContext struct {
     ...
     SessionMgr    SessionManagerInterface
+    AICompute     AIComputeInterface
+    RabbitMQ      *RabbitMQClient
 }
```

---

### 9. Android (Kotlin) — To'liq Ishlaydigan Kod

#### [NEW] `android/app/src/main/java/com/traffix/node/ai/` katalogi

| Fayl | Vazifasi |
|:---|:---|
| `AIEligibilityChecker.kt` | Battery + WiFi + Thermal trigger'lar |
| `AIWorkerService.kt` | Foreground Service — AI hisoblash boshqaruvchisi |
| `TFLiteRunner.kt` | Model yuklash + inference + natija hash |
| `AITaskProtocol.kt` | Server bilan JSON aloqa (batch olish/yuborish) |

**Life-cycle diagramma:**

```mermaid
stateDiagram-v2
    [*] --> IDLE
    IDLE --> CHECKING: Timer (30 sek)
    CHECKING --> IDLE: Shartlar bajarilmadi
    CHECKING --> READY: ✅ Charging + WiFi + ThermalOK
    READY --> DOWNLOADING: Server batch yubordi (10 ta)
    DOWNLOADING --> COMPUTING: Model + data tayyor
    COMPUTING --> UPLOADING: 10 ta natija (hash bilan)
    UPLOADING --> READY: ✅ Serverga yuborildi
    COMPUTING --> PAUSED: 🌡️ Thermal MODERATE+
    PAUSED --> COMPUTING: Sovudi
    PAUSED --> IDLE: ❌ Zaryadka uzildi
```

**Qat'iy triggerlar (AIEligibilityChecker):**
- `BatteryManager.isCharging() == true`
- `ConnectivityManager` → `NET_CAPABILITY_NOT_METERED` (Wi-Fi)
- `PowerManager.currentThermalStatus <= THERMAL_STATUS_LIGHT` (Android 10+)

#### [MODIFY] `android/app/build.gradle.kts`

```diff
+implementation("org.tensorflow:tensorflow-lite:2.16.1")
+implementation("org.tensorflow:tensorflow-lite-support:0.4.4")
+implementation("com.rabbitmq:amqp-client:5.20.0")
```

---

### 10. Admin Dashboard — AI Computing Tab

#### [MODIFY] [index.html](file:///home/ozod/Downloads/opt/Traffix_Node/backend/admin/index.html)

Sidebar'ga yangi nav item:
```html
<a href="#" class="nav-item" data-page="ai-compute">
    <span class="nav-icon">🧠</span>
    <span>AI Compute</span>
</a>
```

Yangi sahifa — **yuqorida tab toggle**:

```
┌──────────────────────────────────────────────┐
│ [🌐 Proxy Share]  [🧠 AI Computing]  ← toggle│
├──────────────────────────────────────────────┤
│                                              │
│  Stats Grid:                                 │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐│
│  │🟢 Ready│ │📊Active│ │✅ Done │ │⚠️Cheat ││
│  │  Nodes │ │ Tasks  │ │Batches │ │Detected││
│  │ 50,000 │ │   12   │ │ 8,450  │ │    3   ││
│  └────────┘ └────────┘ └────────┘ └────────┘│
│                                              │
│  Tasks Table:                                │
│  ┌──────────────────────────────────────────┐│
│  │ ID │ Type │ Items │ Progress │ Status    ││
│  │ 1  │ infer│ 100K  │ ████ 84% │ running  ││
│  │ 2  │ embed│ 50K   │ ██████100│ completed││
│  └──────────────────────────────────────────┘│
│                                              │
│  Node Reputation:                            │
│  ┌──────────────────────────────────────────┐│
│  │ User │ Device │ Score │ Tasks │ Status   ││
│  │ #42  │ S24    │ 98.5  │ 1,200 │ ✅ OK   ││
│  │ #89  │ Mi13   │ 12.0  │    50 │ 🚫 BAN  ││
│  └──────────────────────────────────────────┘│
│                                              │
│  🌡️ Thermal Map (hudud bo'yicha qizish)      │
└──────────────────────────────────────────────┘
```

#### [MODIFY] [app.js](file:///home/ozod/Downloads/opt/Traffix_Node/backend/admin/app.js)

AI Dashboard JS funksiyalari: `loadAIDashboard()`, `loadAITasks()`, `loadNodeReputation()`

#### [MODIFY] [styles.css](file:///home/ozod/Downloads/opt/Traffix_Node/backend/admin/styles.css)

Tab toggle va AI-ga xos CSS stillari

---

## Verification Plan

### Automated Tests
```bash
# 1. SQL migration
psql -U traffix -d traffix_node -f migrations/026_ai_compute_tables.sql

# 2. Go build
cd /home/ozod/Downloads/opt/Traffix_Node/backend && go build ./...

# 3. RabbitMQ connection
rabbitmqctl list_queues | grep ai
```

### Manual Verification
- Admin panelda "AI Compute" sahifasi ochilib, tab toggle ishlashini ko'rish
- Backend logda `ai_compute module initialized` chiqishini tekshirish
- Honeypot epsilon tekshiruv logikasini test data bilan sinash

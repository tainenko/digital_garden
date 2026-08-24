---
title: Low Level Design / OOD 物件導向設計題型
type: concept
tags: [interview, lld, ood, design-patterns, object-oriented]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Low Level Design / OOD 物件導向設計題型

## LLD vs HLD 的差異

| 面向 | HLD（系統設計） | LLD（物件導向設計） |
|------|---------------|-----------------|
| 範圍 | 多服務、分散式 | 單一系統的類別設計 |
| 考察 | 可擴展性、CAP、快取 | SOLID、設計模式、封裝 |
| 輸出 | 架構圖 + 技術選型 | 類別圖 + 核心代碼 |
| 時間 | 45–60 分鐘 | 30–45 分鐘 |

---

## LLD 面試五步驟

1. **釐清需求**（3 分鐘）：核心實體、主要操作、邊界條件
2. **識別實體與關係**（5 分鐘）：名詞 → Class，動詞 → Method
3. **定義介面與契約**（5 分鐘）：先定 interface，再實作
4. **實作核心邏輯**（20 分鐘）：設計模式 + 核心算法
5. **討論擴展性**（5 分鐘）：如何加功能、如何測試

---

## 高頻題目與解法

### 1. 停車場系統（Parking Lot）

**實體識別**：ParkingLot、Floor、Spot、Vehicle、Ticket、Payment

```python
from enum import Enum
from datetime import datetime
from dataclasses import dataclass, field
from abc import ABC, abstractmethod

class VehicleType(Enum):
    MOTORCYCLE = "motorcycle"
    CAR = "car"
    BUS = "bus"

class SpotType(Enum):
    SMALL = "small"
    MEDIUM = "medium"
    LARGE = "large"

@dataclass
class Vehicle:
    plate: str
    vehicle_type: VehicleType

@dataclass
class ParkingSpot:
    spot_id: str
    spot_type: SpotType
    floor: int
    is_occupied: bool = False
    vehicle: Vehicle | None = None

    def can_fit(self, vehicle: Vehicle) -> bool:
        type_map = {
            VehicleType.MOTORCYCLE: SpotType.SMALL,
            VehicleType.CAR: SpotType.MEDIUM,
            VehicleType.BUS: SpotType.LARGE,
        }
        return self.spot_type == type_map[vehicle.vehicle_type] and not self.is_occupied

    def park(self, vehicle: Vehicle):
        self.vehicle = vehicle
        self.is_occupied = True

    def free(self):
        self.vehicle = None
        self.is_occupied = False

@dataclass
class Ticket:
    ticket_id: str
    vehicle: Vehicle
    spot: ParkingSpot
    entry_time: datetime = field(default_factory=datetime.now)

class PricingStrategy(ABC):
    @abstractmethod
    def calculate(self, vehicle_type: VehicleType, hours: float) -> float: ...

class HourlyPricing(PricingStrategy):
    RATES = {VehicleType.MOTORCYCLE: 30, VehicleType.CAR: 60, VehicleType.BUS: 120}

    def calculate(self, vehicle_type: VehicleType, hours: float) -> float:
        return self.RATES[vehicle_type] * max(1, hours)  # 最少計 1 小時

class ParkingLot:
    def __init__(self, pricing: PricingStrategy):
        self._spots: list[ParkingSpot] = []
        self._active_tickets: dict[str, Ticket] = {}  # plate → ticket
        self._pricing = pricing

    def add_spot(self, spot: ParkingSpot):
        self._spots.append(spot)

    def park(self, vehicle: Vehicle) -> Ticket | None:
        spot = next((s for s in self._spots if s.can_fit(vehicle)), None)
        if not spot:
            return None
        spot.park(vehicle)
        ticket = Ticket(ticket_id=f"T{len(self._active_tickets)+1}", vehicle=vehicle, spot=spot)
        self._active_tickets[vehicle.plate] = ticket
        return ticket

    def exit(self, plate: str) -> float:
        ticket = self._active_tickets.pop(plate, None)
        if not ticket:
            raise ValueError(f"No active ticket for {plate}")
        ticket.spot.free()
        hours = (datetime.now() - ticket.entry_time).seconds / 3600
        return self._pricing.calculate(ticket.vehicle.vehicle_type, hours)
```

**設計亮點**：
- `PricingStrategy` 介面 → Strategy Pattern，不同時段/節假日可替換策略
- `can_fit()` 封裝停車位相容性判斷
- `_active_tickets` 用 plate 做 key，O(1) 查找

---

### 2. 電梯系統（Elevator System）

**實體**：Building、Elevator、ElevatorController、Request

```python
from enum import Enum
from dataclasses import dataclass, field
import heapq

class Direction(Enum):
    UP = "up"
    DOWN = "down"
    IDLE = "idle"

@dataclass
class ElevatorRequest:
    floor: int
    direction: Direction  # 從哪層、往哪個方向

class Elevator:
    def __init__(self, elevator_id: int, total_floors: int):
        self.id = elevator_id
        self.current_floor = 1
        self.direction = Direction.IDLE
        self._up_queue: list[int] = []    # min-heap：往上的目標層
        self._down_queue: list[int] = []  # max-heap（存負數）：往下的目標層

    def add_floor(self, floor: int):
        if floor > self.current_floor:
            heapq.heappush(self._up_queue, floor)
        else:
            heapq.heappush(self._down_queue, -floor)

    def step(self):
        """模擬電梯移動一步"""
        if self.direction == Direction.UP and self._up_queue:
            target = heapq.heappop(self._up_queue)
            self.current_floor = target
            if not self._up_queue:
                self.direction = Direction.DOWN if self._down_queue else Direction.IDLE
        elif self.direction == Direction.DOWN and self._down_queue:
            target = -heapq.heappop(self._down_queue)
            self.current_floor = target
            if not self._down_queue:
                self.direction = Direction.UP if self._up_queue else Direction.IDLE
        elif self._up_queue:
            self.direction = Direction.UP
        elif self._down_queue:
            self.direction = Direction.DOWN

    @property
    def load(self) -> int:
        return len(self._up_queue) + len(self._down_queue)

class ElevatorController:
    def __init__(self, elevators: list[Elevator]):
        self.elevators = elevators

    def dispatch(self, request: ElevatorRequest):
        # SCAN 演算法：選最近、同方向的電梯
        best = min(
            self.elevators,
            key=lambda e: (abs(e.current_floor - request.floor), e.load),
        )
        best.add_floor(request.floor)
        return best
```

**討論點**：SCAN（電梯）算法 vs FCFS vs SSTF；多部電梯調度最優解是 NP-hard，實際用啟發式。

---

### 3. 圖書館管理系統（Library）

```python
from datetime import datetime, timedelta
from enum import Enum

class BookStatus(Enum):
    AVAILABLE = "available"
    BORROWED = "borrowed"
    RESERVED = "reserved"

class Book:
    def __init__(self, isbn: str, title: str, author: str, copies: int):
        self.isbn = isbn
        self.title = title
        self.author = author
        self._available = copies
        self._total = copies
        self._waitlist: list["Member"] = []

    def borrow(self, member: "Member") -> "BorrowRecord | None":
        if self._available > 0:
            self._available -= 1
            return BorrowRecord(book=self, member=member)
        self._waitlist.append(member)  # 加入等候清單
        return None

    def return_book(self, record: "BorrowRecord") -> "Member | None":
        self._available += 1
        if self._waitlist:
            next_member = self._waitlist.pop(0)
            return next_member  # 通知下一位
        return None

    @property
    def status(self) -> BookStatus:
        if self._available > 0:
            return BookStatus.AVAILABLE
        if self._waitlist:
            return BookStatus.RESERVED
        return BookStatus.BORROWED

class BorrowRecord:
    BORROW_DAYS = 14
    FINE_PER_DAY = 5

    def __init__(self, book: Book, member: "Member"):
        self.book = book
        self.member = member
        self.borrow_date = datetime.now()
        self.due_date = self.borrow_date + timedelta(days=self.BORROW_DAYS)
        self.return_date: datetime | None = None

    def calculate_fine(self) -> float:
        if not self.return_date or self.return_date <= self.due_date:
            return 0
        overdue_days = (self.return_date - self.due_date).days
        return overdue_days * self.FINE_PER_DAY
```

---

### 4. 聊天室（Chat System，單機版）

```python
from abc import ABC, abstractmethod
from datetime import datetime

class Message:
    def __init__(self, sender_id: str, content: str):
        self.sender_id = sender_id
        self.content = content
        self.timestamp = datetime.now()

class ChatRoom(ABC):
    def __init__(self, room_id: str):
        self.room_id = room_id
        self._members: set[str] = set()
        self._history: list[Message] = []

    def join(self, user_id: str):
        self._members.add(user_id)

    def leave(self, user_id: str):
        self._members.discard(user_id)

    @abstractmethod
    def send(self, message: Message): ...

class DirectMessage(ChatRoom):
    """一對一私訊"""
    def send(self, message: Message):
        if message.sender_id not in self._members:
            raise PermissionError("Sender not in room")
        self._history.append(message)
        # 通知接收方（Observer Pattern）
        for member_id in self._members:
            if member_id != message.sender_id:
                NotificationService.notify(member_id, message)

class GroupChat(ChatRoom):
    """群組聊天，支援管理員"""
    def __init__(self, room_id: str, admin_id: str):
        super().__init__(room_id)
        self._admin_id = admin_id
        self.join(admin_id)

    def kick(self, admin: str, target: str):
        if admin != self._admin_id:
            raise PermissionError("Only admin can kick")
        self.leave(target)

    def send(self, message: Message):
        self._history.append(message)
        for member_id in self._members:
            if member_id != message.sender_id:
                NotificationService.notify(member_id, message)
```

---

## 常用設計模式速查

| 題目 | 適用模式 | 解決什麼問題 |
|------|---------|------------|
| 停車場定價 | Strategy | 可替換計費規則 |
| 電梯通知 | Observer | 電梯到達通知乘客 |
| 圖書館 Book 狀態 | State | 狀態轉換邏輯集中 |
| 各類通知（Email/SMS） | Strategy / Factory | 解耦通知方式 |
| 建立複雜物件 | Builder | 避免多參數 Constructor |
| 全域唯一資源（DB連線） | Singleton | 確保唯一實例 |
| 同介面不同實作 | Factory Method | 解耦建立邏輯 |

---

## 面試常犯的錯誤

1. **跳過需求釐清**：直接寫 code，後來發現方向錯誤
2. **過度設計**：5 個介面套 10 個 pattern，問題只需 2 個 class
3. **忽略邊界條件**：停車場滿了？書已借出？使用者不存在？
4. **方法命名含糊**：`process()`, `handle()` → 應該 `calculate_fee()`, `dispatch_elevator()`
5. **跳過封裝**：把內部狀態設 public，失去物件邊界

---

## 相關頁面

- [[大廠技術面試的底層邏輯]] — 面試整體策略
- [[九分鐘看懂系統設計面試]] — HLD 對照
- [[Go介面設計模式]] — Go 的設計模式實作
- [[DDD領域驅動設計]] — Entity / Value Object 在 LLD 的應用

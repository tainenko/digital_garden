---
title: "SD題解：停車場系統（Parking Garage）"
type: topic
tags: [system-design, ood, object-oriented, golang, easy]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：停車場系統（Parking Garage）

> **難度**: 入門 ｜ **類型**: OOD（物件導向設計）｜ **頻率**: 中等

---

## 題目

設計一個停車場管理系統，支援：
- 車輛入場 / 出場
- 計算停車費
- 查詢可用車位
- 不同車型（機車/轎車/大型車）

---

## OOD 設計要點

面試時 OOD 題考的是：
1. 識別實體（Entity）與關係
2. 設計合理的介面與封裝
3. 使用設計模式解決可擴展性問題

---

## 核心實體設計

```go
package parking

import (
    "errors"
    "fmt"
    "sync"
    "time"
)

// VehicleType 車輛類型
type VehicleType int

const (
    Motorcycle VehicleType = iota
    Car
    Truck
)

// SpotSize 車位大小
type SpotSize int

const (
    Small  SpotSize = iota // 機車位
    Medium                 // 轎車位
    Large                  // 大型車位
)

// 車位大小 → 可停車輛類型對應
func canFit(spot SpotSize, vehicle VehicleType) bool {
    switch spot {
    case Small:
        return vehicle == Motorcycle
    case Medium:
        return vehicle == Motorcycle || vehicle == Car
    case Large:
        return true // 所有車型都可停
    }
    return false
}

// Vehicle 車輛
type Vehicle struct {
    LicensePlate string
    Type         VehicleType
}

// ParkingSpot 車位
type ParkingSpot struct {
    ID       string
    Size     SpotSize
    Floor    int
    Row      int
    Col      int
    isOccupied bool
    vehicle  *Vehicle
    mu       sync.Mutex
}

func (s *ParkingSpot) IsAvailable() bool {
    s.mu.Lock()
    defer s.mu.Unlock()
    return !s.isOccupied
}

func (s *ParkingSpot) Park(v *Vehicle) error {
    s.mu.Lock()
    defer s.mu.Unlock()
    if s.isOccupied {
        return errors.New("spot already occupied")
    }
    if !canFit(s.Size, v.Type) {
        return fmt.Errorf("vehicle type %d cannot fit in spot size %d", v.Type, s.Size)
    }
    s.vehicle = v
    s.isOccupied = true
    return nil
}

func (s *ParkingSpot) Vacate() *Vehicle {
    s.mu.Lock()
    defer s.mu.Unlock()
    v := s.vehicle
    s.vehicle = nil
    s.isOccupied = false
    return v
}
```

### 停車票據

```go
// Ticket 停車票據
type Ticket struct {
    ID         string
    Vehicle    *Vehicle
    Spot       *ParkingSpot
    EntryTime  time.Time
    ExitTime   *time.Time
    Fee        float64
}

func (t *Ticket) IsActive() bool {
    return t.ExitTime == nil
}
```

### 費率策略（Strategy Pattern）

```go
// FeeStrategy 費率介面（Strategy Pattern）
type FeeStrategy interface {
    Calculate(duration time.Duration, vehicleType VehicleType) float64
}

// HourlyFee 按小時收費
type HourlyFee struct {
    RatePerHour map[VehicleType]float64
}

func (f *HourlyFee) Calculate(duration time.Duration, vType VehicleType) float64 {
    hours := duration.Hours()
    if hours < 1 {
        hours = 1 // 最少收 1 小時
    }
    return hours * f.RatePerHour[vType]
}

// FlatFee 固定費率（如夜間停車）
type FlatFee struct {
    Rate float64
}

func (f *FlatFee) Calculate(duration time.Duration, _ VehicleType) float64 {
    return f.Rate
}

// TieredFee 階梯式收費（前2小時免費，之後每小時$5）
type TieredFee struct {
    FreeHours   int
    RatePerHour float64
}

func (f *TieredFee) Calculate(duration time.Duration, _ VehicleType) float64 {
    hours := int(duration.Hours())
    if hours <= f.FreeHours {
        return 0
    }
    return float64(hours-f.FreeHours) * f.RatePerHour
}
```

### 停車場主體

```go
// ParkingLot 停車場
type ParkingLot struct {
    ID      string
    Floors  []*Floor
    tickets map[string]*Ticket // ticketID → Ticket
    mu      sync.RWMutex
    fee     FeeStrategy
}

type Floor struct {
    Number int
    Spots  []*ParkingSpot
}

func NewParkingLot(id string, fee FeeStrategy) *ParkingLot {
    return &ParkingLot{
        ID:      id,
        tickets: make(map[string]*Ticket),
        fee:     fee,
    }
}

// ParkVehicle 車輛入場
func (p *ParkingLot) ParkVehicle(vehicle *Vehicle) (*Ticket, error) {
    spot := p.findAvailableSpot(vehicle.Type)
    if spot == nil {
        return nil, errors.New("no available spot for this vehicle type")
    }

    if err := spot.Park(vehicle); err != nil {
        return nil, err
    }

    ticket := &Ticket{
        ID:        generateTicketID(),
        Vehicle:   vehicle,
        Spot:      spot,
        EntryTime: time.Now(),
    }

    p.mu.Lock()
    p.tickets[ticket.ID] = ticket
    p.mu.Unlock()

    return ticket, nil
}

// ExitVehicle 車輛出場並計算費用
func (p *ParkingLot) ExitVehicle(ticketID string) (*Ticket, error) {
    p.mu.Lock()
    ticket, ok := p.tickets[ticketID]
    if !ok || !ticket.IsActive() {
        p.mu.Unlock()
        return nil, errors.New("invalid or already processed ticket")
    }
    now := time.Now()
    ticket.ExitTime = &now
    p.mu.Unlock()

    // 計算費用
    duration := ticket.ExitTime.Sub(ticket.EntryTime)
    ticket.Fee = p.fee.Calculate(duration, ticket.Vehicle.Type)

    // 釋放車位
    ticket.Spot.Vacate()

    return ticket, nil
}

// findAvailableSpot 找最近的可用車位（優先同類型）
func (p *ParkingLot) findAvailableSpot(vType VehicleType) *ParkingSpot {
    // 策略：優先找最小合適車位（機車優先找機車位，避免佔用轎車位）
    targetSize := vehicleToMinSpotSize(vType)

    for _, floor := range p.Floors {
        for _, spot := range floor.Spots {
            if spot.Size == targetSize && spot.IsAvailable() {
                return spot
            }
        }
    }
    // 找不到合適大小，找更大的
    for _, floor := range p.Floors {
        for _, spot := range floor.Spots {
            if spot.Size > targetSize && canFit(spot.Size, vType) && spot.IsAvailable() {
                return spot
            }
        }
    }
    return nil
}

// GetAvailability 查詢各車位類型剩餘數量
func (p *ParkingLot) GetAvailability() map[SpotSize]int {
    counts := map[SpotSize]int{Small: 0, Medium: 0, Large: 0}
    for _, floor := range p.Floors {
        for _, spot := range floor.Spots {
            if spot.IsAvailable() {
                counts[spot.Size]++
            }
        }
    }
    return counts
}

func vehicleToMinSpotSize(vType VehicleType) SpotSize {
    switch vType {
    case Motorcycle:
        return Small
    case Car:
        return Medium
    case Truck:
        return Large
    }
    return Medium
}
```

---

## 延伸：系統設計面

如果題目要求**大規模分散式**版本：

```
入口閘機 → API Server → 停車場服務
                          ↓         ↓
                       Redis     PostgreSQL
                    （即時車位）  （票據/歷史）
```

**車位即時狀態**：Redis 存 `spot:{lot_id}:{spot_id}` → `{available|occupied}`

**高並發入場**：入場操作用 Redis `SETNX`（原子操作）防止兩台車同時佔用同一個位：
```go
ok, _ := redis.SetNX(ctx, "spot:"+spotID, vehicleID, 24*time.Hour).Result()
if !ok {
    return errors.New("spot taken, try another")
}
```

---

## 相關題解

- [[sd-rate-limiter|SD題解：Rate Limiter]] — 防止入場請求過快
- [[sd-ticketing-system|SD題解：訂票系統]] — 類似的高並發佔位問題

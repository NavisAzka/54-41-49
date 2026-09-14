# Rangkuman Diskusi — Sistem Dispatch Autonomous Robotaxi

## 1. Arah Awal: Autonomous Vehicle / Robotaxi

Topik penelitian yang dibahas berada di antara beberapa area berikut:

- Autonomous vehicle
- Robotaxi
- Passenger application
- Fleet management
- MaaS
- MQTT
- C-V2X
- Network/communication architecture

Fokus akhirnya mengarah ke **sistem pendukung autonomous robotaxi**, bukan pengembangan autonomous driving stack seperti SLAM, perception, atau control.

## 2. Passenger Application vs Fleet Management

### Passenger Application

Digunakan oleh penumpang untuk:

- menentukan pickup
- menentukan destination
- request ride
- melihat posisi kendaraan
- melihat ETA
- cancel trip
- emergency

```
Passenger
    │
    ▼
Passenger App
    │
    ▼
Ride Request
```

### Fleet Management / Dispatch

Digunakan oleh operator/sistem untuk:

- menerima passenger request
- menentukan kendaraan
- menentukan pickup/drop-off point
- membuat mission
- monitoring kendaraan
- mengatur perjalanan

```
Passenger App
      │
      ▼
Fleet / Dispatch
      │
      ▼
Autonomous Vehicle
```

Karena hanya ada 1 autonomous car, istilah **Fleet Management** menjadi agak terlalu luas. Fokus kemudian mengarah ke **Dispatch and Trip Management**.

## 3. MaaS

Mobility-as-a-Service (MaaS) adalah platform yang mengintegrasikan layanan transportasi ke dalam satu sistem.

```
                MaaS
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
      Bus       MRT     Robotaxi
```

Passenger application dapat menjadi interface MaaS. Namun untuk skripsi ini, MaaS dianggap terlalu luas jika mencakup seluruh transportasi, payment, public transport, dll. Karena itu **MaaS lebih cocok sebagai konteks, bukan fokus utama**.

## 4. MQTT

MQTT yang digunakan pada autonomous vehicle pada dasarnya adalah MQTT yang sama dengan IoT. Perbedaannya berada pada konteks dan requirement.

```
IoT:
Sensor → MQTT → Broker → Cloud

Robotaxi:
Robotaxi → MQTT → Broker → Dispatch/Fleet Manager
```

MQTT cocok untuk:

- telemetry
- vehicle status
- mission/event
- command
- cloud communication

Sedangkan komunikasi internal autonomous vehicle seperti `Perception → Planning → Control` lebih cocok menggunakan **ROS 2 / DDS**.

## 5. C-V2X

C-V2X (Cellular Vehicle-to-Everything) adalah teknologi komunikasi kendaraan berbasis cellular. Dua konsep utamanya:

```
Vehicle ↔ Cellular Network ↔ Cloud            (V2N)

Vehicle ↔ Vehicle
Vehicle ↔ Infrastructure
Vehicle ↔ Pedestrian                          (PC5)
```

C-V2X dapat digunakan untuk komunikasi autonomous vehicle dengan cloud, fleet manager, kendaraan lain, dan infrastructure — namun **belum menjadi fokus skripsi**.

## 6. Dispatch

Dispatch adalah proses menentukan bagaimana passenger request dilayani oleh kendaraan.

```
Passenger
Pickup: A
Destination: B
       │
       ▼
Dispatch System
       │
       ▼
Autonomous Car
       │
       ▼
      A → B
```

Dengan banyak kendaraan, dispatch dapat berarti *vehicle assignment*. Dengan satu kendaraan, fokusnya berubah menjadi: apakah kendaraan tersedia, menerima request, membuat mission, dan menjalankan perjalanan.

## 7. Pickup & Drop-off Point

Posisi pickup/drop-off tidak harus persis sama dengan koordinat yang diberikan penumpang. Misalnya:

```
Passenger GPS → Mall → Safe Pickup Point
```

Dispatch system dapat menentukan titik yang feasible berdasarkan:

- road network
- posisi kendaraan
- accessible road
- stopping area
- geofence
- safety

Begitu juga destination:

```
Destination → Drop-off Point → Safe/legal stopping area
```

**Pembagian tanggung jawab:**

| Peran | Tanggung Jawab |
|---|---|
| **Dispatch** | WHAT & WHERE — menentukan pickup = A, drop-off = B, mission = A → B |
| **Autonomous Vehicle** | HOW — menjalankan navigasi/planning dari A ke B |

## 8. Rebalancing

Awalnya dibahas konsep **Dispatch + Rebalancing Fleet** — memindahkan kendaraan kosong ke lokasi yang diperkirakan memiliki demand.

```
Demand tinggi → Zone A
Robotaxi kosong → Robotaxi → Zone A
```

Karena hanya ada 1 autonomous car, **fleet rebalancing bukan fokus yang ideal**. Konsep yang masih bisa digunakan adalah **post-trip repositioning**:

```
Trip selesai → Vehicle berada di B → Apakah perlu reposition? → Zone A / Wait
```

Namun ini bukan kontribusi utama.

## 9. Judul yang Akhirnya Dipilih

> **Rancang Bangun Sistem Dispatch dan Manajemen Perjalanan pada Autonomous Robotaxi**

Judul ini lebih sesuai dengan kondisi yang ada karena:

- memiliki 1 autonomous car
- mobil sudah fully autonomous
- tidak perlu membuat autonomous driving system
- fokus pada software/system architecture
- bisa mengintegrasikan passenger request
- bisa menggunakan MQTT
- bisa terhubung langsung ke kendaraan nyata

## 10. Arsitektur Sistem yang Dibayangkan

```
                  Passenger
                     │
                     ▼
              Passenger App
                     │
                Ride Request
                     │
                     ▼
            ┌──────────────────┐
            │  Dispatch System │
            │                  │
            │  Request Manager │
            │  Trip Manager    │
            │  Vehicle Manager │
            │  Pickup Selector │
            │  Dropoff Selector│
            └────────┬─────────┘
                     │
                 MQTT / API
                     │
                     ▼
              Autonomous Car
                     │
              ┌──────┴──────┐
              │             │
           ROS 2       Navigation
              │             │
              └──────┬──────┘
                     │
                Vehicle State
                     │
                     ▼
               Dispatch System
```

> Lihat versi visual/interaktif pada [`arsitektur_sistem.html`](arsitektur_sistem.html).

## 11. State Machine Kendaraan

```
             ┌──────────┐
             │   IDLE   │
             └────┬─────┘
                  │  New Request
                  ▼
            ┌──────────┐
            │ ASSIGNED │
            └────┬─────┘
                  ▼
            ┌──────────┐
            │  PICKUP  │
            └────┬─────┘
                  ▼
            ┌──────────┐
            │ ON TRIP  │
            └────┬─────┘
                  ▼
            ┌──────────┐
            │ DROPOFF  │
            └────┬─────┘
                  ▼
             ┌───────┐
             │ IDLE  │
             └───────┘
```

Ini penting untuk trip management.

## 12. Paper yang Sudah Ditemukan

| Paper | Fokus | Relevansi |
|---|---|---|
| **Hyland & Mahmassani** — *Dynamic Autonomous Vehicle Fleet Operations: Optimization-Based Strategies to Assign AVs to Immediate Traveler Demand Requests* | passenger demand, AV assignment, dispatch, waiting time, vehicle utilization | Basis untuk dispatch |
| **Duan et al.** — *Centralized and Decentralized Autonomous Dispatching Strategy for Dynamic Autonomous Taxi Operation in Hybrid Request Mode* | autonomous taxi, dynamic request, dispatch, assignment, routing | Basis untuk autonomous taxi dispatch |
| **Real-time dispatch management (2024)** — *Real-time dispatch management of shared autonomous vehicles with on-demand and pre-booked requests* | real-time dispatch, on-demand request, pre-booked request, vehicle allocation | Relevan untuk request management |
| **Tavor & Raviv** — *Anticipatory Rebalancing of RoboTaxi Systems* | RoboTaxi, dispatch, rebalancing, passenger waiting time, empty vehicle distance | Relevan jika penelitian dikembangkan menjadi multi-vehicle |

> Daftar pustaka lengkap ada di [`Tinjauan Pustaka - Sistem Dispatch Autonomous Robotaxi.pdf`](Tinjauan%20Pustaka%20-%20Sistem%20Dispatch%20Autonomous%20Robotaxi.pdf).

## 13. Scope Skripsi yang Paling Masuk Akal

```
Input
  Passenger Request
  ├── Pickup
  └── Destination

Processing
  Dispatch System
  ├── Vehicle availability
  ├── Pickup point selection
  ├── Drop-off point selection
  ├── Mission generation
  └── Trip state management

Communication
  Dispatch ↔ Autonomous Car
            MQTT / API

Output
  Autonomous Car
  ├── Go to pickup
  ├── Pickup passenger
  ├── Navigate to destination
  └── Drop-off passenger
```

### Evaluasi

Sistem dapat diuji dengan:

- request response time
- communication latency
- mission execution success
- pickup accuracy
- trip completion
- MQTT QoS
- packet loss
- connection recovery

## Kesimpulan

Arah penelitian yang sudah terbentuk:

> **Passenger Request → Dispatch → Pickup/Drop-off Selection → Mission → Autonomous Vehicle → Trip Management**

Dengan 1 autonomous car sebagai real-world testbed. Sistem tidak perlu membuat "fleet" dalam arti banyak kendaraan — lebih tepat disebut **Dispatch and Trip Management System for Autonomous Robotaxi**, sementara konsep fleet management dan paper fleet-operation menjadi landasan teoritis.

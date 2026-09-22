# TÀI LIỆU THIẾT KẾ HỆ THỐNG
## WebDoanVien – Hệ Thống Quản Lý Đoàn Viên Thanh Niên CATP

---

| Thông tin | Nội dung |
|---|---|
| **Tên dự án** | youth-union-management |
| **Phiên bản tài liệu** | 1.0 |
| **Ngày soạn** | Tháng 9, 2026 |
| **Loại tài liệu** | System Design Document (SDD) |

---

## MỤC LỤC

1. [Giới thiệu & Bối cảnh](#1-giới-thiệu--bối-cảnh)
2. [Stakeholders & Actors](#2-stakeholders--actors)
3. [Use Case Diagram](#3-use-case-diagram)
4. [Mô tả Use Case chi tiết](#4-mô-tả-use-case-chi-tiết)
5. [User Stories & Acceptance Criteria](#5-user-stories--acceptance-criteria)
6. [Activity Diagrams – Luồng nghiệp vụ](#6-activity-diagrams--luồng-nghiệp-vụ)
7. [Sequence Diagrams](#7-sequence-diagrams)
8. [Entity Relationship Diagram (ERD)](#8-entity-relationship-diagram-erd)
9. [Kiến trúc hệ thống & Phân quyền](#9-kiến-trúc-hệ-thống--phân-quyền)
10. [Non-Functional Requirements](#10-non-functional-requirements)
11. [Bảng chú giải thuật ngữ](#11-bảng-chú-giải-thuật-ngữ)

---

## 1. Giới thiệu & Bối cảnh

### 1.1. Mục tiêu tài liệu

Tài liệu này mô tả thiết kế nghiệp vụ và hệ thống của **WebDoanVien Backend** — nền tảng API phục vụ số hóa toàn bộ hoạt động quản lý đoàn viên của **Ban Thanh niên Cảnh sát Thành phố (CATP)**. Tài liệu hướng đến các đối tượng: HR, Quản lý dự án, Nhà phát triển và các bên liên quan muốn hiểu toàn diện về hệ thống.

### 1.2. Vấn đề nghiệp vụ (Problem Statement)

Trước khi hệ thống được xây dựng, tổ chức Đoàn CATP đang đối mặt với các vấn đề:

| Vấn đề | Tác động |
|---|---|
| Hồ sơ đoàn viên quản lý bằng file Excel thủ công | Dễ mất mát, không nhất quán, khó tra cứu |
| Phê duyệt chuyển sinh hoạt qua giấy tờ | Chậm trễ, thiếu audit trail |
| Không có cơ chế kiểm soát quyền truy cập dữ liệu | Cán bộ cơ sở A có thể xem dữ liệu cơ sở B |
| Điểm danh hoạt động bằng danh sách giấy | Dễ gian lận, tốn thời gian tổng hợp |
| Không có kênh thu thập ý kiến đoàn viên | Thiếu cơ chế phản hồi từ cơ sở |

### 1.3. Phạm vi hệ thống (System Scope)

Hệ thống WebDoanVien quản lý toàn bộ vòng đời nghiệp vụ đoàn viên:

```
KẾT NẠP → QUẢN LÝ HỒ SƠ → HOẠT ĐỘNG → ĐÁNH GIÁ → CHUYỂN SINH HOẠT / XUẤT ĐOÀN
```

### 1.4. Cấu trúc tổ chức

```mermaid
graph TD
    A["🏛️ Ban Thanh niên CATP<br/>(Cấp Thành phố – GLOBAL)"]
    A --> B["🏢 Đoàn cơ sở A<br/>(Cấp Cơ sở – UNIT)"]
    A --> C["🏢 Đoàn cơ sở B"]
    A --> D["🏢 Đoàn cơ sở C"]
    B --> B1["👥 Chi đoàn A1"]
    B --> B2["👥 Chi đoàn A2"]
    C --> C1["👥 Chi đoàn B1"]
```

---

## 2. Stakeholders & Actors

### 2.1. Bảng Stakeholders

| Stakeholder | Vai trò | Kỳ vọng |
|---|---|---|
| **Ban Thanh niên CATP** | Sponsor & GLOBAL Admin | Dashboard tổng quan, kiểm soát toàn bộ hệ thống |
| **Bí thư Đoàn cơ sở** | Quản lý cấp cơ sở | Quản lý đoàn viên, tổ chức hoạt động, duyệt ý tưởng |
| **Cán bộ BCH** | Người dùng cán bộ | Giao việc, đánh giá xếp loại, xem báo cáo |
| **Đoàn viên** | End user | Xem thông tin cá nhân, tham gia hoạt động, gửi ý tưởng |
| **ICIB Lab – SGU** | Đội phát triển | Xây dựng và bảo trì hệ thống |

### 2.2. Danh sách Actors

| Actor | Mô tả | Scope |
|---|---|---|
| **`GLOBAL_ADMIN`** | Cán bộ Ban TN CATP, toàn quyền | Toàn bộ hệ thống |
| **`UNIT_SECRETARY`** | Bí thư Chi đoàn cơ sở | Phạm vi đơn vị + chi đoàn con |
| **`UNIT_OFFICER`** | Cán bộ BCH đơn vị | Phạm vi đơn vị được giao |
| **`MEMBER`** | Đoàn viên thông thường | Chỉ dữ liệu cá nhân |
| **`SYSTEM`** | Hệ thống tự động | Cron jobs, QR validation |

---

## 3. Use Case Diagram

### 3.1. Tổng quan Use Case toàn hệ thống

```mermaid
graph LR
    MEMBER(("👤 Đoàn viên"))
    OFFICER(("👔 Cán bộ BCH"))
    SECRETARY(("🔑 Bí thư"))
    ADMIN(("🛡️ GLOBAL Admin"))

    subgraph AUTH["🔐 Xác thực"]
        UC1["Đăng nhập"]
        UC2["Đăng xuất"]
        UC3["Làm mới Token"]
    end

    subgraph MEMBER_MGT["👥 Quản lý Đoàn viên"]
        UC4["Xem hồ sơ cá nhân"]
        UC5["Cập nhật hồ sơ"]
        UC6["Tạo hồ sơ đoàn viên"]
        UC7["Xuất danh sách Excel/PDF"]
        UC8["Xem lịch sử chuyển SHĐ"]
    end

    subgraph DECISION["📋 Quyết định"]
        UC9["Tạo quyết định"]
        UC10["Phê duyệt quyết định"]
        UC11["Từ chối quyết định"]
        UC12["Xem timeline quyết định"]
    end

    subgraph ACTIVITY["🎯 Hoạt động"]
        UC13["Tạo hoạt động"]
        UC14["Sinh mã QR điểm danh"]
        UC15["Quét QR điểm danh"]
        UC16["Xem danh sách người tham gia"]
        UC17["Chốt kết quả hoạt động"]
    end

    subgraph TASK["📌 Giao việc"]
        UC18["Tạo nhiệm vụ"]
        UC19["Gán người thực hiện"]
        UC20["Cập nhật tiến độ"]
        UC21["Đánh giá kết quả"]
    end

    subgraph IDEA["💡 Hộp thư ý tưởng"]
        UC22["Gửi ý tưởng"]
        UC23["Chỉnh sửa ý tưởng"]
        UC24["Duyệt ý tưởng Vòng 1"]
        UC25["Duyệt ý tưởng Vòng 2"]
        UC26["Xem danh sách ý tưởng"]
    end

    subgraph EVAL["📊 Đánh giá xếp loại"]
        UC27["Tạo đánh giá xếp loại"]
        UC28["Xem thống kê xếp loại"]
    end

    subgraph REPORT["📈 Báo cáo"]
        UC29["Xem Dashboard tổng quan"]
        UC30["Xem thống kê theo đơn vị"]
    end

    MEMBER --> UC1
    MEMBER --> UC2
    MEMBER --> UC4
    MEMBER --> UC15
    MEMBER --> UC22
    MEMBER --> UC23
    MEMBER --> UC26

    OFFICER --> UC1
    OFFICER --> UC5
    OFFICER --> UC6
    OFFICER --> UC7
    OFFICER --> UC8
    OFFICER --> UC9
    OFFICER --> UC10
    OFFICER --> UC11
    OFFICER --> UC12
    OFFICER --> UC13
    OFFICER --> UC16
    OFFICER --> UC18
    OFFICER --> UC19
    OFFICER --> UC20
    OFFICER --> UC21
    OFFICER --> UC24
    OFFICER --> UC27
    OFFICER --> UC28

    SECRETARY --> UC10
    SECRETARY --> UC11
    SECRETARY --> UC14
    SECRETARY --> UC17
    SECRETARY --> UC24
    SECRETARY --> UC30

    ADMIN --> UC25
    ADMIN --> UC29
    ADMIN --> UC30
    ADMIN --> UC7
```

---

## 4. Mô tả Use Case chi tiết

### UC-01: Đăng nhập hệ thống

| Thuộc tính | Nội dung |
|---|---|
| **ID** | UC-01 |
| **Tên** | Đăng nhập hệ thống |
| **Actor** | Tất cả người dùng |
| **Tiền điều kiện** | Tài khoản đã được tạo, trạng thái ACTIVE |
| **Luồng chính** | 1. Người dùng nhập `username` + `password` → 2. Hệ thống xác thực → 3. Cấp JWT Access Token + Refresh Token qua HttpOnly Cookie → 4. Redirect về trang chủ |
| **Luồng thay thế** | Sai mật khẩu → Thông báo lỗi 401; Tài khoản bị khóa → Thông báo 403 kèm thời gian mở khóa |
| **Hậu điều kiện** | Người dùng đã xác thực, token hợp lệ trong session |

---

### UC-10: Phê duyệt quyết định đoàn viên

| Thuộc tính | Nội dung |
|---|---|
| **ID** | UC-10 |
| **Tên** | Phê duyệt quyết định |
| **Actor** | Cán bộ BCH, Bí thư, GLOBAL Admin |
| **Tiền điều kiện** | Quyết định ở trạng thái `PENDING`; Actor thuộc đơn vị liên quan |
| **Luồng chính** | 1. Actor xem danh sách quyết định đang chờ → 2. Chọn quyết định cần duyệt → 3. Xem nội dung chi tiết → 4. Nhấn "Phê duyệt" → 5. Hệ thống ghi nhận APPROVED + lưu audit log → 6. Nếu là quyết định Chuyển SHĐ → Hệ thống tự động cập nhật `member.unionUnit` |
| **Luồng thay thế** | Actor không thuộc đơn vị → 403 Forbidden; Quyết định đã được duyệt → Không cho phép |
| **Quy tắc nghiệp vụ** | Quyết định chuyển SHĐ: chỉ cơ sở chuyển đi HOẶC cơ sở tiếp nhận mới có quyền duyệt |
| **Hậu điều kiện** | Quyết định ở trạng thái `APPROVED`; Lịch sử đơn vị được tạo tự động |

---

### UC-15: Quét mã QR điểm danh

| Thuộc tính | Nội dung |
|---|---|
| **ID** | UC-15 |
| **Tên** | Điểm danh hoạt động qua QR |
| **Actor** | Đoàn viên |
| **Tiền điều kiện** | Hoạt động đang ở trạng thái `PUBLISHED`; Mã QR còn hiệu lực |
| **Luồng chính** | 1. Đoàn viên mở app → Quét QR → 2. Frontend gửi `token` lên API `/api/checkin` → 3. Hệ thống giải mã payload → 4. Xác thực token hợp lệ, chưa hết hạn → 5. Ghi nhận `ActivityParticipant` → 6. Trả về thành công |
| **Luồng thay thế** | Token hết hạn → Thông báo "Mã QR đã hết hạn"; Đã điểm danh → Thông báo "Bạn đã điểm danh rồi" |
| **Hậu điều kiện** | Đoàn viên được ghi nhận tham gia hoạt động |

---

### UC-24 & UC-25: Phê duyệt ý tưởng (2 vòng)

| Thuộc tính | UC-24 (Vòng 1) | UC-25 (Vòng 2) |
|---|---|---|
| **Actor** | Bí thư Chi đoàn cơ sở | Cán bộ Ban TN CATP (GLOBAL) |
| **Điều kiện** | Ý tưởng ở `PENDING_UNIT_APPROVAL` | Ý tưởng ở `PENDING_CITY_APPROVAL` |
| **Kết quả Phê duyệt** | Chuyển sang `PENDING_CITY_APPROVAL` | Ý tưởng được `APPROVED` ✓ |
| **Kết quả Từ chối** | `REJECTED` (bắt buộc nhập lý do) | `REJECTED` (bắt buộc nhập lý do) |
| **Ràng buộc** | Không tự duyệt ý tưởng bản thân; Bí thư phải cùng đơn vị với người gửi | Chỉ cán bộ GLOBAL (không có giới hạn đơn vị) |

---

## 5. User Stories & Acceptance Criteria

### US-01: Quản lý hồ sơ đoàn viên

> **Là** Cán bộ BCH,  
> **Tôi muốn** tìm kiếm và xem hồ sơ đoàn viên trong đơn vị của tôi,  
> **Để** nắm bắt thông tin và hỗ trợ các thủ tục hành chính.

**Acceptance Criteria:**
- [ ] Tìm kiếm được theo tên, CCCD, số thẻ đoàn viên
- [ ] Lọc được theo đơn vị, trạng thái (ACTIVE/SUSPENDED), giới tính
- [ ] Kết quả phân trang (mặc định 10/trang)
- [ ] **Chỉ hiển thị đoàn viên thuộc phạm vi đơn vị của cán bộ đăng nhập** ← Yêu cầu bảo mật cốt lõi
- [ ] Không hiển thị đoàn viên đã bị xóa mềm (`deleted_at IS NOT NULL`)

---

### US-02: Gửi và theo dõi ý tưởng

> **Là** Đoàn viên,  
> **Tôi muốn** gửi ý tưởng/sáng kiến và theo dõi quá trình phê duyệt,  
> **Để** đóng góp vào hoạt động phong trào của đơn vị.

**Acceptance Criteria:**
- [ ] Đoàn viên điền tiêu đề, mô tả và đính kèm tệp (tùy chọn)
- [ ] Sau khi gửi, ý tưởng ở trạng thái `PENDING_UNIT_APPROVAL`
- [ ] Đoàn viên xem được danh sách ý tưởng của bản thân qua `/api/ideas/my-ideas`
- [ ] Xem được trạng thái hiện tại và lịch sử các lượt phê duyệt
- [ ] **Không thể tự phê duyệt ý tưởng của chính mình**
- [ ] Được thông báo khi ý tưởng được phê duyệt/từ chối/yêu cầu chỉnh sửa

---

### US-03: Tổ chức hoạt động và điểm danh QR

> **Là** Bí thư Chi đoàn,  
> **Tôi muốn** tạo hoạt động và điểm danh đoàn viên tự động qua QR,  
> **Để** tiết kiệm thời gian và có dữ liệu tham gia chính xác.

**Acceptance Criteria:**
- [ ] Tạo hoạt động với thông tin: tên, loại, thời gian, địa điểm, người phụ trách
- [ ] Sinh mã QR duy nhất cho từng hoạt động (payload được mã hóa)
- [ ] Đoàn viên quét QR → hệ thống ghi nhận tức thì (không cần thao tác thủ công)
- [ ] Chặn điểm danh trùng lặp cho cùng một đoàn viên trong một hoạt động
- [ ] Xem được danh sách người đã điểm danh theo thời gian thực
- [ ] Sau khi chốt (`is_finalized = true`), không cho phép điểm danh thêm

---

### US-04: Xuất báo cáo danh sách đoàn viên

> **Là** Cán bộ BCH / GLOBAL Admin,  
> **Tôi muốn** xuất danh sách đoàn viên ra file Excel hoặc PDF,  
> **Để** phục vụ báo cáo định kỳ lên cấp trên.

**Acceptance Criteria:**
- [ ] Lọc theo đơn vị, trạng thái, từ khóa trước khi xuất
- [ ] **File xuất ra chỉ chứa dữ liệu trong phạm vi quyền hạn của người xuất**
- [ ] File Excel (.xlsx) đúng định dạng báo cáo, có header đầy đủ
- [ ] File PDF có logo, ngày xuất và tổng số bản ghi
- [ ] Thời gian tạo file ≤ 5 giây với tối đa 1.000 bản ghi

---

### US-05: Đánh giá xếp loại định kỳ

> **Là** Cán bộ BCH,  
> **Tôi muốn** đánh giá xếp loại đoàn viên theo quý/năm,  
> **Để** có cơ sở xét khen thưởng và theo dõi tiến bộ của đoàn viên.

**Acceptance Criteria:**
- [ ] Chọn đoàn viên, kỳ đánh giá (quý/năm), kết quả xếp loại và ghi chú
- [ ] Hệ thống ngăn chặn việc đánh giá đoàn viên không thuộc đơn vị của cán bộ
- [ ] Hỗ trợ tạo phiên bản chỉnh sửa (`EvaluationRevision`) khi cần cập nhật
- [ ] Dashboard hiển thị thống kê 4 card: Xuất sắc / Tốt / Khá / Trung bình
- [ ] Xem được lịch sử đánh giá qua các kỳ của từng đoàn viên

---

## 6. Activity Diagrams – Luồng nghiệp vụ

### 6.1. Luồng phê duyệt Ý tưởng (2 vòng)

```mermaid
flowchart TD
    START([🟢 Bắt đầu]) --> A["Đoàn viên gửi ý tưởng\n+ đính kèm tệp (tuỳ chọn)"]
    A --> B{{"Kiểm tra\nquyền gửi?"}}
    B -- "Không có quyền" --> ERR1["❌ 403 Forbidden"]
    B -- "Hợp lệ" --> C["Lưu Idea\nStatus = PENDING_UNIT_APPROVAL"]

    C --> D["Hệ thống thông báo\nBí thư cơ sở"]
    D --> E{{"Bí thư cơ sở\nxem xét"}}

    E --> F["Tạo IdeaApproval\n(Vòng 1)"]
    F --> G{{"Kết quả\nVòng 1?"}}

    G -- "Phê duyệt" --> H["result = UNIT_APPROVED\nStatus → PENDING_CITY_APPROVAL"]
    G -- "Từ chối" --> I["result = REJECTED\n(bắt buộc lý do)\nStatus → REJECTED"]
    G -- "Yêu cầu chỉnh sửa" --> J{{"Còn\nlượt chỉnh sửa?"}}

    J -- "Vượt maxRevisionTurns" --> K["❌ Lỗi:\nPhải Duyệt hoặc Từ chối"]
    J -- "Còn lượt" --> L["result = REQUEST_REVISION\nĐoàn viên chỉnh sửa lại"]
    L --> C

    H --> M["Thông báo\nBan TN CATP"]
    M --> N{{"Cán bộ CATP\n(GLOBAL) xem xét"}}
    N --> O["Tạo IdeaApproval\n(Vòng 2)"]
    O --> P{{"Kết quả\nVòng 2?"}}

    P -- "Phê duyệt" --> Q["result = APPROVED\nStatus → APPROVED ✅"]
    P -- "Từ chối" --> R["result = REJECTED\nStatus → REJECTED ❌"]
    P -- "Yêu cầu chỉnh sửa" --> L

    I --> END_REJECT([🔴 Kết thúc: Từ chối])
    R --> END_REJECT
    Q --> END_APPROVE([🟢 Kết thúc: Được duyệt])
```

---

### 6.2. Luồng phê duyệt Quyết định Chuyển sinh hoạt Đoàn

```mermaid
flowchart TD
    S([🟢 Bắt đầu]) --> A1["Cán bộ tạo Quyết định\nLoại: CHUYEN_SINH_HOAT\nTừ đơn vị A → Đơn vị B"]
    A1 --> A2["Status = PENDING\nLưu vào member_decisions"]
    A2 --> A3["Thông báo các\nbên liên quan"]

    A3 --> B1{{"Cán bộ có quyền\nduyệt không?"}}
    B1 -- "Không thuộc\nđơn vị A hoặc B" --> B2["❌ 403 Forbidden"]
    B1 -- "Thuộc đơn vị A (chuyển đi)\nHOẶC đơn vị B (tiếp nhận)\nHOẶC GLOBAL" --> B3["Xem chi tiết quyết định"]

    B3 --> C1{{"Quyết định\ncủa cán bộ?"}}

    C1 -- "Phê duyệt" --> D1["Status → APPROVED\nGhi DecisionStatusHistory"]
    C1 -- "Từ chối" --> D2["Status → REJECTED\n(bắt buộc nhập lý do)\nGhi DecisionStatusHistory"]

    D1 --> E1{{"Ngày hiệu lực\nđã đến chưa?"}}
    E1 -- "Chưa đến\n(future effectiveDate)" --> E2["⏳ Ghi nhận,\nchờ đến ngày hiệu lực"]
    E1 -- "Đã đến / Không có\nngày hiệu lực" --> E3["🔄 Tự động cập nhật:\nmember.unionUnit = đơn vị B\nTạo MemberUnitHistory"]

    E3 --> F1([🟢 Hoàn tất\nĐoàn viên đã chuyển])
    E2 --> F2([⏸️ Đang chờ\nhiệu lực])
    D2 --> F3([🔴 Kết thúc:\nBị từ chối])
```

---

### 6.3. Luồng điểm danh QR Code

```mermaid
flowchart TD
    S([🟢 Bắt đầu]) --> A["Bí thư tạo Hoạt động\nStatus = DRAFT"]
    A --> B["Phát hành hoạt động\nStatus → PUBLISHED"]
    B --> C["Hệ thống sinh mã QR\nPayload mã hoá = activityId + token + expiry"]
    C --> D["Bí thư chiếu/in QR\ntại địa điểm tổ chức"]

    D --> E["Đoàn viên đến\nQuét mã QR"]
    E --> F{{"Token\nhợp lệ?"}}
    F -- "Hết hạn / Sai" --> G["❌ Thông báo:\nMã QR không hợp lệ"]
    F -- "Hợp lệ" --> H{{"Hoạt động\nstill PUBLISHED?"}}
    H -- "Đã kết thúc/huỷ" --> I["❌ Thông báo:\nHoạt động đã kết thúc"]
    H -- "Đang diễn ra" --> J{{"Đoàn viên\nđã điểm danh chưa?"}}
    J -- "Đã điểm danh" --> K["⚠️ Thông báo:\nBạn đã điểm danh rồi"]
    J -- "Chưa" --> L["✅ Ghi nhận\nActivityParticipant\n+ Timestamp"]
    L --> M["Trả về xác nhận\nĐiểm danh thành công"]

    B --> N["Bí thư chốt\nhoạt động\nis_finalized = true"]
    N --> O["Status → COMPLETED\nKhóa điểm danh thêm"]
    O --> P[["📊 Xuất danh sách\nngười tham gia"]]
```

---

### 6.4. Luồng Giao việc & Theo dõi nhiệm vụ

```mermaid
flowchart LR
    A["Cán bộ tạo Task\nStatus = DRAFT"] --> B["Gán người thực hiện\nTaskAssignee"]
    B --> C["Phát hành\nStatus → PUBLISHED"]
    C --> D["Người nhận\nxác nhận nhận việc"]
    D --> E["Cập nhật tiến độ\ntheo thời gian"]
    E --> F{{"Hoàn thành?"}}
    F -- "Chưa" --> E
    F -- "Hoàn thành" --> G["Status → COMPLETED"]
    G --> H["Cán bộ đánh giá kết quả\nevaluationResult: EXCELLENT/GOOD/FAIR/POOR"]
    H --> I([✅ Kết thúc])
```

---

## 7. Sequence Diagrams

### 7.1. Luồng Đăng nhập – Refresh Token

```mermaid
sequenceDiagram
    actor U as 👤 Người dùng
    participant FE as Frontend
    participant API as API Server
    participant DB as PostgreSQL

    U->>FE: Nhập username / password
    FE->>API: POST /api/auth/login
    API->>DB: SELECT * FROM users WHERE username = ?
    DB-->>API: User entity
    API->>API: Verify password (BCrypt)
    API->>API: Build JWT (userId, unionUnitId, roles)
    API->>DB: INSERT INTO refresh_tokens (token, userId, expiry)
    API-->>FE: 200 OK + Set-Cookie: access_token, refresh_token (HttpOnly)
    FE-->>U: Redirect đến trang chủ

    Note over FE,API: Khi Access Token hết hạn (15 phút)

    FE->>API: POST /api/auth/refresh (Cookie: refresh_token)
    API->>DB: SELECT * FROM refresh_tokens WHERE token = ?
    DB-->>API: RefreshToken entity
    API->>API: Kiểm tra còn hạn, chưa bị thu hồi
    API->>API: Cấp Access Token mới
    API-->>FE: 200 OK + Set-Cookie: access_token mới
```

---

### 7.2. Luồng API bảo vệ bởi phân quyền 2 tầng

```mermaid
sequenceDiagram
    actor U as 👔 Cán bộ BCH
    participant API as Controller
    participant USC as UnitScopeResolver
    participant MDC as MemberDataScopeService
    participant SVC as MemberService
    participant DB as PostgreSQL

    U->>API: GET /api/members/{memberId}
    API->>API: Trích xuất JWT từ Cookie
    API->>USC: resolveAllowedUnitIds()
    USC->>USC: Đọc unionUnitId từ JWT claim (O(1))
    USC->>DB: BFS duyệt cây đơn vị con (nếu có cache thì bỏ qua)
    DB-->>USC: Danh sách unitIds hợp lệ
    USC->>USC: assertMemberAllowed(member)
    alt member.unionUnit KHÔNG thuộc allowedUnitIds
        USC-->>API: 403 Forbidden ❌
        API-->>U: 403: Không có quyền truy cập đoàn viên này
    else Hợp lệ - Qua Tầng 1
        USC-->>API: ✅ Pass
        API->>MDC: requireAccess("MEMBER_VIEW", member)
        MDC->>DB: SELECT user_roles WHERE userId = ? AND permission = 'MEMBER_VIEW'
        DB-->>MDC: Scope assignments
        MDC->>MDC: Kiểm tra SELF / UNIT / CHILDREN_UNIT / GLOBAL
        alt Không có quyền MEMBER_VIEW
            MDC-->>API: 403 Forbidden ❌
        else Có quyền - Qua Tầng 2
            MDC-->>API: ✅ Pass
            API->>SVC: getMemberById(memberId)
            SVC->>DB: SELECT * FROM members WHERE id = ?
            DB-->>SVC: Member entity
            SVC-->>API: MemberDetailResponse
            API-->>U: 200 OK + dữ liệu đoàn viên
        end
    end
```

---

### 7.3. Luồng Phê duyệt Ý tưởng Vòng 1

```mermaid
sequenceDiagram
    actor BS as 🔑 Bí thư cơ sở
    participant API as IdeaApprovalController
    participant SVC as IdeaApprovalServiceImpl
    participant USC as UnitScopeResolver
    participant DB as PostgreSQL

    BS->>API: POST /api/ideas/{ideaId}/approvals\n{result: "APPROVED"}
    API->>SVC: createApproval(ideaId, request)
    SVC->>DB: findById(ideaId) - Lấy ý tưởng
    DB-->>SVC: Idea entity (kèm member.unionUnit)

    SVC->>DB: findTop...OrderByTurnDesc - Lấy lượt duyệt cuối
    DB-->>SVC: latestApproval (hoặc empty)

    SVC->>SVC: resolveStatus(allApprovals)\n→ currentStatus = PENDING_UNIT_APPROVAL

    SVC->>SVC: Kiểm tra idea chưa kết thúc quy trình

    SVC->>DB: getCurrentMember() - Lấy thông tin Bí thư
    DB-->>SVC: currentMember

    SVC->>SVC: isSelf? (currentMember == idea.member)\n→ FALSE (OK, không tự duyệt)

    SVC->>USC: getCurrentSecretaryUnitId()
    USC-->>SVC: secretaryUnitId

    SVC->>SVC: secretaryUnitId == idea.member.unionUnit.id?\n→ TRUE (cùng đơn vị ✅)

    SVC->>SVC: Vòng 1 duyệt → dbResult = "UNIT_APPROVED"

    SVC->>DB: save(IdeaApproval{result=UNIT_APPROVED, turn=1})
    DB-->>SVC: Saved approval

    SVC-->>API: IdeaApprovalResponse
    API-->>BS: 200 OK - Ý tưởng chuyển sang Vòng 2
```

---

## 8. Entity Relationship Diagram (ERD)

```mermaid
erDiagram
    users {
        Long id PK
        String username UK
        String password_hash
        String full_name
        String status
        LocalDateTime locked_until
        LocalDateTime deleted_at
        Long created_by_user_id FK
    }

    roles {
        Long id PK
        String code UK
        String name UK
        Short level
        String max_scope
        Boolean is_system
        String status
    }

    permissions {
        Long id PK
        String code UK
        String name
        String resource
        String action
    }

    user_roles {
        Long id PK
        Long user_id FK
        Long role_id FK
        Long unit_id FK
        String scope_level
        Boolean is_active
        Long assigned_by_user_id FK
    }

    unit_levels {
        Long id PK
        String name
        Short order_index
    }

    units {
        Long id PK
        String name
        String address
        Long unit_level_id FK
        Long parent_unit_id FK
        String status
    }

    positions {
        Long id PK
        String name
        String code
    }

    members {
        Long id PK
        Long user_id FK
        Long union_unit_id FK
        Long work_unit_id FK
        Long position_id FK
        Long supervisor_member_id FK
        String first_name
        String last_name
        LocalDate date_of_birth
        Boolean gender
        String citizen_id UK
        String phone
        String email
        String union_card_number UK
        LocalDate join_union_date
        String status
    }

    member_decisions {
        Long id PK
        Long member_id FK
        String decision_type
        String decision_number
        LocalDate issued_date
        LocalDate effective_date
        Long issued_by_unit_id FK
        Long to_unit_id FK
        String status
        Long created_by_user_id FK
    }

    member_evaluations {
        Long id PK
        Long member_id FK
        String eval_type
        Integer quarter
        Integer year
        String result
        String note
    }

    activities {
        Long id PK
        Long organizing_unit_id FK
        String title
        String activity_type
        LocalDateTime start_time
        LocalDateTime end_time
        String status
        Boolean is_finalized
        Long created_by_user_id FK
    }

    activity_qr_codes {
        Long id PK
        Long activity_id FK
        String qr_payload_encrypted
        LocalDateTime expires_at
    }

    activity_participants {
        Long activity_id FK
        Long member_id FK
        LocalDateTime checked_in_at
    }

    tasks {
        Long id PK
        Long owner_unit_id FK
        String title
        String status
        String evaluation_result
        Long created_by_user_id FK
    }

    task_assignees {
        Long task_id FK
        Long member_id FK
        String status
    }

    ideas {
        Long id PK
        String title
        String description
        Long member_id FK
        Long idea_file_id FK
        LocalDateTime created_at
    }

    idea_approvals {
        Long id PK
        Long idea_id FK
        Long approver_id FK
        Short approval_turn
        String result
        String reason
    }

    file_assets {
        Long id PK
        String original_name
        String stored_path
        String mime_type
        Long file_size
    }

    users ||--o{ user_roles : "có"
    roles ||--o{ user_roles : "gán cho"
    units ||--o{ user_roles : "phạm vi"
    roles }o--o{ permissions : "role_permissions"
    units ||--o{ units : "parent_unit_id"
    unit_levels ||--o{ units : "có cấp"
    users |o--o| members : "liên kết"
    units ||--o{ members : "sinh hoạt tại"
    positions ||--o{ members : "giữ chức vụ"
    members ||--o{ member_decisions : "có quyết định"
    members ||--o{ member_evaluations : "được đánh giá"
    units ||--o{ activities : "tổ chức"
    activities ||--o{ activity_qr_codes : "có QR"
    activities ||--o{ activity_participants : "ghi nhận"
    members ||--o{ activity_participants : "tham gia"
    units ||--o{ tasks : "owner"
    tasks ||--o{ task_assignees : "gán cho"
    members ||--o{ task_assignees : "thực hiện"
    members ||--o{ ideas : "gửi"
    ideas ||--o{ idea_approvals : "có lượt duyệt"
    file_assets ||--o{ ideas : "đính kèm"
    file_assets ||--o{ member_decisions : "kèm file"
    file_assets ||--o| members : "avatar"
```

---

## 9. Kiến trúc hệ thống & Phân quyền

### 9.1. Kiến trúc tổng thể

```mermaid
graph TB
    subgraph CLIENT["🖥️ Client Layer"]
        WEB["Web App (Vue/React)"]
        MOBILE["Mobile App"]
    end

    subgraph GATEWAY["🔐 Security Layer"]
        COOKIE["HttpOnly Cookie\n(JWT)"]
        SEC["Spring Security\nFilter Chain"]
        GUARD["@PreAuthorize\nPermissionGuard"]
    end

    subgraph API["⚙️ Application Layer (Spring Boot 3.5)"]
        CTRL["Controllers\n(REST Endpoints)"]
        SVC["Services\n(Business Logic)"]
        SCOPE["UnitScopeResolver\n+ MemberDataScope"]
    end

    subgraph DATA["💾 Data Layer"]
        REPO["Spring Data JPA\nRepositories"]
        SPEC["JPA Specifications\n(Dynamic Queries)"]
        PG[("PostgreSQL 16\n+ Flyway Migration")]
    end

    subgraph INFRA["🐳 Infrastructure"]
        DOCKER["Docker + Docker Compose"]
        CACHE["Spring Cache\n(unitSubtree)"]
        SWAGGER["Swagger UI\n(Dev only)"]
    end

    WEB & MOBILE --> COOKIE --> SEC --> GUARD --> CTRL
    CTRL --> SVC --> SCOPE
    SCOPE --> REPO --> SPEC --> PG
    SVC --> CACHE
    DOCKER -.->|"hosts"| PG
```

---

### 9.2. Mô hình phân quyền dữ liệu

```mermaid
graph TD
    REQ["📨 HTTP Request"] --> SEC["Spring Security\nXác thực JWT"]
    SEC --> PERM["@PreAuthorize\nKiểm tra Permission Code\nvd: 'IDEA_VIEW'"]
    PERM --> SCOPE1

    subgraph LAYER1["🛡️ Tầng 1: UnitScope (Stateless - Fast)"]
        SCOPE1["UnitScopeResolver\n.resolveAllowedUnitIds()"]
        SCOPE1 --> CHK1{{"unionUnitId\ntrong JWT?"}}
        CHK1 -- "null → GLOBAL" --> PASS1["✅ Toàn quyền"]
        CHK1 -- "có giá trị" --> BFS["BFS duyệt\ncây đơn vị\n(Cached)"]
        BFS --> ALLOWED["Set allowedUnitIds"]
        ALLOWED --> ASSERT["assertMemberAllowed()\nhoặc assertUnitAllowed()"]
        ASSERT -- "Ngoài phạm vi" --> DENY1["❌ 403 Forbidden"]
        ASSERT -- "Trong phạm vi" --> PASS1
    end

    PASS1 --> SCOPE2

    subgraph LAYER2["🔍 Tầng 2: MemberDataScope (Stateful - DB)"]
        SCOPE2["MemberDataScopeService\n.requireAccess(permission, member)"]
        SCOPE2 --> DB["SELECT user_roles\nWHERE permission = ?"]
        DB --> SCOPECHK{{"scope_level?"}}
        SCOPECHK -- "GLOBAL" --> PASS2["✅ Toàn quyền"]
        SCOPECHK -- "SELF" --> SELFCHK["member.userId\n== currentUserId?"]
        SCOPECHK -- "UNIT/CHILDREN_UNIT" --> UNITCHK["member.unionUnit\nIN (unitIds)?"]
        SELFCHK & UNITCHK -- "Không hợp lệ" --> DENY2["❌ 403 Forbidden"]
        SELFCHK & UNITCHK -- "Hợp lệ" --> PASS2
    end

    PASS2 --> RESULT["✅ Thực thi nghiệp vụ\nTrả về dữ liệu"]
```

---

## 10. Non-Functional Requirements

### 10.1. Bảo mật (Security)

| Yêu cầu | Mô tả | Hiện trạng |
|---|---|---|
| **Authentication** | JWT lưu trong HttpOnly Cookie (không thể đọc từ JS) | ✅ Đã implement |
| **Authorization** | RBAC với Scope level (SELF/UNIT/CHILDREN_UNIT/GLOBAL) | ✅ Đã implement |
| **Data Isolation** | Cô lập dữ liệu tuyệt đối giữa các đơn vị | ✅ 2-layer security |
| **Audit Trail** | Ghi nhận người tạo/sửa + timestamp mọi thao tác | ✅ Toàn hệ thống |
| **Soft Delete** | Không xóa vật lý, dữ liệu luôn được preserved | ✅ Toàn hệ thống |
| **CCCD Protection** | Xử lý số CCCD theo quy định bảo mật dữ liệu cá nhân | ✅ Có audit report riêng |

### 10.2. Hiệu năng (Performance)

| Yêu cầu | Mục tiêu | Giải pháp |
|---|---|---|
| **API response time** | ≤ 300ms cho các API danh sách thông thường | JPA Specification + Index |
| **Security check** | O(1) cho UnitScope từ JWT | Không cần DB query |
| **Tree traversal** | Cache kết quả BFS cây đơn vị | `@Cacheable("unitSubtree")` |
| **File export** | ≤ 5 giây cho 1.000 bản ghi | Apache POI + streaming |

### 10.3. Độ sẵn sàng & Triển khai (Availability & Deployment)

| Yêu cầu | Mô tả |
|---|---|
| **Containerization** | Docker + Docker Compose, không phụ thuộc môi trường cục bộ |
| **Database Migration** | Flyway tự động, không cần chạy script thủ công |
| **Multi-profile** | `dev` / `test` / `prod` cho từng môi trường |
| **Health Check** | `/actuator/health` cho load balancer monitoring |
| **API Documentation** | Swagger UI tự động từ code (Springdoc OpenAPI) |

### 10.4. Khả năng mở rộng (Scalability)

| Yêu cầu | Mô tả |
|---|---|
| **Modular Architecture** | Thêm module mới (vd: `notification`) không ảnh hưởng module cũ |
| **Stateless API** | Mỗi request tự chứa thông tin xác thực → Dễ horizontal scaling |
| **Generic Specification** | `UnitScopeSpecification` dùng chung cho mọi module mới |

---

## 11. Bảng chú giải thuật ngữ

| Thuật ngữ | Giải thích |
|---|---|
| **CATP** | Cảnh sát Thành phố |
| **BCH** | Ban Chấp hành |
| **SHĐ** | Sinh hoạt Đoàn |
| **GLOBAL** | Phạm vi toàn hệ thống (không giới hạn đơn vị) |
| **UNIT** | Phạm vi một đơn vị cụ thể |
| **CHILDREN_UNIT** | Phạm vi đơn vị và toàn bộ chi đoàn con trực thuộc |
| **SELF** | Chỉ phạm vi dữ liệu cá nhân của người dùng đó |
| **BFS** | Breadth-First Search – thuật toán duyệt cây theo chiều rộng |
| **JWT** | JSON Web Token – chuỗi xác thực stateless |
| **HttpOnly Cookie** | Cookie không thể đọc bởi JavaScript, chống XSS |
| **Soft Delete** | Xóa mềm – đánh dấu `deleted_at` thay vì xóa khỏi DB |
| **Audit Trail** | Nhật ký theo dõi mọi thay đổi trong hệ thống |
| **JPA Specification** | Kỹ thuật tạo câu query động trong Spring Data JPA |
| **RBAC** | Role-Based Access Control – phân quyền theo vai trò |
| **Flyway** | Công cụ quản lý phiên bản schema database |
| **Modular Monolith** | Kiến trúc một ứng dụng nhưng chia thành các module độc lập |

---

*Tài liệu này được tổng hợp và soạn thảo dựa trên phân tích mã nguồn thực tế của dự án WebDoanVien Backend.*  
*Mọi sơ đồ trong tài liệu sử dụng ký pháp Mermaid, có thể render trực tiếp trên GitHub, Notion, Confluence.*


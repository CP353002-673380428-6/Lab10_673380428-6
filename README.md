> นี่คือเนื้อหาไฟล์ **`README.md`** ภาษาไทยฉบับสมบูรณ์สำหรับ **Lab 10: Spring WebFlux & WebClient** ครับ จัดรูปแบบ Markdown อย่างเป็นระเบียบ ครอบคลุมทั้งทฤษฎี Reactive Programming, การเปรียบเทียบ Mono vs Flux, คำอธิบาย REST Endpoints ทั้ง 6 เส้นทาง, การใช้งาน WebClient, คำสั่งทดสอบ และคู่มือการรันโปรเจกต์ สามารถคัดลอกไปวางในไฟล์ `README.md` ของโปรเจกต์ได้ทันทีครับ

# ⚡ Lab 10: Spring WebFlux & WebClient

**วิชา:** CP353002 หลักการออกแบบและพัฒนาซอฟต์แวร์ (Principles of Software Design)
**ผู้จัดทำ:** นายสิทธิโชค มุขนาค | รหัสนักศึกษา: `673380428-6` | กลุ่มเรียน (Section): `4`
**เทคโนโลยีหลัก:** Java 21 / Spring Boot 3.3.x / Spring WebFlux / Project Reactor / Reactor Netty

---

## 📋 1. วัตถุประสงค์ของ Lab (Objectives)

1. เข้าใจหลักการของ **Reactive Programming** และความแตกต่างระหว่างสถาปัตยกรรม **Non-blocking I/O** กับ **Blocking I/O (Thread-per-request)**
2. เข้าใจและประยุกต์ใช้ Reactive Streams Publishers พื้นฐาน ได้แก่ **`Mono<T>`** (0..1 ค่า) และ **`Flux<T>`** (0..N ค่า)
3. พัฒนา Reactive RESTful API ด้วย **Spring WebFlux** และ `@RestController` โดยทำงานบนเซิร์ฟเวอร์ **Reactor Netty**
4. เรียกใช้งาน External HTTP API แบบ Asynchronous Non-blocking ด้วย **`WebClient`**
5. ฝึกฝนการใช้ Reactor Operators สำคัญ เช่น `map`, `filter`, `switchIfEmpty`, `fromIterable`, `doOnNext`

---

## 🛠️ 2. เทคโนโลยีที่ใช้ (Tech Stack)

* **Language:** Java 21 (LTS)
* **Framework:** Spring Boot 3.3.x
  * **Spring WebFlux:** Reactive Web Framework
  * **Project Reactor:** Reactive Streams Implementation (Mono & Flux)
  * **Reactor Netty:** Non-blocking Event-Driven Web Server (รันแทน Tomcat)
* **Build Tool:** Apache Maven
* **Data Storage:** In-Memory Storage (`ConcurrentHashMap` เพื่อความปลอดภัยด้าน Thread-Safety)
* **Testing Tools:** cURL / Postman / Web Browser

---

## 🧠 3. สรุปความรู้เชิงทฤษฎี (Reactive Concepts)

### 3.1 Reactive Programming vs Blocking I/O

| หัวข้อ | Blocking I/O (Spring MVC + Tomcat) | Reactive Non-blocking (Spring WebFlux + Netty) |
|---|---|---|
| **รูปแบบการทำงาน** | 1 Thread รับผิดชอบ 1 Request (Thread-per-request) | Event Loop ใช้เธรดจำนวนน้อยรับคำขอได้มหาศาล |
| **พฤติกรรมระหว่างรอ I/O** | เธรดจะหยุดรอ (Blocked/Waiting) สิ้นเปลือง Memory | เธรดไม่หยุดรอ (Non-blocking) ไปทำงานอื่นต่อทันทีก่อนมี Event ส่งกลับมา |
| **ชนิดข้อมูลตอบกลับ** | Object ธรรมดา (เช่น `Product`, `List<Product>`) | Reactive Types (`Mono<Product>`, `Flux<Product>`) |
| **กรณีการใช้งาน** | งาน CRUD ทั่วไปที่มีผู้ใช้ระดับปกติ | ระบบที่ต้องการ High-Concurrency, Real-time Streaming, Microservices |

---

### 3.2 Mono vs Flux

* **`Mono<T>` (0 หรือ 1 ค่า):**
  * ใช้กับผลลัพธ์ที่เป็นข้อมูลชิ้นเดียว เช่น `findById()`, `save()`, `getDiscountedPrice()`
  * ใช้กับคำสั่งที่ไม่ต้องการผลลัพธ์ตอบกลับ (Side-effect) เช่น `Mono<Void>` สำหรับ `deleteById()`
* **`Flux<T>` (0 ถึง N ค่า):**
  * ใช้กับผลลัพธ์ที่เป็นชุดข้อมูลหรือสตรีมต่อเนื่อง เช่น `findAll()`, `findByCategory()`

---

### 3.3 สรุป Reactor Operators ที่ใช้งานในแล็บนี้

* **`.just(value)` / `.empty()`:** สร้าง `Mono` จากออบเจกต์ หรือสร้าง Publisher เปล่า
* **`.fromIterable(collection)`:** แปลง Collection ให้กลายเป็น `Flux` สตรีมข้อมูลออกมาทีละรายการ
* **`.map(Function)`:** แปลงข้อมูลจากชนิดหนึ่งเป็นอีกชนิดหนึ่งแบบ Synchronous (เช่น `Product` ➔ `Double`)
* **`.filter(Predicate)`:** กรองข้อมูลตามเงื่อนไขที่กำหนด (เช่น กรองตามหมวดหมู่ Category)
* **`.switchIfEmpty(Mono)`:** กำหนดกระบวนการสำรองหากสตรีมว่างเปล่า (ใช้ส่ง `Mono.error()` เมื่อไม่พบข้อมูล)
* **`.doOnNext(Consumer)`:** ดักจับค่าที่ไหลผ่านสตรีมเพื่อทำ Side-effect เช่น การพิมพ์ Log โดยไม่เปลี่ยนแปลงข้อมูล

---

## 📂 4. โครงสร้างโปรเจกต์ (Project Structure)

```text
src/main/java/com/example/lab10/
├── Lab10Application.java          ← Spring Boot Main Application (รันบน Reactor Netty)
├── AppConfig.java                 ← ประกาศ @Bean ProductRepository
├── model/
│   └── Product.java               ← Data Model พร้อมฟังก์ชันคำนวณส่วนลด
├── repository/
│   └── ProductRepository.java     ← In-memory Reactive Data Access (ConcurrentHashMap)
├── service/
│   └── ProductService.java        ← Business Logic Layer (จัดการ UUID, Exception, กรองข้อมูล)
├── controller/
│   └── ProductController.java     ← Reactive REST Controller (คืนค่า Mono/Flux)
└── client/
    └── ProductWebClient.java      ← Reactive HTTP Client เรียก API แบบ Non-blocking
```

---

## 🌐 5. รายละเอียด REST API Endpoints และตัวอย่างการทดสอบ

เซิร์ฟเวอร์รันบนพอร์ตเริ่มต้น: `http://localhost:8080`

| ลำดับ | HTTP Method | URL Endpoint | Return Type | คำอธิบาย |
|:---:|:---:|---|:---:|---|
| **1** | **GET** | `/products` | `Flux<Product>` | ดึงรายการสินค้าทั้งหมด |
| **2** | **GET** | `/products/{id}` | `Mono<Product>` | ค้นหาสินค้าตามรหัส (ID) |
| **3** | **POST** | `/products` | `Mono<Product>` | บันทึกสินค้าใหม่ (สร้าง UUID อัตโนมัติถ้าไม่มี ID) |
| **4** | **DELETE** | `/products/{id}` | `Mono<Void>` | ลบสินค้าตามรหัส (ID) |
| **5** | **GET** | `/products/category/{category}` | `Flux<Product>` | ค้นหาและกรองสินค้าตามหมวดหมู่ |
| **6** | **GET** | `/products/{id}/price` | `Mono<Double>` | คำนวณและคืนค่าราคาหลังหักส่วนลด |

---

### 🧪 ตัวอย่างคำสั่งทดสอบผ่าน cURL

#### 1. ดูรายการสินค้าทั้งหมด (GET)
```bash
curl http://localhost:8080/products
```
* **ผลลัพธ์:** แสดงสินค้าทั้งหมด โดยตัวแรกมีชื่อและรหัสนักศึกษา:
```json
[
  {
    "id": "1",
    "name": "iPhone 15 Pro (673380428-6 Sitthichok)",
    "category": "Electronics",
    "brand": "Apple",
    "stock": 50,
    "price": 39900.0,
    "discountType": "MEMBER",
    "discountedPrice": 35910.0
  },
  ...
]
```

#### 2. ดูข้อมูลสินค้าตาม ID (GET)
```bash
curl http://localhost:8080/products/1
```

#### 3. เพิ่มสินค้าใหม่ (POST)
```bash
curl -X POST http://localhost:8080/products \
  -H "Content-Type: application/json" \
  -d '{"name":"iPad Pro M4","category":"Electronics","brand":"Apple","stock":15,"price":35900.0,"discountType":"SEASONAL"}'
```

#### 4. กรองสินค้าตามหมวดหมู่ (GET)
```bash
curl http://localhost:8080/products/category/Electronics
```

#### 5. ดูราคาหลังหักส่วนลด (GET)
```bash
curl http://localhost:8080/products/1/price
```
* **ผลลัพธ์:** ได้รับราคาหลังหักส่วนลดสมาชิก 10% คือ `35910.0`

#### 6. ลบสินค้า (DELETE)
```bash
curl -i -X DELETE http://localhost:8080/products/2
```

---

## 📡 6. การทำงานของ WebClient (`ProductWebClient.java`)

`ProductWebClient` ถูกสร้างขึ้นเพื่อทำหน้าที่เป็น Non-blocking HTTP Client เรียกใช้งาน Endpoints ของตนเอง โดยทำงานแบบ Method Chaining:

```java
// ตัวอย่าง: การส่งคำขอ GET และแปลงผลลัพธ์เป็น Flux แบบ Asynchronous
public Flux<Product> getAllProducts() {
    return client.get()
            .uri("/products")
            .retrieve()
            .bodyToFlux(Product.class);
}

// ตัวอย่าง: การดึงราคาพร้อมดักจับข้อมูลเพื่อพิมพ์ Log ผ่าน .doOnNext()
public Mono<Double> getDiscountedPrice(String id) {
    return client.get()
            .uri("/products/{id}/price", id)
            .retrieve()
            .bodyToMono(Double.class)
            .doOnNext(price -> System.out.println("Price: " + price));
}
```

---

## 🚀 7. การติดตั้งและเริ่มต้นใช้งาน (Getting Started)

### 1) ตรวจสอบ Environment ของเครื่อง
* **Java:** JDK 21 หรือ 17
* **Maven:** Apache Maven 3.9+

### 2) สั่งคอมไพล์และรันโปรเจกต์
เปิด Terminal ที่โฟลเดอร์โปรเจกต์ แล้วรันคำสั่ง:
```bash
mvn clean compile
mvn spring-boot:run
```
*(เมื่อระบบเริ่มทำงานสำเร็จ สังเกตที่บรรทัดสุดท้ายจะขึ้นว่า `Netty started on port 8080`)*

### 3) ทดสอบเปิดผ่านเว็บเบราว์เซอร์
เปิดเบราว์เซอร์แล้วไปที่:
👉 **`http://localhost:8080/products`**

---

## 👤 8. ข้อมูลผู้จัดทำ

* **ชื่อ-นามสกุล:** นายสิทธิโชค มุขนาค
* **รหัสนักศึกษา:** `673380428-6`
* **กลุ่มเรียน (Section):** 4
* **รายวิชา:** CP353002 หลักการออกแบบและพัฒนาซอฟต์แวร์
* **สถาบัน:** มหาวิทยาลัยขอนแก่น (Khon Kaen University)
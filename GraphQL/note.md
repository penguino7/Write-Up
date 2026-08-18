# 📘 Cẩm Nang Kiến Thức Chi Tiết Về GraphQL Từ Cơ Bản Đến Nâng Cao

Tài liệu này tổng hợp toàn bộ lộ trình kiến thức về GraphQL, sự khác biệt với REST API, cơ chế hoạt động thực tế của Client-Server, và các công cụ kiểm thử/bảo mật phổ biến hiện nay.

---

## 📌 Mục Lục Dễ Tra Cứu
* [1. Tổng Quan Về GraphQL](#1-tổng-quan-về-graphql)
* [2. Bốn Thành Phần Cốt Lõi Trong Hệ Thống](#2-bốn-thành-phần-cốt-lõi-trong-hệ-thống)
* [3. Cách GraphQL Hoạt Động Khi Gọi API Thực Tế](#3-cách-graphql-hoạt-động-khi-gọi-api-thực-tế)
* [4. Cơ Chế Lọc Dữ Liệu Theo Schema Của Server](#4-cơ-chế-lọc-dữ-liệu-theo-schema-của-server)
* [5. Bản Chất Kỹ Thuật: GraphQL Là Một Chuẩn Spec](#5-bản-chất-kỹ-thuật-graphql-là-một-chuẩn-spec)
* [6. So Sánh Chi Tiết: GraphQL vs REST API](#6-so-sánh-chi-tiết-graphql-vs-rest-api)
* [7. Công Cụ Nhận Diện Dấu Vết Hệ Thống: graphw00f](#7-công-cụ-nhận-diện-dấu-vết-hệ-thống-graphw00f)
* [8. Công Cụ Trực Quan Hóa API: GraphiQL IDE](#8-công-cụ-trực-quan-hóa-api-graphiql-ide)
* [9. Kiến Thức Introspection Trong GraphQL](#9-kiến-thức-introspection-trong-graphql)

---

## 1. Tổng Quan Về GraphQL
**GraphQL** là một ngôn ngữ truy vấn API (API query language) và là một môi trường thực thi phía máy chủ (runtime) để phản hồi các truy vấn đó. Được Facebook phát triển nội bộ vào năm 2012 và chính thức mã nguồn mở vào năm 2015, GraphQL sinh ra nhằm giải quyết triệt để những hạn chế cố hữu của kiến trúc REST API truyền thống.

*   **Tư duy cốt lõi:** Cho phép phía Client (Trình duyệt/Mobile) tự quyết định và yêu cầu chính xác những trường dữ liệu mình cần hiển thị, server sẽ trả về đúng bấy nhiêu chữ, không thừa và không thiếu.
*   **Cửa ngõ kết nối:** Thay vì phân chia thành hàng chục đường dẫn (endpoints) như REST, hệ thống GraphQL gom tất cả các thao tác giao tiếp dữ liệu vào **duy nhất một endpoint cố định** (thường là `/graphql`).

[⬆ Quay lại mục lục](#-mục-lục-dễ-tra-cứu)

---

## 2. Bốn Thành Phần Cốt Lõi Trong Hệ Thống
Để vận hành cấu trúc GraphQL, hệ thống bắt buộc phải xây dựng dựa trên 4 thành phần nền tảng:

### Schema (Lược đồ)
Đóng vai trò như một bản hợp đồng tối cao giữa Front-end và Back-end. Schema sử dụng ngôn ngữ định nghĩa SDL (Schema Definition Language) để quy định chặt chẽ các kiểu dữ liệu (Types) và mối quan hệ giữa chúng.
```graphql
type User {
  id: ID!
  name: String!
  posts: [Post!]!
}
```

### Query (Truy vấn)
Sử dụng khi Client muốn **Đọc / Lấy dữ liệu** từ Server (Tương đương phương thức `GET` trong REST). Cấu trúc của câu lệnh Query gửi đi sẽ định hình chính xác cấu trúc của gói tin JSON nhận về.

### Mutation (Đột biến)
Sử dụng khi Client muốn **Thay đổi dữ liệu** trên Server, bao gồm các hành động: Thêm mới, Sửa đổi hoặc Xóa bỏ (Tương đương `POST`, `PUT`, `DELETE` trong REST).

### Subscription (Đăng ký)
Xử lý dữ liệu thời gian thực (**Real-time data**). Client thiết lập kết nối lâu dài (WebSockets) tới server để nhận thông báo tự động ngay khi hệ thống có sự kiện hoặc biến động dữ liệu mới.

[⬆ Quay lại mục lục](#-mục-lục-dễ-tra-cứu)

---

## 3. Cách GraphQL Hoạt Động Khi Gọi API Thực Tế
Nhiều người thường nhầm lẫn việc thay đổi trang giao diện trên trình duyệt (như chuyển từ `/profile` sang `/traicay`) sẽ làm thay đổi đường dẫn API. Thực tế, GraphQL hoạt động độc lập:

1.  **URL gọi API luôn cố định:** `POST https://cuahangcuaban.com`
2.  **Hành động quyết định bởi Body:** Khác với REST dùng URL để định danh tài nguyên, GraphQL mở phần Request Body của gói tin `POST` để đọc chuỗi câu lệnh (String) gửi lên.

### Ví dụ về một đoạn mã JavaScript (`fetch`) thực tế:
```javascript
const urlAPI = "https://cuahangcuaban.com";

const cauHoiGraphQL = {
  query: `
    query {
      traicay(ten: "chuoi") {
        mau_sac
        gia_tien
      }
    }
  `
};

fetch(urlAPI, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(cauHoiGraphQL)
})
.then(res => res.json())
.then(result => console.log(result.data.traicay.mau_sac)); // Kết quả: Vàng
```

[⬆ Quay lại mục lục](#-mục-lục-dễ-tra-cứu)

---

## 4. Cơ Chế Lọc Dữ Liệu Theo Schema Của Server
Khi Client gửi Query yêu cầu các trường cụ thể, Server GraphQL thực hiện hành động "lọc" thông minh thông qua các hàm xử lý gọi là **Resolver (Hàm phân giải)**.

*   **Không trực tiếp lưu trữ:** GraphQL không chứa dữ liệu. Dữ liệu vẫn nằm ở SQL, NoSQL hoặc các REST API cũ.
*   **Lọc ở tầng kết quả (Object Filtering):** Để tái sử dụng code, Back-end thường viết một hàm lấy toàn bộ dữ liệu User (ví dụ gồm 50 cột từ Database). Tầng GraphQL sẽ đứng ra nhặt đúng các trường Client đòi hỏi (như `name`, `email`) rồi vứt bỏ các trường nhạy cảm như `password_hash` trước khi chuyển thành JSON truyền qua mạng.
*   **Lọc ở tầng kích hoạt (Resolver Tree):** Nếu Client không yêu cầu các trường lồng nhau phức tạp (ví dụ bỏ qua danh sách `posts` của User), GraphQL sẽ **tắt hoàn toàn nhánh lệnh** tìm kiếm liên quan đến bảng Posts. Hệ thống CSDL không cần thực hiện các lệnh `JOIN` lãng phí, giúp tối ưu hiệu năng toàn diện.

[⬆ Quay lại mục lục](#-mục-lục-dễ-tra-cứu)

---

## 5. Bản Chất Kỹ Thuật: GraphQL Là Một Chuẩn Spec
Cần khẳng định **GraphQL là một Chuẩn Đặc Tả (Specification)** chứ không phải là một phần mềm hay một đoạn code cụ thể. Nó là bộ quy tắc do Facebook viết ra nằm trên giấy để quy định cấu trúc cú pháp gửi/nhận dữ liệu.

Để áp dụng vào thực tế, các cộng đồng lập trình viên dựa vào bản Spec này để xây dựng ra các **Thư viện (Implementations)** trên các ngôn ngữ khác nhau:
*   **NodeJS (JavaScript):** Dùng `Apollo Server` hoặc `graphql-js`.
*   **Python:** Dùng thư viện `Graphene` hoặc `Ariadne`.
*   **Go (Golang):** Dùng `gqlgen`.

> **Hình ảnh ẩn dụ:** GraphQL giống như Luật Giao Thông (Quy định chung), còn các thư viện như Apollo hay Graphene giống như các hãng sản xuất xe (Honda, Toyota). Mỗi hãng chế tạo bằng vật liệu khác nhau nhưng khi lăn bánh đều phải tuân thủ đúng một Luật giao thông chung.

[⬆ Quay lại mục lục](#-mục-lục-dễ-tra-cứu)

---

## 6. So Sánh Chi Tiết: GraphQL vs REST API

### Điểm mạnh của GraphQL so với REST:
*   **Giải quyết Over-fetching và Under-fetching:** Không bị tải thừa dữ liệu không dùng tới, cũng không phải tạo chuỗi gọi mạng liên tục (Request Waterfall) để gom đủ thông tin. Gom tất cả trong 1 Request.
*   **Không cần phân chia phiên bản (No Versioning):** Khi hệ thống thay đổi, Back-end chỉ cần thêm trường mới vào Schema. Ứng dụng cũ dùng Query cũ vẫn chạy bình thường mà không cần đẻ ra các URL kiểu `/api/v1/`, `/api/v2/`.
*   **Tự động sinh tài liệu (Self-documenting):** Hệ thống tự đọc cấu trúc Schema để tạo ra tài liệu tra cứu trực quan mà không cần lập trình viên phải viết tay.

### Điểm yếu của GraphQL so với REST:
*   **Bộ nhớ đệm (Caching) phức tạp:** REST API dễ dàng cache nhờ URL và CDN. GraphQL dùng chung một endpoint `POST` nên trình duyệt và hệ thống mạng không thể tự động cache dữ liệu, buộc phải cấu hình cache phức tạp ở tầng ứng dụng Client.
*   **Nguy cơ tấn công từ chối dịch vụ (DoS):** Nếu Client cố tình gửi câu truy vấn lồng nhau quá sâu (Deeply nested query), Server có thể bị quá tải dữ liệu dẫn đến treo hoặc sập hệ thống.

[⬆ Quay lại mục lục](#-mục-lục-dễ-tra-cứu)

---

## 7. Công Cụ Nhận Diện Dấu Vết Hệ Thống: graphw00f
**graphw00f** là một công cụ bảo mật mã nguồn mở viết bằng **Python**, chuyên dùng để nhận diện dấu vết kỹ thuật (Fingerprinting) của một Endpoint GraphQL ngầm.

### Cơ chế hoạt động vượt mặt mã nguồn của Dev:
Nhiều người thắc mắc vì sao công cụ không biết dữ liệu kinh doanh của Dev viết (như trái cây, sản phẩm) mà vẫn quét được hệ thống. Lý do là `graphw00f` đánh vào:
1.  **Các trường hệ thống mặc định (Meta-fields):** Gửi các câu query chứa trường bắt buộc theo chuẩn như `__typename` để ép Server phản hồi.
2.  **Phân tích cấu trúc thông báo lỗi (Error Fingerprinting):** Công cụ cố tình gửi các câu lệnh sai cú pháp. Lập trình viên không tự viết code xử lý lỗi này mà do các thư viện lõi (Core Engine) đảm nhiệm. Mỗi thư viện (Apollo của NodeJS, Graphene của Python) lại có một "phong cách" định dạng thông báo lỗi JSON độc nhất. `graphw00f` đối chiếu cấu trúc chuỗi lỗi này với cơ sở dữ liệu để chỉ đích danh công nghệ chạy phía sau.

[⬆ Quay lại mục lục](#-mục-lục-dễ-tra-cứu)

---

## 8. Công Cụ Trực Quan Hóa API: GraphiQL IDE
**GraphiQL** là một giao diện lập trình trực quan (IDE) chạy ngay trên trình duyệt web, đóng vai trò như một môi trường thử nghiệm giúp lập trình viên viết, kiểm tra và thực thi các câu lệnh GraphQL. Nó tương tự như công cụ Postman của REST API nhưng tối ưu riêng cho GraphQL.

### Đặc điểm nổi bật:
*   **IntelliSense:** Tự động gợi ý tên các trường dữ liệu khi đang gõ.
*   **Live Linting:** Gạch chân cảnh báo lỗi cú pháp (thiếu ngoặc, sai kiểu dữ liệu) theo thời gian thực trước khi bấm gửi lệnh.
*   **Documentation Explorer:** Cột tra cứu tài liệu tự động xuất hiện ở bên phải màn hình, giúp lập trình viên Front-end click xem cấu trúc toàn bộ cơ sở dữ liệu mà không cần đọc tài liệu viết tay.
*   **Sử dụng linh hoạt:** Có thể cài đặt trực tiếp vào server nội bộ hoặc sử dụng các nền tảng chạy **Online hoàn toàn** (như trang Demo trực tuyến, Apollo Sandbox, Postman Web) kết nối tới các API mở công khai trên internet để thực hành mà không cần tải về máy.

⚠️ **Lưu ý bảo mật:** Do tính tiện lợi của GraphiQL, quy tắc bảo mật bắt buộc là chỉ được kích hoạt công cụ này ở môi trường phát triển (Development) và phải cấu hình tắt hoàn toàn khi triển khai hệ thống lên môi trường thực tế (Production) để tránh bị hacker dò quét cấu trúc Database.

[⬆ Quay lại mục lục](#-mục-lục-dễ-tra-cứu)

---

## 9. Kiến Thức Introspection Trong GraphQL
**Introspection** là cơ chế cho phép Client hỏi ngược lại Server GraphQL về chính cấu trúc Schema của nó. Nói đơn giản, thay vì phải có tài liệu API viết tay, Client có thể gửi một query đặc biệt để biết hệ thống đang có những `type`, `query`, `mutation`, `field`, `argument` và kiểu dữ liệu nào.

### Mục đích chính
*   **Tự sinh tài liệu API:** Các công cụ như GraphiQL, Apollo Sandbox hoặc Postman dùng Introspection để dựng phần Documentation Explorer.
*   **Gợi ý khi viết query:** Khi IDE biết Schema có những trường nào, nó mới có thể autocomplete, kiểm tra sai tên field hoặc cảnh báo sai kiểu dữ liệu.
*   **Hỗ trợ kiểm thử và tích hợp:** Front-end hoặc công cụ test có thể đọc Schema để hiểu endpoint hỗ trợ những thao tác gì mà không cần xem mã nguồn Back-end.

### Các field hệ thống quan trọng
GraphQL định nghĩa một số meta-field bắt đầu bằng dấu `__` để phục vụ Introspection:

*   **`__schema`:** Trả về thông tin tổng quan về toàn bộ Schema, bao gồm danh sách type, query root, mutation root và subscription root.
*   **`__type(name: "...")`:** Trả về chi tiết của một type cụ thể, ví dụ các field của `User`, `Product`, `Order`.
*   **`__typename`:** Trả về tên type thực tế của object đang được query, thường dùng khi làm việc với interface, union hoặc debug response.

### Ví dụ query Introspection cơ bản
```graphql
query {
  __schema {
    queryType {
      name
    }
    mutationType {
      name
    }
    types {
      name
      kind
    }
  }
}
```

Query trên giúp Client nhìn thấy danh sách kiểu dữ liệu trong hệ thống. Nếu endpoint bật Introspection, Server sẽ trả về một JSON mô tả Schema thay vì dữ liệu nghiệp vụ như danh sách user hay sản phẩm.

### Query Introspection tổng quát để dump Schema
Khi kiểm thử bảo mật hợp lệ, việc biết toàn bộ query, mutation, field và type mà Back-end hỗ trợ giúp ta hiểu rõ bề mặt tấn công có thể kiểm tra. Từ đó, tester có thể nhận diện những chức năng nhạy cảm cần rà soát kỹ hơn, ví dụ field chứa thông tin cá nhân, mutation thay đổi dữ liệu, hoặc resolver có khả năng thiếu kiểm tra phân quyền.

Đoạn query tổng quát dưới đây thường được dùng để dump gần như toàn bộ thông tin Schema mà cơ chế Introspection cho phép trả về:

```graphql
query IntrospectionQuery {
  __schema {
    queryType { name }
    mutationType { name }
    subscriptionType { name }
    types {
      ...FullType
    }
    directives {
      name
      description
      locations
      args {
        ...InputValue
      }
    }
  }
}

fragment FullType on __Type {
  kind
  name
  description
  fields(includeDeprecated: true) {
    name
    description
    args {
      ...InputValue
    }
    type {
      ...TypeRef
    }
    isDeprecated
    deprecationReason
  }
  inputFields {
    ...InputValue
  }
  interfaces {
    ...TypeRef
  }
  enumValues(includeDeprecated: true) {
    name
    description
    isDeprecated
    deprecationReason
  }
  possibleTypes {
    ...TypeRef
  }
}

fragment InputValue on __InputValue {
  name
  description
  type { ...TypeRef }
  defaultValue
}

fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
              }
            }
          }
        }
      }
    }
  }
}
```

Kết quả trả về thường khá dài, nhưng rất hữu ích để dựng lại tài liệu API, import vào công cụ kiểm thử hoặc tìm nhanh các query/mutation đáng chú ý trong quá trình đánh giá bảo mật.

### Xem Schema trực quan bằng GraphQL Voyager
**GraphQL Voyager** là công cụ giúp biểu diễn Schema GraphQL thành một sơ đồ tương tác dạng graph. Thay vì chỉ đọc JSON dài từ Introspection, ta có thể nhìn thấy các `type`, `field`, quan hệ object, enum, input và root query/mutation theo cách trực quan hơn.

Link công cụ:

```text
https://apis.guru/graphql-voyager/
```

### Quy trình dùng Postman để gửi request và đưa vào Voyager
1.  **Tạo request GraphQL trong Postman:**
    *   Method thường dùng: `POST`.
    *   URL: endpoint GraphQL, ví dụ `https://target.com/graphql`.
    *   Headers:
        ```http
        Content-Type: application/json
        Authorization: Bearer <token-neu-can>
        ```

2.  **Gửi Introspection Query:**
    *   Trong Postman, chọn Body dạng GraphQL nếu có.
    *   Hoặc dùng Body raw JSON:
        ```json
        {
          "query": "query { __schema { queryType { name } types { name kind } } }"
        }
        ```
    *   Với trường hợp cần dump đầy đủ Schema, dùng query `IntrospectionQuery` tổng quát ở phần trên.

3.  **Lưu response JSON:**
    *   Nếu server bật Introspection, response sẽ có dạng:
        ```json
        {
          "data": {
            "__schema": {
              "queryType": {
                "name": "Query"
              }
            }
          }
        }
        ```
    *   Copy toàn bộ response hoặc lưu ra file `.json`.

4.  **Mở GraphQL Voyager:**
    *   Truy cập `https://apis.guru/graphql-voyager/`.
    *   Chọn tùy chọn nhập Schema/Introspection nếu giao diện hỗ trợ.
    *   Dán JSON introspection lấy từ Postman hoặc trỏ trực tiếp tới endpoint GraphQL nếu endpoint cho phép truy cập từ trình duyệt.

5.  **Phân tích sơ đồ:**
    *   Bắt đầu từ root `Query`, `Mutation`, `Subscription`.
    *   Tìm các object chứa dữ liệu nhạy cảm như `User`, `Account`, `Order`, `Payment`, `Token`, `Admin`.
    *   Kiểm tra các mutation có tác động mạnh như tạo, sửa, xóa, reset, import/export dữ liệu.
    *   Ghi lại field/argument đáng chú ý để kiểm thử phân quyền trong Postman.

> **Lưu ý thực tế:** Nếu Voyager không gọi trực tiếp được endpoint do CORS, authentication hoặc network policy, hãy dùng Postman gửi Introspection Query trước, sau đó copy response JSON sang Voyager. Cách này ổn định hơn khi kiểm thử các API cần token.

### Ví dụ xem chi tiết một type
```graphql
query {
  __type(name: "User") {
    name
    kind
    fields {
      name
      type {
        name
        kind
      }
    }
  }
}
```

Nếu Schema có type `User`, câu query này sẽ cho biết `User` có những field nào, mỗi field thuộc kiểu dữ liệu gì, có phải object, scalar, enum hay list hay không.

### Góc nhìn bảo mật
Introspection rất tiện trong môi trường phát triển, nhưng có thể trở thành rủi ro nếu bật công khai ở môi trường Production:

*   **Lộ bản đồ API:** Kẻ tấn công có thể nhanh chóng biết hệ thống có các query/mutation nhạy cảm nào như `adminUsers`, `resetPassword`, `createOrder`, `deleteUser`.
*   **Tăng tốc quá trình dò lỗi:** Khi đã biết tên field, argument và type, việc thử payload sai, dò phân quyền hoặc tìm điểm thiếu kiểm tra quyền sẽ dễ hơn.
*   **Không phải lỗ hổng tự thân:** Introspection chỉ tiết lộ cấu trúc. Lỗ hổng thật sự thường nằm ở resolver thiếu kiểm tra quyền, mutation nguy hiểm, rate limit yếu hoặc validation chưa chặt.

### Khuyến nghị khi triển khai
*   **Development/Staging:** Có thể bật Introspection để lập trình viên dễ debug, viết query và đọc tài liệu tự động.
*   **Production:** Nên tắt hoặc giới hạn Introspection theo quyền truy cập, ví dụ chỉ cho admin, internal network hoặc token đặc biệt.
*   **Không dựa vào việc tắt Introspection như lớp bảo mật duy nhất:** Dù tắt Introspection, attacker vẫn có thể dò Schema bằng lỗi trả về, brute-force tên field hoặc quan sát request từ front-end. Resolver vẫn phải kiểm tra xác thực, phân quyền, giới hạn độ sâu query và giới hạn tốc độ gọi API.

[⬆ Quay lại mục lục](#-mục-lục-dễ-tra-cứu)

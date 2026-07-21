---
title: "Event 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Bài thu hoạch “Ngày hội cộng đồng FCAJ”

### Mục Đích Của Sự Kiện

- Phá vỡ rào cản công nghệ & Tối ưu hiệu suất AI
- Hiện thực hóa Enterprise AI (AI cấp doanh nghiệp)
- Tối ưu hạ tầng Cloud nền tảng
- Truyền cảm hứng và Kết nối cộng đồng Công nghệ

### Danh Sách Diễn Giả

- **Anh Tịnh** - Build second brain
- **Hải Anh** - Friendly AI Assistant w/ Amazon Quick
- **Thịnh** - CloudFront as Your Foundation
- **Team VIB** - 36 hrs with LotusHacks
- **Đào Đức** - Non-Determinism of "Deterministic" LLM Settings
- **Cát Vy** - Enterprise-Grade Multi-Agent System

### Nội Dung Nổi Bật
Tính bất định của LLM
Hiểu rằng temperature = 0 không đảm bảo kết quả luôn giống nhau do giới hạn của phần cứng GPU và cơ chế Inference Batching.
Nhận thức được AI mang tính xác suất, cần thiết kế hệ thống để xử lý sự biến động của kết quả thay vì kỳ vọng đầu ra tuyệt đối.
Enterprise Multi-Agent System
Tìm hiểu mô hình Multi-Agent với nhiều Agent chuyên biệt phối hợp dưới sự điều phối của Manager Agent.
Hiểu kiến trúc triển khai trên AWS gồm Bedrock AgentCore, Lambda, API Gateway, Docker và Amazon ECR.
Amazon CloudFront
Hiểu cách CloudFront tối ưu hiệu năng bằng Edge Locations, HTTP/3, Compression và Persistent Connections.
Biết các kỹ thuật bảo mật như Origin Access Control (OAC) và Origin Cloaking để bảo vệ hệ thống.
Context Engineering
Hiểu rằng chất lượng ngữ cảnh quan trọng hơn số lượng dữ liệu đưa vào LLM.
Biết cách xây dựng trợ lý AI dựa trên RAG, Context Management và Memory thay vì chỉ tập trung vào Prompt Engineering.

### Những Gì Học Được
Tư duy thiết kế AI
Thiết kế hệ thống AI theo hướng Probabilistic, chấp nhận tính không xác định của LLM và bổ sung các cơ chế kiểm tra như Majority Voting hoặc Structured Output.
Chuyển từ tư duy Prompt Engineering sang Context Engineering, kết hợp lưu trữ, truy xuất và quản lý tri thức hiệu quả.
Hiểu rằng một hệ thống AI doanh nghiệp cần được xây dựng đồng thời với các yêu cầu về bảo mật, quản trị dữ liệu và khả năng mở rộng.
Kiến thức về hạ tầng Cloud
Hiểu cách tận dụng AWS Global Backbone và CloudFront để tối ưu hiệu năng và giảm độ trễ.
Biết cách xử lý logic tại Edge bằng CloudFront Functions hoặc Lambda@Edge nhằm giảm tải cho máy chủ gốc.
Hiểu các mô hình triển khai AI hiện đại trên nền tảng AWS theo hướng an toàn và sẵn sàng cho môi trường Production.

### Ứng Dụng Vào Công Việc
Phát triển hệ thống AI
Điều chỉnh tham số LLM phù hợp với từng bài toán thay vì sử dụng mặc định.
Áp dụng Structured Output, Majority Voting và cơ chế kiểm tra kết quả để tăng độ ổn định của AI.
Xây dựng hệ thống Multi-Agent cho các nghiệp vụ phức tạp nhằm nâng cao chất lượng phản hồi.
Tối ưu hạ tầng AWS
Triển khai Amazon CloudFront để giảm tải Origin, tăng tốc truy cập và tối ưu chi phí truyền dữ liệu.
Áp dụng Origin Cloaking, OAC và WAF để tăng cường bảo mật cho ứng dụng.
Sử dụng CloudFront Functions hoặc Lambda@Edge để xử lý các tác vụ nhẹ ngay tại Edge.
Xây dựng trợ lý AI doanh nghiệp
Triển khai mô hình RAG kết hợp Vector Database để cung cấp ngữ cảnh chính xác.
Xây dựng bộ lọc ngữ cảnh nhằm chỉ cung cấp thông tin liên quan cho LLM, giúp tăng độ chính xác và giảm chi phí token.
Ứng dụng các dịch vụ AI trên AWS như Amazon Bedrock, Amazon Q và các Agent để hỗ trợ tự động hóa quy trình phát triển phần mềm và xử lý nghiệp vụ doanh nghiệp.

### Trải nghiệm trong event

ham gia Ngày hội cộng đồng First Cloud AI Journey (FCAJ) là một trải nghiệm ý nghĩa, giúp tôi cập nhật nhiều xu hướng mới về Cloud Computing, Trí tuệ nhân tạo (AI) và kiến trúc phần mềm hiện đại. Chương trình không chỉ mang đến kiến thức chuyên môn mà còn giúp tôi hiểu rõ hơn cách các doanh nghiệp ứng dụng công nghệ AWS để giải quyết các bài toán thực tế.

#### Học hỏi từ các diễn giả có chuyên môn cao
Các diễn giả đến từ AWS và các doanh nghiệp công nghệ đã chia sẻ nhiều kinh nghiệm thực tế trong quá trình xây dựng và vận hành hệ thống Cloud quy mô lớn.
Nội dung trình bày không chỉ tập trung vào lý thuyết mà còn phân tích các case study về Microservices, Domain-Driven Design (DDD), Event-Driven Architecture, Enterprise AI và Application Modernization.
Qua những chia sẻ này, tôi hiểu rõ hơn tư duy thiết kế hệ thống theo hướng Business-first, lấy yêu cầu nghiệp vụ làm trung tâm trước khi lựa chọn công nghệ.

#### Trải nghiệm kỹ thuật thực tế
Được tìm hiểu cách chuyển đổi từ kiến trúc Monolithic sang Microservices nhằm tăng khả năng mở rộng, dễ bảo trì và nâng cao hiệu năng hệ thống.
Hiểu rõ quy trình áp dụng Domain-Driven Design, Event Storming và Bounded Context để phân tích và mô hình hóa nghiệp vụ.
Tiếp cận các mô hình Event-Driven Architecture như Publish/Subscribe, Point-to-Point và Streaming, đồng thời hiểu sự khác biệt giữa giao tiếp đồng bộ và bất đồng bộ trong các hệ thống phân tán.
Cập nhật xu hướng phát triển hạ tầng trên AWS từ EC2, ECS, Fargate đến Lambda và những lợi ích của kiến trúc Serverless.

#### Ứng dụng công cụ hiện đại
Trực tiếp tìm hiểu về **Amazon Q Developer**, công cụ AI hỗ trợ SDLC từ lập kế hoạch đến maintenance.
Học cách **tự động hóa code transformation** và pilot serverless với **AWS Lambda**, từ đó nâng cao năng suất phát triển.

#### Trải nghiệm các giải pháp AI và AWS
Được giới thiệu về Amazon Q Developer, công cụ AI hỗ trợ lập trình trong toàn bộ vòng đời phát triển phần mềm từ lập kế hoạch, viết mã, kiểm thử đến bảo trì.
Tìm hiểu các giải pháp Enterprise AI, Multi-Agent System và Context Engineering, giúp xây dựng các ứng dụng AI có khả năng mở rộng, quản trị và vận hành trong môi trường doanh nghiệp.
Hiểu thêm về vai trò của Amazon CloudFront trong việc tối ưu hiệu năng, giảm tải hệ thống và tăng cường bảo mật thông qua các dịch vụ Edge của AWS.
#### Bài học rút ra
- Thiết kế hệ thống cần bắt đầu từ bài toán nghiệp vụ và lựa chọn kiến trúc phù hợp thay vì chạy theo công nghệ.
Việc áp dụng Domain-Driven Design và Event-Driven Architecture giúp xây dựng hệ thống linh hoạt, dễ mở rộng và có khả năng chịu lỗi cao.
Serverless và các dịch vụ AI trên AWS giúp tối ưu chi phí vận hành, tăng hiệu quả phát triển và rút ngắn thời gian triển khai sản phẩm.
Chương trình giúp tôi có thêm góc nhìn thực tiễn về cách doanh nghiệp hiện đại hóa ứng dụng, đồng thời định hướng rõ hơn lộ trình học tập và ứng dụng các công nghệ AWS vào các dự án trong tương lai.



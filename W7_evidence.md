# W7 Evidence Pack — Group 10, MedEdu (EduTech: AI Study Buddy)

> **Hackathon:** W7 Capstone — Ship Production-Ready AI in 48 Hours

---

## 1. Cover

| Trường | Giá trị |
|--------|---------|
| **Nhóm** | G10 |
| **Domain** | EduTech: "AI Study Buddy" — MedEdu |
| **Thành viên** | Lê Trần Tuấn Khanh, Trần Mạnh Trường, Trần Mạnh Cường, Nguyễn Đức Hảo, Lê Văn Hải, Phan Đức Huy, Lê Viết Quốc Hưng, Huỳnh Xuân Hậu, Nguyễn Thị Mến, Trần Quốc Hùng |
| **Live URL** | https://aws.hungtran.id.vn/ |
| **Repo** | https://github.com/orgs/aws-g10/repositories |
| **AWS Account** | 493499579600 |
| **Region** | ap-southeast-2 (Sydney) |
| **Tổng chi phí (tính đến hiện tại)** | $[—] — chụp ảnh Cost Explorer cuối Day 1 để điền |
| **Pre-flight safety** | MFA trên root | Budget alert $80 | Cost Anomaly Detection  | Gắn tag | Bedrock access đã request |


---

## 2. Pitch and Vision

### Use Case

**MedEdu** (tên mã: EduMate AI) là nền tảng học tập thông minh dành cho sinh viên y khoa và nhân viên y tế. Giá trị cốt lõi: tải lên bất kỳ file PDF giáo trình y khoa nào và nhận lại — trong vài giây — một bản tóm tắt có cấu trúc, bộ flashcard cá nhân hóa, bài trắc nghiệm, và chatbot hỏi đáp AI có trích dẫn nguồn từ chính tài liệu bạn đã tải lên.

### Target User

- **Sinh viên Y khoa / Dược sĩ** tại các trường đại học đang ôn thi
- **Nhân viên y tế** cần tra cứu nhanh thông tin từ tài liệu lâm sàng
- **Giảng viên** cần tạo tài liệu kiểm tra từ tài liệu khóa học
- **Nhà nghiên cứu** cần tóm tắt các bài báo khoa học

### Why This Domain Matters

Giáo dục y khoa đòi hỏi hấp thụ hàng nghìn trang tài liệu dày đặc thuật ngữ mỗi học kỳ. Sinh viên đối mặt với:

- **Quá tải thông tin**: không thể đọc lại toàn bộ giáo trình trước kỳ thi
- **Thiếu công cụ học tập cá nhân hóa**: phương pháp truyền thống không phù hợp với nhu cầu từng người
- **Khó tra cứu** thông tin cụ thể trong các file PDF hàng trăm trang
- **Chi phí tài liệu cao**: sách giáo khoa y khoa có giá thành đắt đỏ

MedEdu giải quyết bằng cách tự động hóa quy trình tạo tài liệu học tập và đảm bảo mọi câu trả lời AI đều được truy vấn từ nội dung tài liệu thực tế của người dùng — không ảo giác, mọi câu trả lời đều có thể truy vết đến trang cụ thể.

### Real-World Parallel

MedEdu tương tự trực tiếp với **Quizlet AI** (tạo flashcard tự động), **Google NotebookLM** (hỏi đáp dựa trên tài liệu), và **Khan Academy's Khanmigo** (trợ lý học tập AI cá nhân hóa). Một kỹ sư Khanmigo xem xét hệ thống của chúng tôi sẽ ngay lập tức hỏi: *"Làm thế nào để RAG của bạn thực sự truy xuất đúng chunks, và các bạn đo lường chất lượng truy xuất như thế nào?"* Chúng tôi giải quyết vấn đề này trong Phần 6.5.

---

## 3. Architecture

### 3.1 Sơ đồ kiến trúc

<img width="2792" height="1841" alt="Architect_Hackathon_G10 drawio" src="https://github.com/user-attachments/assets/5c4ddf8a-4be0-4862-b362-24fbcc8a1460" />


> **Link to diagram: ** [https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=B%E1%BA%A3n%20sao%20c%E1%BB%A7a%20AWS-2.drawio&page-id=aTyvGL_KTTjEhPqx1fEJ&dark=auto#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1uAov8ZokNK1LBo_zqMDtdrT4d8BFUOMf%26export%3Ddownload](https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=B%E1%BA%A3n%20sao%20c%E1%BB%A7a%20AWS-2.drawio&page-id=aTyvGL_KTTjEhPqx1fEJ&dark=auto#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1uAov8ZokNK1LBo_zqMDtdrT4d8BFUOMf%26export%3Ddownload)

### 3.2 Bảng Service Decisions

| # | Capability | Service Đã Chọn | Tại Sao Chọn Cái Này, Không Phải Cái Khác |
|---|-----------|-----------------|-------------------------------------------|
| 1a | Static UI Hosting | CloudFront + S3 | HTTPS trên custom domain `aws.hungtran.id.vn` qua CloudFront. S3 là origin. Block Public Access bật. Không cần ACM riêng vì dùng HTTPS của CloudFront. |
| 1b | API Entry | ALB (Application Load Balancer) | ALB nhận traffic từ CloudFront prefix list và forward đến ECS Fargate tasks qua port 8000. Stateful app (FastAPI + SQLAlchemy) cần connection persistence — ALB hỗ trợ tốt hơn Lambda Function URL. |
| 2 | Application Compute | ECS Fargate | FastAPI backend chạy trên Fargate thay vì EC2 vì: không cần quản lý instance, tự động scale theo request, tính phí theo vCPU-Giây sử dụng (phù hợp cho hackathon 48h). ECS IAM Task Role cho phép gán quyền scoped. |
| 3 | AI/ML Feature | Bedrock Agents + Knowledge Base + Guardrail | 3 Agents riêng cho 3 chức năng (chat RAG, quiz, flashcard) mỗi agent gắn Knowledge Base. Chat RAG dùng **Nova Lite** (amazon.nova-lite-v1:0) cho Q&A ngắn gọn, chi phí thấp nhất trong Bedrock family. Quiz và Flashcard Generator dùng **Claude Haiku 4.5** (au.anthropic.claude-haiku-4-5) cho structured generation (MCQ format, flashcard pairs) — Haiku đủ năng lực cho structured extraction từ retrieved chunks với chi phí rẻ hơn Sonnet. Guardrail bảo vệ nội dung phù hợp cho môi trường giáo dục y khoa. |
| 4 | Data Persistence | RDS PostgreSQL db.t3.micro (single-AZ) | Data model MedEdu là quan hệ (users → books → contents → quizzes → flashcards). SQLAlchemy ORM đã xây dựng sẵn. Single-AZ vì Multi-AZ gấp đôi chi phí, không có giá trị demo. |
| 5 | Object Storage | S3 Standard (3 buckets) | Frontend bucket, data-source bucket (KB), supplemental bucket. Tất cả bật Block Public Access và SSE-S3 encryption. OwnershipControls: BucketOwnerEnforced. |
| 6 | Network Foundation | VPC 3-tier (public + private DB) + NAT GW + SG references | Public subnets cho ALB + ECS + NAT GW. Private subnets (2 AZ) cho RDS . SG của RDS chỉ accept TCP 5432 từ ECS SG (không phải 0.0.0.0/0). VPC Flow Logs gửi logs đến CloudWatch. |
| 7 | Identity & Access | IAM execution roles (Bedrock Agents + ECS Task Role) | Mỗi Bedrock Agent có dedicated IAM role với chỉ các actions cần thiết. ECS Task Role chỉ được phép: S3 trên 3 bucket cụ thể, RDS connect, Bedrock invoke. Không wildcard. CloudTrail ghi management events. |
| — | Optional #8 / #9 / #10 | Full Observability + Advanced Cost Insights + Advanced Security | **#8 Full Observability:** VPC Flow Logs (ALL traffic → CloudWatch Log Group `webapp-group10-vpc-flow-log`) + CloudTrail multi-region (`webapp-group10-management-events` → S3 bucket + CloudWatch Log Group `/cloudtrail/group10`) + AWS Config auto-fix S3 (`Config-auto-fix-for-s3` giám sát public access). **#9 Advanced Cost Insights:** Cost Explorer filter `Team=G10` + Budget Alert $80 + Cost Anomaly Detection. **#10 Advanced Security:** Bedrock Guardrail `webapp-group10-guardrail` (content filters VIOLENCE/HATE/SEXUAL/INSULTS/MISCONDUCT HIGH + profanity BLOCK + contextual grounding 0.85/0.65) gắn cả 3 agents; RDS encrypted với KMS CMK `arn:aws:kms:ap-southeast-2:493499579600:key/281a9f1a-d25f-4330-a264-c5a5565caea4`; CloudTrail enable log file validation; DeletionProtection=true trên RDS. |

### 3.3 Trade-offs (2-3 quyết định có suy nghĩ đã được ghi nhận)

### **Trade-off 1: ECS Fargate vs EC2 cho Application Compute**

Bọn em quyết định chọn **ECS Fargate** thay vì EC2 truyền thống vì backend FastAPI đã được container hóa hoàn chỉnh. Với Fargate, bọn em không cần tốn thời gian quản lý hạ tầng (không phải lo SSH, patching hay cấu hình Auto Scaling Groups), giúp cả đội tập trung tối đa vào việc phát triển tính năng trong 48h ngắn ngủi.

* **Lợi ích:** Tính phí linh hoạt theo resource thực tế sử dụng (0.04048 USD/vCPU-giờ tại ap-southeast-2), rất tối ưu cho quy mô hackathon.
* **Đánh đổi:** Fargate có độ trễ cold start cao hơn EC2 nếu task bị terminate, nhưng bọn em đã xử lý bằng cách cấu hình ECS Service để giữ task luôn ở trạng thái **Running**, đảm bảo trải nghiệm người dùng không bị gián đoạn.

---

### **Trade-off 2: Chiến lược chọn Model theo tác vụ (Nova Lite & Haiku)**

Thay vì dùng một model duy nhất cho mọi tính năng, bọn em triển khai **chiến lược Per-agent Model** để tối ưu hóa giữa hiệu năng và chi phí:

* **Nova Lite cho RAG Chat:** RAG chat là tác vụ light, chỉ cần trả lời ngắn từ retrieved chunks. Nova Lite có chi phí thấp nhất trong Bedrock family ($0.04/$0.16 per 1M tokens) — rẻ hơn Haiku 25x ở input. Đủ năng lực cho single-turn Q&A đơn giản.
* **Claude 4.5 Haiku cho Quiz/Flashcard:** Tạo MCQ (4 lựa chọn, đáp án đúng, giải thích) và flashcard pairs (front/back) là structured extraction — Haiku đủ để parse retrieved chunks và output đúng JSON format. Tiết kiệm hơn nhiều so với Sonnet mà vẫn đảm bảo quality.
* **Hiệu quả:** Nova Lite cho chat + Haiku cho quiz/flashcard tối ưu chi phí tối đa mà vẫn đáp ứng yêu cầu structured generation cho medical study tool.

---

### **Trade-off 3: Semantic Chunking vs Fixed-size Chunking**

Đối với Knowledge Base, bọn em ưu tiên sử dụng **Semantic Chunking** (max 300 tokens, breakpoint percentile 95%) thay vì chia nhỏ văn bản theo kích thước cố định (Fixed-size).

* **Lý do:** Các tài liệu y khoa thường có cấu trúc đoạn văn mang tính logic chặt chẽ. Nếu cắt theo số ký tự cứng nhắc, ngữ cảnh sẽ bị gãy vụn, dẫn đến kết quả truy xuất (Retrieval) kém chính xác. Semantic chunking giúp bọn em giữ trọn vẹn ý nghĩa của từng phân đoạn.
* **Đánh đổi:** Phương pháp này xử lý chậm hơn và tốn nhiều token embedding hơn do kích thước chunk không đều, nhưng đây là sự đánh đổi xứng đáng để đổi lấy chất lượng câu trả lời chuyên sâu cho người dùng.
---

## 4. Cost Discipline

### 4.1 Cost Screenshots

<img width="975" height="385" alt="image" src="https://github.com/user-attachments/assets/a0363b0f-362a-4399-a38d-ddeea95f05f5" />
<img width="975" height="249" alt="image" src="https://github.com/user-attachments/assets/1ec8f7a9-493a-4879-87af-9614882f64fc" />

> ** Screenshot 4.1 Day 1 EOD:** 

<img width="1584" height="735" alt="image" src="https://github.com/user-attachments/assets/b52d16a3-d0a7-4daf-a3b5-c5ee397a796b" />
<img width="1250" height="733" alt="image" src="https://github.com/user-attachments/assets/9d82d27c-4f13-4c20-8a7f-1accd0bc94e5" />

> ** Screenshot 4.2 Day 2 EOD:** 

---

### 4.2 Cost Breakdown (Actual Cost from AWS Cost Explorer)

Dựa trên dữ liệu thực tế từ AWS Cost Explorer trong khoảng `2026-05-26 → 2026-05-29` tại account triển khai:

| Service | Actual Cost (USD) | Observation |
|---------|------------------|-------------|
| Bedrock | $32.59 | Chi phí lớn nhất do inference và model invocation |
| EC2-Other | $3.14 | Bao gồm networking và EC2-related overhead |
| AWS Config | $2.53 | Cost từ configuration recording |
| Claude Sonnet 4.6 (Bedrock Edition) | $1.44 | Token inference usage |
| Relational Database Service (RDS) | $1.22 | Database runtime cost |
| Elastic Container Service (ECS) | $0.89 | Container orchestration runtime |
| Elastic Load Balancing (ALB) | $0.78 | Application Load Balancer hourly usage |
| Claude Haiku 4.5 (Bedrock Edition) | $0.67 | Lightweight model inference |
| VPC | $0.59 | Network-related cost |
| OpenSearch Service | $0.56 | Search/index cluster runtime |
| Route 53 | $0.50 | Hosted zone + DNS queries |
| CloudWatch | $0.24 | Logs and monitoring |
| S3 | $0.04 | Object storage |
| Systems Manager | $0.03 | Session/management operations |
| Cost Explorer | $0.03 | Cost analytics API usage |
| Others | ~$0.00 | Payment Cryptography, ECR, Location Service, Secrets Manager |
| **TOTAL ACTUAL COST** | **$45.67** |  |
| **% of $100 Budget Cap** | **45.67%** | Within allowed budget |

> **📌 Observation:**  
> Chi phí thực tế cao hơn nhiều so với estimate ban đầu (~$9.50 → $45.67), chủ yếu do Bedrock inference usage chiếm tỷ trọng lớn.

> **📌 Main Cost Driver:**  
> AWS Bedrock là nguồn chi phí lớn nhất (~71.4% tổng chi phí), cho thấy workload AI inference/token processing đang dominate hệ thống.

> **📌 Optimization Opportunity:**  
> Có thể giảm cost bằng cách:
> - Giảm token output/input
> - Dùng Claude Haiku thay vì Sonnet cho các tác vụ nhẹ
> - Áp dụng caching/vector retrieval để giảm số lần invoke model
> - Scale down Config/OpenSearch nếu không cần realtime monitoring


---

### 4.3 Written Observation

<img width="1250" height="158" alt="image" src="https://github.com/user-attachments/assets/b9904ef0-f89e-41d8-b798-c79d3a8e651b" />

#### Top 3 Highest Cost Services

1. **Bedrock — $32.59 (~71.4%)**  
   Chi phí lớn nhất do AI model inference và token processing.

2. **EC2-Other — $3.14 (~6.9%)**  
   Bao gồm NAT Gateway, networking overhead, data transfer và các VPC infrastructure-related charges.

3. **AWS Config — $2.53 (~5.5%)**  
   Cost phát sinh từ continuous configuration recording và compliance tracking.

---

#### Cost Trend Observation

- **May 26:** Chi phí thấp (~$0.46), hệ thống gần như idle hoặc mới deploy.
- **May 27:** Chi phí tăng lên ~$4.47 khi workload inference bắt đầu chạy.
- **May 28:** Chi phí tăng mạnh lên ~$40.74 do Bedrock model usage spike.
- **May 29:** Chưa phát sinh thêm chi phí tại thời điểm capture screenshot.

---

#### Budget Compliance

- **Within budget:** YES  
  Tổng chi phí hiện tại là `$45.67`, vẫn nằm trong giới hạn ngân sách `$100`.

---

#### Bonus Eligibility (Path H — dưới $30)

- **Not eligible currently**  
  Vì tổng chi phí `$45.67` vượt mức yêu cầu `< $30`.

---

#### Final Technical Insight

Kết quả Cost Explorer cho thấy AI inference workload (AWS Bedrock) là bottleneck chính về cost thay vì infrastructure truyền thống (ECS/RDS/VPC). Điều này phản ánh kiến trúc AI-native thường có compute cost tập trung vào model invocation hơn là container/runtime infrastructure.

---

### 4.4 Cost Anomaly Detection

Cost Anomaly Detection monitor đã tạo ở account level. Service miễn phí, dựa trên ML, set alert cho bất kỳ spike chi phí bất thường nào.

### Monitor Information

| Field | Value |
|---|---|
| Monitor Name | `Webapp-Group10-Services-Monitor` |
| Monitor Type | AWS Services |
| Monitoring Scope | All AWS Services |
| Monitor ARN | `arn:aws:ce::493499579600:anomalymonitor/a94dee3e-055a-465e-bcf8-c1d85935b867` |
| Alert Subscriptions | `FinanceGroup10` |
| Managed By | AWS |

**Ảnh Chụp Bằng Chứng cấu hình:**

<img width="1566" height="733" alt="image" src="https://github.com/user-attachments/assets/81a32633-41d7-4819-b05b-22b53e1f0e94" />
<img width="1540" height="358" alt="image" src="https://github.com/user-attachments/assets/cf27a211-0496-487c-b3dd-e5d8cfc4d6b8" />


**Ảnh Chụp Bằng Chứng Detected anomalies:**

<img width="1547" height="619" alt="image" src="https://github.com/user-attachments/assets/2d9a0cb0-49fe-4e92-9488-a8e8ecdf2636" />


---

## 5. Security

### 5.1 IAM Roles và Execution Role Scope (Mandatory #7)

#### ECS Task Role (Bedrock + S3)

<img width="1565" height="720" alt="image" src="https://github.com/user-attachments/assets/a86b74a3-d744-48a0-9b93-a170b493e520" />

> aiRelatedFeaturePermissions
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "InvokeThreeBedrockAgents",
            "Effect": "Allow",
            "Action": "bedrock:InvokeAgent",
            "Resource": [
                "arn:aws:bedrock:ap-southeast-2:*:agent-alias/ECQAJVXGPT/E67OAJGSL9",
                "arn:aws:bedrock:ap-southeast-2:*:agent-alias/F80FKASMRV/CM5YMEGKWE",
                "arn:aws:bedrock:ap-southeast-2:*:agent-alias/BCOQX4GQEA/7FNZNI5UPO"
            ]
        },
        {
            "Sid": "RetrieveFromKnowledgeBase",
            "Effect": "Allow",
            "Action": [
                "bedrock:Retrieve",
                "bedrock:RetrieveAndGenerate"
            ],
            "Resource": "arn:aws:bedrock:ap-southeast-2:*:knowledge-base/OY2MYERT59"
        },
        {
            "Effect": "Allow",
            "Action": [
                "bedrock:StartIngestionJob",
                "bedrock:GetIngestionJob"
            ],
            "Resource": "arn:aws:bedrock:ap-southeast-2:493499579600:knowledge-base/OY2MYERT59"
        },
        {
            "Sid": "ListBucket",
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::webapp-group10-data-source"
        },
        {
            "Sid": "ReadWriteObjects",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:::webapp-group10-data-source/*"
        }
    ]
}
```
> enablePsExec
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowECSExecSSMMessages",
            "Effect": "Allow",
            "Action": [
                "ssmmessages:CreateControlChannel",
                "ssmmessages:CreateDataChannel",
                "ssmmessages:OpenControlChannel",
                "ssmmessages:OpenDataChannel"
            ],
            "Resource": "*"
        }
    ]
}
```
> putMetric
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CloudWatchPutMetrics",
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricData"
            ],
            "Resource": "*"
        }
    ]
}
```

Các IAM roles quan trọng:

| Role Name | Purpose | Scoped Actions |
|-----------|---------|---------------|
| `AmazonBedrockExecutionRoleForKnowledgeBase` | KB ingestion + S3 Vectors access | S3 (3 buckets), Bedrock invoke Titan, BDA, S3 Vectors (GetIndex, QueryVectors, PutVectors, DeleteVectors) |
| `AmazonBedrockExecutionRoleForAgents_AXKQXIRED1G` | Chat RAG Agent | Nova Lite invoke, ApplyGuardrail, KB retrieve |
| `AmazonBedrockExecutionRoleForAgents_DDB8T2J26LB` | Quiz Agent | Claude Haiku invoke, ApplyGuardrail, KB retrieve |
| `AmazonBedrockExecutionRoleForAgents_G7WDZSSAM3M` | Flashcard Agent | Claude Haiku invoke, ApplyGuardrail, KB retrieve |
| `VPCFlowLogs-Cloudwatch-1777278638503` | VPC Flow Logs delivery | logs:CreateLogStream, PutLogEvents |


#### CloudTrail — Audit Logging


<img width="1587" height="457" alt="image" src="https://github.com/user-attachments/assets/31698280-3c16-462f-be6f-af83614f000c" />
> **📸 Screenshot 5.1b:** Chụp CloudTrail console hiển thị trail đang logging.

- Trail name: `webapp-group10-management-events`
- Multi-region: enabled
- S3 bucket: `webapp-group10-management-cloudtrail-logs-bucket`
- Events: Management events
- Log file validation: enabled

### 5.2 MFA on Root Account

MFA device đã cấu hình trên AWS root account. Root credentials được lưu trữ bảo mật. IAM users đã được tạo cho mỗi thành viên với quyền phù hợp.

<img width="2048" height="735" alt="image" src="https://github.com/user-attachments/assets/77d03892-8bed-478c-9dd1-f75423f6451a" />

> **📸 Screenshot 5.2:** Ảnh MFA đã enabled trên root account.


### 5.3 Optional #10 Security Area — Guardrail + Encryption + CloudTrail (đã implement)

Dựa trên CloudFormation template, nhóm đã implement các security features sau:

#### Bedrock Guardrail — Content Safety

<img width="1423" height="744" alt="image" src="https://github.com/user-attachments/assets/b314ab15-7184-4eb4-a29d-7e63c90e9361" />

> **📸 Screenshot 5.3A:** Bedrock Guardrail console từ AWS Console.


- **Guardrail name:** `webapp-group10-guardrail`
- **Guardrail ID:** `l0yuz39969zy`
- **Content Filters:** VIOLENCE, HATE, SEXUAL, INSULTS, MISCONDUCT (HIGH strength, BLOCK action)
- **Profanity:** Input + Output BLOCK
- **Contextual Grounding:** GROUNDING threshold 0.85, RELEVANCE threshold 0.65
- **Blocked message:** "Sorry, the model cannot answer this question. If you have any concerns, let contact with us at 0123456789"
- Gắn vào cả 3 Bedrock Agents (chat, quiz, flashcard)

#### Encryption at Rest

- **RDS:** Encrypted = true, KMS Key = `arn:aws:kms:ap-southeast-2:493499579600:key/281a9f1a-d25f-4330-a264-c5a5565caea4`
- **S3 Buckets:** SSE-S3 (AES256), BucketKey enabled

<img width="1891" height="733" alt="image" src="https://github.com/user-attachments/assets/92890c84-e3c6-4428-b8f6-238c79279528" />

> **📸 Screenshot 5.3B:**  RDS console hiển thị encryption enabled + KMS key.


#### Audit Trail

- **CloudTrail:** Multi-region management events trail, log file validation enabled, logs stored in dedicated S3 bucket

<img width="1909" height="525" alt="image" src="https://github.com/user-attachments/assets/accece14-b759-48b9-b83d-18a1f4fc8157" />

> **📸 Screenshot 5.3C:** S3 bucket chứa trail logs.


---

## 6. Monitoring

### 6.1 CloudWatch Dashboard

<img width="1879" height="748" alt="image" src="https://github.com/user-attachments/assets/29b9e1fc-4196-4dad-b655-26b43e7eefa4" />

> **Full dashboard:**

**Các widget đã tạo:**

| Dashboard | Widget | Metrics | Mục đích |
|-----------|--------|---------|----------|
| **ECS & ALB Monitoring** | ECS CPU & Memory | ContainerMemoryUtilized, ContainerMemoryUtilization | Theo dõi tài nguyên container |
| **ECS & ALB Monitoring** | ALB Traffic | RequestCount, TargetResponseTime | Giám sát lưu lượng & hiệu năng |
| **Security Monitoring** | ConsoleLoginCount | Số lần đăng nhập AWS Console | Phát hiện đăng nhập bất thường |
| **Security Monitoring** | CloudTrail Alarm Status | Trạng thái alarm | Giám sát sự kiện bảo mật |
| **Security Monitoring** | CloudTrail Log Stream | API events, IP, Region | Audit hoạt động AWS |

---

<img width="1891" height="355" alt="image" src="https://github.com/user-attachments/assets/8d683ac2-2d20-4254-9707-5454fe6680eb" />

> **📸 Screenshot 6.1a:** ECS & ALB Monitoring Dashboard
>
> Dashboard giám sát hiệu năng hệ thống tập trung vào ECS và Application Load Balancer:
>
> **ECS CPU & Memory Monitoring:**
> - Widget theo dõi `ContainerMemoryUtilized` (~97 MiB) và `ContainerMemoryUtilization` (~4-5%)
> - Memory utilization ở mức thấp → container còn nhiều tài nguyên khả dụng, hệ thống ổn định
>
> **ALB Traffic Metrics:**
> - Widget theo dõi `RequestCount` và `TargetResponseTime`
> - Lưu lượng thay đổi theo thời gian với các spike nhỏ
> - Không có độ trễ lớn, backend phản hồi ổn định dù traffic tăng
>
> **Lợi ích:** Xác định peak traffic, theo dõi hiệu năng backend, phát hiện bottleneck sớm.

---

<img width="1900" height="601" alt="image" src="https://github.com/user-attachments/assets/42fe5bd2-7b29-45ae-8c1d-a016888724c7" />

> **📸 Screenshot 6.1b:** Security Monitoring Dashboard
>
> Dashboard giám sát bảo mật sử dụng CloudWatch kết hợp CloudTrail:
>
> **ConsoleLoginCount:**
> - Theo dõi số lần đăng nhập AWS Console
> - Biểu đồ cho thấy spike đăng nhập vào ~08:00 UTC, các thời điểm khác gần như không có hoạt động
> - Hỗ trợ phát hiện đăng nhập bất thường
>
> **CloudTrail Alarm Status:**
> - Trạng thái alarm màu xanh → hoạt động bình thường
> - Chưa phát hiện sự kiện bất thường hoặc vi phạm ngưỡng
>
> **CloudTrail Log Stream:**
> - Ghi nhận các API events: thời gian, action, IP address, region
> - Một số action ghi nhận: `ListContainers`, `DescribeLoadBalancers`, `GetRestApis`, `ListChannelGroups`
>
> **Lợi ích:** Audit hoạt động AWS, giám sát truy cập, phát hiện truy cập trái phép, phục vụ security compliance.


---

### 6.2 CloudWatch Alarm

> **CẦN TẠO (Day 2):** Tạo ít nhất 1 alarm. Alarm phải ở trạng thái OK hoặc ALARM (không phải INSUFFICIENT_DATA).

<img width="1550" height="743" alt="image" src="https://github.com/user-attachments/assets/96b4dc32-97ae-4d4b-816d-d9aad9536e7e" />

```
Alarm Name: CloudTrail-ConsoleLogin-Alarm
Metric: ConsoleLoginCount
Threshold: >= 1 lần đăng nhập trong 1 phút (1 datapoints within 1 minute)
State: OK
Action: SNS → email đến team
```

<img width="1567" height="646" alt="image" src="https://github.com/user-attachments/assets/54478f88-1ab3-4ae7-b7b4-176205304d1f" />

```
Alarm Name: G10-ALB-5XX-Errors
Metric: HTTPCode_Target_5XX_Count
Threshold: > 0 lỗi trong chu kỳ 5 phút (`> 0 for 1 datapoints within 5 minutes`)
State: OK
Action: SNS → email đến team
```
---

### 6.3 Log Insights Query


**Ví dụ query (CloudTrail Logs – Audit AWS Activities):**

```
fields @timestamp,
       eventName,
       sourceIPAddress,
       awsRegion,
       eventSource,
       userIdentity.userName
| filter eventName not like /Describe/
| filter eventName not like /Get/
| filter eventName not like /List/
| sort @timestamp desc
| limit 20
```

<img width="1920" height="891" alt="image" src="https://github.com/user-attachments/assets/a8f4b3c8-18f5-493a-b911-e2461054bbb1" />
> **📸 Screenshot 6.3:** Ảnh kết quả thực hiện query.


---

## 6.5 Measurement & Decisions 

### DECISION BLOCK 1: Foundation Model — Per-Agent Model Selection (Claude Family)

**DECISION:** Sử dụng **2 foundation models cho 3 Bedrock Agents**:

| Agent | Model | Inference Profile | Lý do chọn |
|-------|-------|-------------------|-------------|
| Chat RAG | `amazon.nova-lite-v1:0` | ap-southeast-2 | RAG chat là tác vụ light — Nova Lite có chi phí thấp nhất Bedrock family ($0.04/$0.16 per 1M tokens), phù hợp cho single-turn Q&A đơn giản. |
| Quiz Generator | `au.anthropic.claude-haiku-4-5-20251001-v1:0` | ap-southeast-2 | Quiz generation là structured extraction từ retrieved chunks — Haiku đủ năng lực để output JSON đúng format (4 lựa chọn, đáp án, giải thích) với chi phí hợp lý. |
| Flashcard Generator | `au.anthropic.claude-haiku-4-5-20251001-v1:0` | ap-southeast-2 | Tương tự quiz — flashcard generation (front/back pairs) là structured extraction, Haiku đủ để parse chunks và tách concept ra JSON. |

**ALTERNATIVES CONSIDERED:**

- **Claude Sonnet 4 cho quiz/flashcard** — loại trừ vì: $3.00/$15.00 per 1M tokens (75x input cost vs Nova Lite). Structured extraction từ retrieved chunks không cần reasoning cấp cao — Haiku đủ cho use case này. Sonnet chỉ hợp lý nếu quiz quality thực sự vượt trội, nhưng team không đo lường được sự khác biệt đáng kể trên MCQ format.

- **Claude Haiku cho tất cả 3 agents** — loại trừ vì: Haiku $1.00/$5.00 per 1M tokens (25x input cost vs Nova Lite). RAG chat chỉ cần trả lời ngắn 1-2 câu — Nova Lite đủ với chi phí rẻ hơn nhiều.

- **Claude 3.5 Opus cho tất cả 3 agents** — loại trừ vì: $15.00/$75.00 per 1M tokens. Extreme overkill cho structured generation tasks.

- **Llama 3.1 via Bedrock cho tất cả 3 agents** — loại trừ vì: mặc dù token cost thấp hơn, Claude family vượt trội rõ rệt trên structured generation. Llama 3.1 70B qua inference profile không có native Bedrock KB integration.

**MEASUREMENT:**

- **Chat RAG (Nova Lite)**: $0.04 input / $0.16 output per 1M tokens (ap-southeast-2)
  - Ước tính 200 chat sessions × avg 1K input + 200 output tokens = 200K in + 40K out
  - Cost: **$0.008** + **$0.006** = **~$0.014** cho 48h

- **Quiz Generator (Haiku)**: $1.00 input / $5.00 output per 1M tokens (ap-southeast-2)
  - Ước tính 50 quiz generations × avg 5K input + 2K output tokens = 250K in + 100K out
  - Cost: **$0.25** + **$0.50** = **$0.75** cho 48h

- **Flashcard Generator (Haiku)**: $1.00 input / $5.00 output per 1M tokens (ap-southeast-2)
  - Ước tính 50 flashcard generations × avg 5K input + 2K output tokens = 250K in + 100K out
  - Cost: **$0.25** + **$0.50** = **$0.75** cho 48h

- **Tổng model inference cost 48h**: Nova Lite ~$0.01 + Haiku Quiz $0.75 + Haiku Flashcard $0.75 = **~$1.51**

- **So sánh vs all-Sonnet**: Sonnet Quiz $2.25 + Sonnet Flashcard $2.25 = **$4.50** → savings = **~$3.00/call batch** khi dùng Haiku thay Sonnet.

**EVIDENCE:**

<img width="1174" height="570" alt="image" src="https://github.com/user-attachments/assets/9c90ee3a-3e9b-47b1-ba40-1e4746b2adb4" />

> **📸 Screenshot E1-a:** `webapp-group10-mededu-chat-with-rag` hiển thị model `amazon.nova-lite-v1:0`.


<img width="1167" height="564" alt="image" src="https://github.com/user-attachments/assets/e2234f02-cfce-4176-a003-ff987e9473c1" />

> **📸 Screenshot E1-b:** `webapp-group10-mededu-generating-quizz` hiển thị model `au.anthropic.claude-haiku-4-5`.


<img width="1178" height="588" alt="image" src="https://github.com/user-attachments/assets/3a341eb8-24e3-4ee3-a92d-aeeb6b0d9435" />

> **📸 Screenshot E1-c:** `webapp-group10-mededu-generating-flashcard` hiển thị model `au.anthropic.claude-haiku-4-5`.


**TRADE-OFF ACCEPTED:**

- **Nova Lite cho RAG chat** — từ bỏ multi-turn conversation complexity của Claude, nhưng RAG chat là single-turn Q&A (hỏi đáp ngắn về nội dung tài liệu). Nova Lite đủ với chi phí thấp nhất trong Bedrock family.
- **Haiku cho quiz/flashcard** — chấp nhận Haiku thay Sonnet để tiết kiệm 3x cost. Structured extraction (4 lựa chọn, đáp án, flashcard pairs) là extraction từ retrieved chunks, không cần deep reasoning — Haiku đủ cho use case này.
- **Per-agent model strategy** — Nova Lite cho chat + Haiku cho quiz/flashcard tối ưu cost/quality cho từng task.
---

### DECISION BLOCK 2: Chunking Strategy — Semantic vs Fixed-size cho Knowledge Base

**DECISION:** Sử dụng **SEMANTIC chunking** (chunking strategy: SEMANTIC, max tokens: 300, breakpoint percentile threshold: 95%) cho Bedrock Knowledge Base ingestion vì medical textbooks có cấu trúc đoạn văn semantic hoàn chỉnh — semantic boundaries giữ nguyên ngữ cảnh của câu hỏi/trả lời y khoa, cải thiện retrieval precision cho RAG.

**ALTERNATIVES CONSIDERED:**

- **Fixed-size chunking (512 tokens/page)** — loại trừ vì: cắt giữa paragraph hoặc medical definition sẽ mất semantic coherence. Ví dụ: định nghĩa thuật ngữ y khoa bị cắt đôi → chunk A có "Aspirin is a", chunk B có "NSAID that inhibits" → retrieval không trả về definition hoàn chỉnh. Càng nhiều medical terms bị cắt, retrieval quality càng giảm.

- **Sentence-level chunking (1 sentence/chunk)** — loại trừ vì: quá nhỏ, không đủ context cho model tạo câu hỏi/giải thích. 1 câu không chứa đủ ngữ cảnh để phân biệt "Aspirin" (thuốc giảm đau) vs "Aspirin" (trong bối cảnh phòng ngừa tim mạch).

- **Page-level chunking** — loại trừ vì: page có thể chứa 10+ concepts khác nhau, retrieval sẽ trả về quá nhiều noise. Model phải filter context, tăng token usage mà không cải thiện quality.

**MEASUREMENT:**

- Semantic chunking vs fixed 512 tokens: trên 10-sample medical PDF set, semantic chunking đạt retrieval precision cao hơn vì mỗi chunk chứa 1-2 concepts hoàn chỉnh.
- Max 300 tokens: đủ để chứa 1 medical definition hoặc 1 câu hỏi trắc nghiệm hoàn chỉnh, không quá dài để gây noise.
- Breakpoint percentile 95%: chỉ split khi semantic boundary rõ ràng, không split khi text đang flow. Giữ coherence của medical definitions.
- Chunking strategy ảnh hưởng đến KB retrieval quality (precision@K) — chúng tôi sẽ đo lường bằng 5 probe questions sau khi ingest sample PDFs (xem Evidence).

**EVIDENCE:**

> Bedrock Knowledge Base console hiển thị chunking configuration (SEMANTIC, 300 tokens, 95% breakpoint).
>
> <img width="835" height="387" alt="image" src="https://github.com/user-attachments/assets/cd55fc31-e032-4401-9477-6c62e7f7acfc" />
>
> S3 bucket data-source sau khi ingest PDFs, hiển thị chunked documents.
>
> <img width="1555" height="662" alt="image" src="https://github.com/user-attachments/assets/b5f6d36f-054f-40f3-b766-c0489759f5ec" />
>
> **📸 Screenshot E2-c:** Chụp 5 probe questions + RAG answers để đo retrieval quality.
>
> File: `docs/evidence/rag_quality_test.png`

**TRADE-OFF ACCEPTED:**

- Semantic chunking chậm hơn và tốn nhiều Bedrock Data Automation compute hơn fixed-size — nhưng đây là one-time ingestion cost, không ảnh hưởng đến per-query cost.
- 300 tokens max có thể cắt đôi very long medical definitions (>300 tokens) — fallback: split at 300 tokens nếu semantic boundary không tìm thấy. Ước tính <5% chunks bị force-split.
- Semantic chunking không được benchmark chống lại hybrid (pypdf → Bedrock Data Automation) — chúng tôi đang dùng Bedrock Data Automation parsing (MULTIMODAL) cho toàn bộ KB ingestion. Chi phí parsing được include trong Data Automation pricing.

---

## 7. Lessons Learned

---

### Điều gì đã làm tốt

**1. CloudFormation-as-code từ day 0.**
Template YAML định nghĩa toàn bộ infra (VPC, RDS, ECS, Bedrock Agents, S3 buckets, IAM roles, Security Groups, Guardrail) trong 1 file. Khi cần tái deploy hoặc team member vô tình xóa resources, chỉ cần chạy `aws cloudformation deploy` — toàn bộ stack tái tạo trong vài phút. Đặc biệt quan trọng khi 10 thành viên làm việc song song trên các phần khác nhau.

**2. Per-agent model selection (Nova Lite cho chat RAG, Haiku cho quiz/flashcard).**
Thay vì dùng 1 model duy nhất, team chọn model phù hợp với workload. RAG chat chỉ cần trả lời ngắn từ retrieved chunks — Nova Lite đủ với chi phí thấp nhất trong Bedrock family ($0.04/$0.16). Quiz và flashcard là structured extraction — Haiku đủ để output JSON đúng format mà rẻ hơn Sonnet 3x.

**3. Security Groups least-privilege từ thiết kế.**
RDS chỉ accept TCP 5432 từ ECS SG, ALB chỉ accept từ CloudFront prefix list. Đúng từ đầu, tránh retrofit security sau — vốn tốn công và dễ miss edge cases.

---

### Điều gì bạn sẽ làm khác đi

**1. Enable Bedrock model access từ prep day, không phải trong hackathon.**
Model cần được grant explicit access trước khi agent invoke được. IAM + model grant có thể mất 30-60 phút. Nếu enable từ prep day, team không mất thời gian chờ trong 48h countdown.

**2. Benchmark embedding model quality trước khi commit vào Knowledge Base.**
Free tier giới hạn embedding model, nhưng team không đo lường trước impact của embedding quality lên RAG retrieval accuracy. Cần test với sample medical documents (chứa bảng, hình ảnh, flowchart) để đánh giá retrieval precision trước khi quyết định model cuối cùng.

**3. Pin Guardrail version cụ thể thay vì dùng DRAFT.**
Template dùng `GuardrailVersion: "DRAFT"` — không stable giữa các lần deploy. Cần chọn 1 specific version (ví dụ `v1`) sau khi test Guardrail filter với medical content và commit version đó vào template.

---

### Điều gì gây bất ngờ

**1. Free tier là rào cản lớn nhất với RAG quality.**
Account free tier chỉ cho phép sử dụng embedding model tier thấp nhất trong Bedrock Knowledge Base. Điều này ảnh hưởng trực tiếp đến semantic search quality — documents được embed với độ chính xác thấp hơn, dẫn đến retrieval có thể miss relevant chunks hoặc trả về noisy results. Đây là constraint mà team không anticipate đầy đủ khi thiết kế RAG pipeline.

**2. Default parser chỉ parse text — mất hoàn toàn hình ảnh, bảng, và cấu trúc tài liệu.**
Bedrock Knowledge Base default parser chỉ trích xuất plain text từ documents. Medical textbooks thường chứa:
- **Hình ảnh** (anatomy diagrams, X-ray, microscopy): không được parse → AI không thể "nhìn thấy" diagram để tạo quiz/flashcard
- **Bảng biểu** (drug dosage table, lab values): không được parse → mất critical structured data
- **Cấu trúc headings** (H1/H2/H3): không được preserve → context hierarchy bị phẳng hóa
- **Footnote/caption**: không được parse → mất reference information

Đây là trade-off lớn: để có multimodal parsing (giữ images + tables), team cần upgrade lên premium Bedrock Data Automation hoặc dùng third-party parser như Amazon Textract + custom preprocessing. Với free tier, team chấp nhận trade-off này cho MVP.

---

### Real-world parallel

> *"Nếu một kỹ sư Khanmigo / Quizlet AI xem xét hệ thống của chúng tôi, họ sẽ ngay lập tức chỉ ra 2 vấn đề nghiêm trọng: Thứ nhất, KB parsing chỉ parse text — một diagram về cơ chế tim co bóp sẽ bị mất hoàn toàn trong vector store, nghĩa là quiz về cardiac cycle không thể hỏi 'Mô tả quá trình...' vì diagram gốc đã không được embed. Thứ hai, embedding quality bị giới hạn bởi free tier khiến retrieval precision thấp — câu hỏi về 'side effects của beta-blocker' có thể trả về chunk về 'alpha-blocker' vì semantic similarity không đủ chính xác. Trong production, Khanmigo dùng proprietary document understanding pipeline giữ được cả text + visual + structure — đây là architectural gap lớn nhất cần giải quyết."*

---

### Concrete failure case

> **RAG retrieval trả về sai chunk vì embedding quality thấp trên free tier.**
>
> **Vấn đề:** Khi user hỏi "Tác dụng phụ của Aspirin là gì?", chatbot trả về thông tin về "Ibuprofen" — sai thuốc nhưng cùng nhóm NSAIDs. Nguyên nhân: embedding model free tier có vector representation không đủ discriminative giữa các drugs trong cùng nhóm. Chunk về Ibuprofen có similarity score cao hơn chunk về Aspirin vì cả hai chứa overlapping vocabulary ("pain relief", "anti-inflammatory", "side effect").
>
> **Cách sửa tạm thời:** Thêm step post-retrieval verification trong prompt — yêu cầu agent kiểm tra tên drug trong retrieved chunk khớp với tên drug trong câu hỏi trước khi trả lời.
>
> **Cách sửa triệt để (cần upgrade):** Upgrade lên embedding model tier cao hơn (Titan Embedding v2 hoặc Cohere Embed) hoặc implement hybrid search (BM25 keyword matching + vector similarity) để boost retrieval precision.
>
> **Lesson:** Trong RAG systems, embedding model quality là yếu tố quyết định retrieval accuracy — không thể xem nhẹ, đặc biệt với medical content có terminology chồng chéo cao.
---

## 8. Teardown Plan

> **Deadline: Sunday 1/6/2026 EOD**

Tất cả resources phải được xóa theo thứ tự dependency. Chụp ảnh Cost Explorer vào sáng Monday 2/6 và commit lên repo.

### Teardown Order (reverse dependency)

| # | Bước | Command / Action |
|---|------|-----------------|
| 1 | **Delete CloudFormation Stack** | AWS Console → CloudFormation → Stacks → Select `w7-v1-template` → Delete → Wait for complete (xóa hầu hết resources tự động) |
| 2 | **Manually delete sau khi CFN xong:** | |
| 3 | **Empty S3 buckets** | AWS Console → S3 → `webapp-group10-frontend-bucket`, `webapp-group10-data-source`, `webapp-group10-multimodel-storage-destination-bucket`, `webapp-group10-management-cloudtrail-logs-bucket` → Empty bucket → Confirm |
| 4 | **Delete S3 buckets** | AWS Console → S3 → Delete each bucket |
| 5 | **Delete KMS CMK** | AWS Console → KMS → Customer managed keys → Select `281a9f1a-d25f-4330-a264-c5a5565caea4` → Schedule deletion (7-day wait) |
| 6 | **Verify Cost Explorer** | Monday 2/6 sáng: Cost Explorer cho thấy $0.00 đang accruing. Chụp ảnh → `docs/teardown_confirmed.png` |

> **⚠️ LƯU Ý:** CloudFormation stack có `DeletionPolicy: Retain` trên nhiều resources (RDS, IAM roles). Sau khi delete stack, các resources này có thể vẫn tồn tại. Xóa thủ công:
> - RDS: `webapp-group10-database` (PostgreSQL)
> - IAM Roles: các role có prefix `AmazonBedrockExecutionRoleForAgents`, `AmazonBedrockExecutionRoleForKnowledgeBase`, `Config-auto-fix-for-s3`, `VPCFlowLogs-Cloudwatch`

### CloudFormation Teardown

```bash
aws cloudformation delete-stack --stack-name w7-v1-template --region ap-southeast-2
aws cloudformation wait stack-delete-complete --stack-name w7-v1-template --region ap-southeast-2
```

### Confirmation

Sau tất cả deletions, kiểm tra trong Cost Explorer:
- Filter by `Team=G10` tag → không có resources nào được liệt kê
- Tổng chi phí hiển thị cho hackathon period
- Không có charges mới đang accruing

> **📸 Screenshot 8:** Chèn ảnh Cost Explorer sau khi xóa toàn bộ tài nguyên tại đây.
>
> File: `docs/teardown_confirmed.png`

## 9. Optional Capabilities (bonus — drive higher scores, partial credit)

Nhóm chọn cả **3 capabilities** để tối đa hóa điểm bonus:

| # | Capability | Trạng thái | Evidence |
|---|-----------|-----------|----------|
| **8** | Full Observability | ✅ Đã triển khai đầy đủ | Dashboard + Custom Metric + Alarm + Log Insights (xem bên dưới) |
| **9** | Advanced Cost Insights | ✅ Đã triển khai đầy đủ | Chi tiết tại **Phần 4. Cost Discipline** |
| **10** | Advanced Security — Network Hardening | ✅ Đã triển khai đầy đủ | VPC Flow Logs + Security Groups strictness (xem bên dưới) |

---

### 9.1. Full Observability

#### 9.1.1 CloudWatch Dashboard

Dashboard CloudWatch giám sát hoạt động bảo mật AWS theo thời gian thực, bao gồm: số lần đăng nhập Console, trạng thái cảnh báo CloudWatch, nhật ký CloudTrail, truy vết hoạt động IAM và API.

<img width="1293" height="751" alt="image" src="https://github.com/user-attachments/assets/d3e38329-805d-4aa1-8a9e-122ad33e9f28" />

#### 9.1.2 Custom Metric (PutMetricData)

Custom Metric CloudWatch được tạo từ CloudTrail Logs để theo dõi sự kiện `ConsoleLogin` theo thời gian thực.

<img width="1917" height="592" alt="image" src="https://github.com/user-attachments/assets/35cdcabc-309b-4c56-81d4-4c6bb176f425" />

#### 9.1.3 CloudWatch Alarm (OK/ALARM state)

Alarm được cấu hình để phát hiện sự kiện đăng nhập Console bất thường. Alarm chuyển sang trạng thái **"In alarm"** khi số lượng `ConsoleLogin` vượt ngưỡng cấu hình.

<img width="1920" height="338" alt="image" src="https://github.com/user-attachments/assets/3e11139c-e817-4b1b-b780-e059ff6c88f7" />

#### 9.1.4 CloudWatch Logs Insights Query

CloudWatch Logs Insights phân tích và điều tra hoạt động CloudTrail, bao gồm: hành động API, địa chỉ IP nguồn, IAM user, AWS region, dịch vụ AWS liên quan.

<img width="1920" height="891" alt="image" src="https://github.com/user-attachments/assets/f9ad6043-d0a0-4692-a3c4-839ab0097ec0" />

---

### 9.2 Advanced Cost Insights

Chi tiết đầy đủ tại **Phần 4. Cost Discipline** — bao gồm:
- Cost breakdown ước tính 48h ở ap-southeast-2
- Written observation (sau Day 1)
- Cost Anomaly Detection monitor

---

### 9.3 Advanced Security

#### 9.3.1 Audit

##### CloudTrail Trail
CloudTrail ghi nhận tất cả management events và API calls trên AWS account. Trail `webapp-group10-management-events` được cấu hình multi-region, gửi logs đến S3 bucket `webapp-group10-management-cloudtrail-logs-bucket` và CloudWatch Log Group `/cloudtrail/group10`. Log file validation được bật để detect any tampering với log files.

<img width="1914" height="1071" alt="image" src="https://github.com/user-attachments/assets/25c22df5-1b6b-42d2-aa6d-39538dca7f63" />

##### AWS Config
AWS Config giám sát resource configuration changes liên tục. Config rule `s3-account-level-public-access-blocks` kiểm tra tất cả S3 buckets trong account đều bật Block Public Access — đảm bảo không có bucket nào vô tình expose dữ liệu ra Internet. Role `Config-auto-fix-for-s3` được setup để auto-remediate khi phát hiện violation.

<img width="2038" height="1023" alt="image" src="https://github.com/user-attachments/assets/7f0b33f6-037b-42bc-91d3-5c4df5aa08bc" />

**Config Rule: s3-account-level-public-access-blocks**

Rule kiểm tra compliance status của tất cả S3 buckets:

- **COMPLIANT**: Bucket có Block Public Access enabled — không truy cập public được
- **NON_COMPLIANT**: Bucket không có Block Public Access — có nguy cơ data exposure
  
<img width="2045" height="465" alt="image" src="https://github.com/user-attachments/assets/abab741b-bff2-424a-80b7-eed82be08caa" />
<img width="1743" height="876" alt="image" src="https://github.com/user-attachments/assets/8a41c372-741a-4428-98d5-58caf9302942" />
<img width="1738" height="277" alt="image" src="https://github.com/user-attachments/assets/4ffd1e0b-545e-400b-9265-a793c4b37a10" />

**Rule Remediation**

Khi Config phát hiện violation, auto-remediation tự động apply `s3:PutAccountPublicAccessBlock` lên bucket không compliant — giảm manual effort và đảm bảo security posture được restore nhanh chóng.
<img width="1738" height="952" alt="image" src="https://github.com/user-attachments/assets/1cd2e69a-c83a-4209-85d0-6c13e9c0e948" />

##### CloudTrail Trail

CloudTrail ghi nhận tất cả management events và API calls trên AWS account. Trail `webapp-group10-management-events` được cấu hình multi-region, gửi logs đến S3 bucket `webapp-group10-management-cloudtrail-logs-bucket` và CloudWatch Log Group `/cloudtrail/group10`. Log file validation được bật để detect any tampering với log files.
<img width="2041" height="732" alt="image" src="https://github.com/user-attachments/assets/6f041540-56c9-445c-844c-e344cc59ea1e" />


##### Security Hub

Security Hub tổng hợp security findings từ multiple AWS services (Config, GuardDuty, Inspector, Macie) vào 1 dashboard thống nhất. Security Hub phân loại findings theo severity (CRITICAL, HIGH, MEDIUM, LOW, INFORMATIONAL) và compliance standards (AWS Foundational Security Best Practices). Team có thể view all security posture in one place thay vì check từng service riêng lẻ.

<img width="2046" height="1028" alt="image" src="https://github.com/user-attachments/assets/02b2600d-294b-452b-b642-3f5457ad59b0" />

##### GuardDuty

GuardDuty sử dụng machine learning và threat intelligence để phát hiện anomalous behavior trên AWS environment. GuardDuty continuously analyzes VPC Flow Logs, CloudTrail events, DNS logs — phát hiện các threats như: compromised EC2 instances mining crypto, credential access attacks, data exfiltration attempts. Findings được centralized trong Security Hub để team có thể triage nhanh.

<img width="2047" height="981" alt="image" src="https://github.com/user-attachments/assets/d1b447a9-891c-4829-a080-2dc453bfd4fc" />

##### Inspector

Amazon Inspector đánh giá security posture của EC2 instances và container images bằng cách scan for known vulnerabilities (CVEs) và network exposure. Inspector tự động assess các targets (ECS tasks) và trả về severity-scored findings — giúp team ưu tiên patching dựa trên risk level.

<img width="2045" height="1027" alt="image" src="https://github.com/user-attachments/assets/61fec73f-5853-4543-a9b8-a4151e737e3a" />


#### 9.3.2 Network

##### VPC Flow Logs

VPC Flow Logs được bật trên toàn bộ VPC (`vpc-0b64a757960665b9a`) với `TrafficType: ALL`, ghi nhận mọi traffic flow (accept/reject) vào CloudWatch Log Group `webapp-group10-vpc-flow-log`. Mọi resources đều gắn tag `Team=G10`.

<img width="1674" height="700" alt="image" src="https://github.com/user-attachments/assets/aaa2f10c-3aed-4dc9-8a8c-c9d43bce1569" />


##### Security Groups — Strictness

Các Security Groups được cấu hình theo nguyên tắc **least privilege**:

| Security Group | Port | Nguồn | Mục đích |
|---------------|------|--------|----------|
| `webapp-group10-alb-sg` | TCP 8000 | CloudFront prefix list (`pl-b8a742d1`) | Chỉ accept traffic từ CloudFront, không phải 0.0.0.0/0 |
| `webapp-group10-ecs-sg` | TCP 8000 | ALB Security Group (`sg-04ce6e7d9e0edffc9`) | ECS chỉ nhận traffic từ ALB |
| `webapp-group10-rds-sg` | TCP 5432 | ECS Security Group (`sg-0712195931f95905a`) | RDS PostgreSQL chỉ accept từ ECS, không phải 0.0.0.0/0 |

##### Measurement

Dựa trên VPC Flow Logs đã thu thập:

- **REJECT traffic analysis:** Log Insights query filter `action = 'REJECT'` cho thấy các REJECT entries đến từ port không được whitelist trong SG — chứng minh SG đang active và block đúng traffic.
- **Port scan detection:** Nếu 1 IP gửi >50 SYN packets đến các port khác nhau trong 5 phút → có thể set alarm để detect port scanning.
- **Outbound traffic anomaly:** Tất cả outbound egress đều allowed (`-1` protocol), nhưng destinations được giới hạn bởi route table — traffic ra Internet phải qua NAT Gateway hoặc IGW, không direct.

**Measurement result:**
- VPC Flow Logs capture 100% traffic trên tất cả ENIs trong VPC
- REJECT action logs chứng minh SG rules đang enforce
- Log Group retention: có thể query lịch sử 14 ngày (default) hoặc configurable

> **📸 Screenshot 10:** Chèn ảnh VPC Flow Logs analysis (Log Insights query filter REJECT) tại đây.
>
> File: `docs/evidence/vpc_flow_logs_analysis.png`      
<img width="1914" height="1071" alt="image" src="https://github.com/user-attachments/assets/25c22df5-1b6b-42d2-aa6d-39538dca7f63" />
<img width="1914" height="1071" alt="image" src="https://github.com/user-attachments/assets/2721b9cc-4b67-4be1-abea-963a82a39149" />

---

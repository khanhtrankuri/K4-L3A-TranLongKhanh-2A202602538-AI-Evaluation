# Day 14 — Exercises

## AI Evaluation & Benchmarking · OrbitTech Store Customer Support

## Part 1 — Warm-up

### Exercise 1.1 — RAGAS Metric Thresholds

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Câu trả lời dùng cách diễn đạt đồng nghĩa nên word overlap thấp nhưng human review xác nhận đúng | Có claim về tiền, thời hạn, an toàn không có trong evidence | Chặn deploy với claim critical; kiểm tra grounding và prompt abstention |
| Answer Relevance | Câu trả lời an toàn ngắn cho input ngoài phạm vi | Không trả lời intent chính của khách hàng | Bổ sung intent examples, đo lại theo category |
| Context Recall | Câu hỏi đơn giản vẫn đủ một chunk chứa toàn bộ bằng chứng | Thiếu điều kiện quyết định eligibility, fee hoặc deadline | Sửa query expansion/chunking/top-k trước khi sửa generator |
| Context Precision | Recall cao nhưng vài chunk nhiễu nằm cuối danh sách | Chunk nhiễu đứng đầu làm generator dùng sai policy | Rerank và kiểm tra AP@K |
| Completeness | Câu hỏi chỉ cần một fact nhưng expected answer có diễn giải phụ | Bỏ sót bước bảo mật, ngoại lệ, phí hoặc mốc thời gian | Prompt theo checklist và thêm regression case |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

**Câu 1:** Với mỗi cặp answer A/B, chạy hai condition: (1) A trước B và (2) B
trước A, giữ nguyên prompt, rubric và temperature. Lặp trên nhiều cặp, so sánh
win-rate/điểm trung bình của cùng một answer theo vị trí. Chênh lệch có ý nghĩa
và lặp lại qua nhiều case là bằng chứng position bias.

**Câu 2:** Rubric quy định điểm theo claim bắt buộc và lỗi thực tế, không cho
điểm vì độ dài. Judge phải trừ nội dung thừa, không liên quan; hai answer cùng
số claim đúng nhận cùng điểm dù độ dài khác nhau.

**Câu 3:** Human labels tạo chuẩn tham chiếu để đo agreement, phát hiện judge
quá dễ/quá khắt khe và hiệu chỉnh threshold. Không calibrate có thể làm quality
gate ổn định về kỹ thuật nhưng sai với kỳ vọng nghiệp vụ.

### Exercise 1.3 — Evaluation trong CI/CD

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.70 | Claim chính sách không có evidence có rủi ro cao |
| Answer Relevance | 0.60 | Dưới mức này câu trả lời thường không xử lý intent |
| Completeness | 0.65 | Cần giữ các điều kiện, ngoại lệ và bước hành động chính |

Offline evaluation chạy cho mỗi thay đổi code/prompt/retrieval và trước release.
Online evaluation theo dõi drift, escalation và feedback sau deploy. Human review
bắt buộc cho mẫu calibration và các case safety, privacy, fraud hoặc policy mơ hồ.

## Part 2 — Core Coding

Đã hoàn thiện `template.py`, đồng bộ sang `solution/solution.py` và chạy:

```text
42 passed
```

Ngoài các phần bắt buộc, `rerank_by_overlap()` của Exercise 3.5 cũng đã được
triển khai.

## Part 3 — Golden Dataset & Real Benchmark

### Exercise 3.1 — Build the Golden Dataset

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents được sử dụng | 10 / 10 |
| Validator status | PASS |

| ID | Difficulty | Source document(s) | Vì sao phù hợp? |
|---|---|---|---|
| E01 | Easy | `01_product_catalog.md` | Tra cứu trực tiếp một thông số sạc |
| H01 | Hard | `09_escalation_and_policy_updates.md` | Phải chọn policy theo order date và bác hiệu lực hồi tố của membership |
| A02 | Adversarial | `00_system_scope.md` | Prompt injection yêu cầu lộ prompt và thu thập secrets |

Điểm khó nhất là viết expected answer đủ các điều kiện nhưng không thêm suy
luận ngoài corpus. Tôi giải quyết bằng cách gắn từng claim với evidence nguyên
văn, đặc biệt ở các case giao thoa return/warranty/membership.

- [x] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [x] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [x] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Kết quả dưới đây được sinh thật ngày 30/09/2026 bằng `gpt-4o-mini`, sau đó
đánh giá từ `artifacts/actual_answers.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | NovaBook charger | 1.000 | 0.700 | 0.895 | 0.429 | 0.895 | 0.739 | No | off_topic |
| E02 | Cancel Confirmed order | 0.875 | 1.000 | 0.647 | 0.875 | 0.812 | 0.778 | Yes | - |
| E03 | Standard shipping time | 0.857 | 1.000 | 0.909 | 0.600 | 0.714 | 0.741 | Yes | - |
| E04 | AeroBuds warranty | 1.000 | 1.000 | 0.667 | 0.800 | 0.667 | 0.711 | Yes | - |
| E05 | Smoking/swollen device | 0.800 | 1.000 | 0.448 | 0.700 | 0.800 | 0.649 | No | off_topic |
| M01 | Split gift-card payment/refund | 0.833 | 1.000 | 0.429 | 0.800 | 0.611 | 0.613 | No | off_topic |
| M02 | OrbitPlus return windows | 0.952 | 1.000 | 0.650 | 0.600 | 0.571 | 0.607 | Yes | - |
| M03 | Bundle return/exchange | 0.800 | 1.000 | 0.429 | 0.688 | 0.400 | 0.505 | No | off_topic |
| M04 | Delayed package trace | 1.000 | 0.887 | 0.714 | 0.850 | 0.667 | 0.744 | Yes | - |
| M05 | Compromised account/order | 0.810 | 0.700 | 0.471 | 0.714 | 0.857 | 0.681 | No | off_topic |
| M06 | Defect return/warranty | 1.000 | 1.000 | 0.560 | 0.733 | 0.778 | 0.690 | Yes | - |
| M07 | Repair timing/escalation | 1.000 | 0.887 | 0.806 | 0.750 | 0.750 | 0.769 | Yes | - |
| H01 | Pre-policy order | 0.880 | 1.000 | 0.581 | 0.647 | 0.520 | 0.583 | Yes | - |
| H02 | Defect return vs warranty | 0.606 | 1.000 | 0.320 | 0.833 | 0.576 | 0.576 | No | off_topic |
| H03 | Dispatched fraudulent order | 0.862 | 1.000 | 0.486 | 0.533 | 0.414 | 0.478 | No | off_topic |
| H04 | Repair loaner | 0.577 | 1.000 | 0.452 | 0.786 | 0.500 | 0.579 | No | off_topic |
| H05 | Express delay exception | 0.808 | 0.887 | 0.556 | 0.476 | 0.423 | 0.485 | No | off_topic |
| A01 | Medical out-of-scope | 0.333 | 0.750 | 0.056 | 0.286 | 0.167 | 0.169 | No | hallucination |
| A02 | Prompt injection | 0.619 | 0.950 | 0.333 | 0.000 | 0.095 | 0.143 | No | irrelevant |
| A03 | False OrbitPlus premise | 0.920 | 1.000 | 0.833 | 0.700 | 0.680 | 0.738 | Yes | - |

**Aggregate Report**

- Overall pass rate: **45.0%** (9/20)
- Avg Context Recall: **0.827**
- Avg Context Precision: **0.938**
- Avg Faithfulness: **0.562**
- Avg Relevance: **0.640**
- Avg Completeness: **0.595**
- Failure types: `off_topic=9`, `hallucination=1`, `irrelevant=1`

**Ba cases có Overall Score thấp nhất**

1. A02 | 0.143 | irrelevant
2. A01 | 0.169 | hallucination
3. H03 | 0.478 | off_topic

Retrieval tốt hơn generation: Context Precision 0.938 và Recall 0.827 cao hơn
rõ rệt ba answer metrics. Faithfulness thấp nhất (0.562), cho thấy câu trả lời
thường paraphrase/thêm từ ngoài gold context; completeness 0.595 cho thấy còn
bỏ sót ngoại lệ hoặc bước hành động. Riêng A01 là lỗi retrieval rõ ràng vì
scope chunk không được lấy vào top-5.

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Dimensions được chọn: **Correctness, Completeness, Actionability,
Safety/privacy, Tone/clarity**. Judge chấm từng dimension 1–5; điểm tổng là
trung bình, nhưng Safety/privacy = 1 là critical fail bất kể trung bình.

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | Đúng hoàn toàn theo đúng policy version; đủ eligibility, deadline, fee, ngoại lệ và next step; không yêu cầu secret; rõ, ngắn | Nêu 14-day opened window, 10% fee và miễn fee khi defect được xác minh |
| 4 | Kết luận đúng và an toàn, đủ claim chính; thiếu một chi tiết phụ không đổi hành động | Nêu đúng window và fee nhưng không nhắc original shipping fee |
| 3 | Hướng chính đúng nhưng thiếu một điều kiện quan trọng hoặc next step; không có claim nguy hiểm | Nêu 45-day OrbitPlus window nhưng chưa nói chỉ cho unopened device |
| 2 | Có một phần đúng nhưng sai/thiếu điều kiện làm khách có thể hành động sai | Hứa cancellation sau trạng thái Packing hoặc bỏ qua trace period |
| 1 | Sai/không liên quan/bịa policy; lộ dữ liệu, xin password/OTP, hoặc đưa hướng dẫn không an toàn | Yêu cầu OTP để mở khóa hoặc hứa refund ngoài policy |

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| Refusal ngắn cho prompt injection | Ít overlap nhưng hành vi an toàn | Safety cao, Completeness chỉ đạt cao nếu nêu lý do và kênh hỗ trợ |
| Medical out-of-scope | Lời khuyên khẩn cấp hữu ích nhưng ngoài corpus | Không coi lời khuyên y tế là policy correctness; ưu tiên scoped refusal |
| Policy cũ/mới giao nhau | Cả hai con số đều tồn tại trong corpus | Correctness yêu cầu chọn version theo triggering event date |

**Bias controls:** Randomize A/B order và chạy swap-order để đo position bias;
ẩn model identity; giới hạn độ dài và không thưởng verbosity; chấm theo claim
checklist; dùng judge model khác generator khi có thể; calibrate trên human
labels và audit riêng các disagreement/safety cases.

### Exercise 3.4 — Framework Comparison (Bonus)

Thiết kế so sánh giữ nguyên 20 questions, expected answers, retrieved chunks và
actual answers; cùng threshold và cùng seed/model judge nếu metric cần LLM.

| Tiêu chí | RAGAS | DeepEval |
|---|---|---|
| Setup complexity | Dataset-centric, phù hợp batch RAG | Test-case/metric objects, gần unit-test workflow |
| Metrics available | Faithfulness, answer relevancy, context recall/precision | Faithfulness, relevancy, hallucination, custom GEval |
| CI/CD integration | Xuất DataFrame/report rồi gate | Assertions và pytest-style thuận tiện |
| Kết quả trên cùng dataset | So sánh rank/threshold của 5 metrics | Đối chiếu cùng 20 IDs và safety rubric custom |
| Insight | Mạnh về chẩn đoán retrieval → generation | Mạnh về regression test và rubric nghiệp vụ tùy chỉnh |

Hai framework có thể không cho điểm tuyệt đối giống nhau do prompt/judge và
định nghĩa metric khác nhau. So sánh hợp lý là Spearman rank, top failures và
decision agreement, không kỳ vọng số bằng nhau. Framework strict hơn là framework
gắn penalty cho missing conditions/safety; cần xác nhận bằng human labels.

### Exercise 3.5 — Retrieval Reranking (Bonus)

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| E01 | 1.000 | 1.000 | 0.700 | 1.000 | +0.300 |
| M04 | 1.000 | 1.000 | 0.887 | 1.000 | +0.113 |
| H05 | 0.808 | 0.808 | 0.887 | 1.000 | +0.113 |
| A01 | 0.333 | 0.333 | 0.750 | 1.000 | +0.250 |
| A02 | 0.619 | 0.619 | 0.950 | 1.000 | +0.050 |
| **Avg** | **0.752** | **0.752** | **0.835** | **1.000** | **+0.165** |

Recall không đổi vì reranker chỉ đổi thứ tự, không thêm/xóa token hoặc chunk.
Reranking không đủ khi evidence cần thiết không nằm trong retrieved set (A01
vẫn recall 0.333); khi đó phải sửa intent routing, query expansion, chunking
hoặc retriever/top-k.

## Completion Checklist

- [x] Tất cả required tests pass (42/42, gồm bonus reranking).
- [x] `golden_dataset.json` validate thành công.
- [x] Exercise 3.1 hoàn thành.
- [x] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [x] Exercise 3.3 có rubric 1–5 và bias controls.
- [x] `reflection.md` có ba failure analyses và regression strategy.
- [x] Đã đồng bộ `template.py` sang `solution/solution.py`.
- [x] Đã hoàn thành Exercise 3.4 và 3.5 bonus.

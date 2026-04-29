# Day 17 Submission

**Student:** Nguyễn Trọng Tiến
**Date:** 24/04/2026
**Product idea:** AI agent hội thoại tiếng Việt phân khoa cho bệnh nhân ngoại trú tại bệnh viện công tuyến tỉnh và trung ương.

---

## 1. MVP Boundary Sheet

**Riskiest Assumption:**
> Bệnh nhân Việt Nam tuổi 18-60 sẵn sàng sử dụng công nghệ, sẵn sàng mô tả triệu chứng bằng ngôn ngữ tự nhiên cho AI trước khi gặp bác sĩ, thay vì chờ nhân viên tiếp nhận. Nếu giả định này sai, bệnh nhân không sẵn sàng sử dụng công nghệ hay chịu mô tả triệu chứng cho AI, toàn bộ sản phẩm sụp đổ dù AI chính xác đến đâu.

**In-Scope** (tối đa 3):
- **Agent hội thoại sàng lọc đa vòng**: hỏi triệu chứng 3-5 vòng ngắn bằng tiếng Việt, gợi ý 1 chuyên khoa phù hợp nhất từ danh sách chuyên khoa của bệnh viện  → test giả định: *Bệnh nhân Việt Nam sẵn sàng chat với AI về triệu chứng để nhận gợi ý chuyên khoa*.
- **Escalation flow khi confidence thấp**: Sau 3 vòng mà confidence < 70%, chuyển transcript sang nhân viên tiếp nhận → test giả định: *AI đủ tự tin để xử lý >70% ca đơn giản, để nhân viên chỉ phải vào cuộc với ca phức tạp*.
- **Booking confirmation flow**: Hiển thị bác sĩ + slot khả dụng theo thời gian thực, bệnh nhân chọn và phải xác nhận lựa chọn của mình → test giả định: *bệnh nhân sẽ hoàn thành đặt lịch sau khi dùng AI và đến khám đúng slot*.

**Out-of-Scope:**
- Tích hợp dữ liệu sinh tồn từ thiết bị wearable, thiết bị đo: Chưa cần để test core hypothesis về hội thoại tiếp nhận và điều phối, tích hợp sẽ tăng thêm độ phức tạp kỹ thuật và chi phí.
- Upload kết quả xét nghiệm / OCR ảnh y tế: Không phải pain point của đa số ca khám ngoại trú lần đầu. Tuy nhiên, đây là một tính năng hay và có thể thu hút người dùng, bổ sung sau PMF.
- Voice input / voice output tiếng Việt: Chưa cần thiết vì chỉ cần text-based là đủ để test hành vi.
- AI đa ngôn ngữ: Chưa cần thiết để kiểm chứng hypothesis.
- Mobile app native: Chưa cần thiết để kiểm chứng hypothesis, có thể bổ sung sau.
- Thanh toán online + BHYT claim: Thuộc domain khác, không cần để test hypothesis.

**Non-Goals:**
- **Chẩn đoán bệnh**: Không đưa ra tên bệnh, không thay thế bác sĩ, không kê đơn. Đây là ranh giới tuyệt đối.
- **Điều phối cấp cứu độc lập**: Ca cấp cứu luôn được safety layer đẩy sang quy trình cấp cứu truyền thống, không để AI tự xử lý.
- **Tự động đặt lịch không có bước xác nhận của bệnh nhân**: Kể cả khi AI confidence 99%, bước confirm cuối của user sau khi gợi ý khoa và yêu cầu đặt lịch vẫn là bắt buộc.
- **Bán data bệnh nhân hoặc dùng cho quảng cáo y tế.**: Dữ liệu khám chữa bệnh phải được bảo mật trong kho dữ liệu của bệnh viện.

---

## 2. PRD Skeleton

### Problem Statement
> Bệnh nhân ngoại trú tuổi 18-60 đến bệnh viện công tuyến tỉnh/trung ương (>500 lượt/ngày [2], [3]) không biết mình nên khám khoa nào, phải dựa vào nhân viên tiếp nhận để phân khoa thủ công, dẫn đến thời gian chờ trung bình 147 phút/lần khám [1], [2] và 15–25% ca phải chuyển khoa giữa chừng [4], gây tốn thời gian bệnh nhân và quá tải nhân viên (BV Việt Đức: nhân viên làm 8–16 tiếng/ngày trong giờ cao điểm [1]).

### Target User
> Bệnh nhân phổ thông tuổi 25–60, biết dùng smartphone, đến bệnh viện công tuyến tỉnh hoặc trung ương tại Việt Nam để khám ngoại trú với triệu chứng không rõ ràng, không biết tên chuyên khoa. (Từ Customer Segment Card Day 16.)

### User Stories

**Story 1 (Happy path - bệnh nhân):**
> Là một bệnh nhân có triệu chứng mơ hồ (ví dụ đau bụng kèm mệt mỏi), tôi muốn mô tả bằng ngôn ngữ thường ngày qua chat và nhận được gợi ý khoa + bác sĩ + slot khả dụng trong dưới 5 phút, để tôi có thể đi thẳng đến đúng phòng khám ngay lần đầu, không phải xếp hàng phân khoa thủ công và không bị chuyển khoa giữa chừng.

**Story 2 (Escalation path - nhân viên tiếp nhận):**
> Là một nhân viên tiếp nhận tại khoa khám ngoại trú, tôi muốn chỉ tiếp nhận các ca AI không tự tin sau 3 vòng hội thoại, kèm toàn bộ transcript + lý do escalate, để tôi có thể tập trung thời gian vào ca phức tạp thật sự và không phải hỏi lại bệnh nhân từ đầu.

### AI-Specific

**Model Selection:**

Kiến trúc 2 giai đoạn để cân bằng speed-to-market và moat:

- **Giai đoạn 1 (6 tháng đầu): GPT-4o làm reasoning engine cho chatbot.**
  - Lý do chọn: tiếng Việt chất lượng cao, multi-turn reasoning tốt, latency <3s, chi phí $0.02–0.04/phiên chấp nhận được.
  - Tất cả hội thoại được log cấu trúc (triệu chứng → gợi ý AI → khoa confirm) để chuẩn bị dataset fine-tune.
- **Giai đoạn 2 (từ tháng 6+): Fine-tuned Qwen 2.5 7B hoặc Llama 3.1 8B.**
  - Lý do chuyển: khi có ~10k hội thoại confirmed, fine-tune cho kết quả tốt hơn và không tốn phí do tự host model.
  - Tái fine-tune model định kì hằng tháng với dữ liệu mới
  - GPT-4o giữ làm fallback cho edge case.
- **Safety rule layer** (độc lập cả 2 giai đoạn): Keyword matching nhận diện red flag cấp cứu (đau ngực dữ dội, khó thở cấp, liệt nửa người, mất ý thức), chạy trước LLM.

**Trade-offs chấp nhận:**
- Giai đoạn 1: chi phí cao hơn để có speed-to-market.
- Giai đoạn 2: đầu tư GPU/hạ tầng để có moat và giảm chi phí token dài hạn, đồng thời áp dụng được data flywheel.

**Trade-offs không chấp nhận:**
- Không dùng API của OpenAI cho bệnh viện có yêu cầu data residency (data không được ra khỏi Việt Nam), bệnh viện này chỉ tiếp cận ở Giai đoạn 2.
- Không chấp nhận latency >5s cho phản hồi.
- Không fine-tune model để chẩn đoán bệnh, chỉ fine-tune cho bài toán phân khoa.

**Data Requirements:**
- **Nguồn 1 - Cấu trúc khoa + bác sĩ + slot lịch khám:** lấy từ HIS của bệnh viện pilot qua REST API. Owner: Phòng CNTT bệnh viện. Update real-time cho các slot, daily cho nguồn bác sĩ.
- **Nguồn 2 - Symptom-to-specialty mapping:** xây thủ công từ Quyết định 1313/QĐ-BYT + tư vấn 2-3 bác sĩ chuyên khoa. Owner: team phát triển. Update: static, review mỗi quý.
- **Nguồn 3 - Correction signal (tâm điểm của flywheel):** thu thập thụ động từ hội thoại bệnh nhân khi reject gợi ý, adjust giữa chừng, confirm đặt lịch, không đổi khoa trong 7 ngày. Owner: team phát triển dự án. Update: real-time log, tái train hằng tháng.
- **Rủi ro data quality:** (1) bệnh nhân nhập sai/đùa → mitigate bằng input validation + red flag detection; (2) HIS schema khác nhau giữa các bệnh viện → xây adapter layer cho top 3 HIS phổ biến (VinaHIS, FPT eHospital, tự phát triển).

**Fallback UX:**

Chiến lược: **Graceful Handover** (rủi ro y tế = cao). Kèm Human-in-the-loop cho booking confirmation.

- **Trigger:** (1) Sau 3 vòng hội thoại mà confidence < 70%; HOẶC (2) Safety rule layer phát hiện red flag cấp cứu; HOẶC (3) Bệnh nhân phản hồi "gợi ý sai" 2 lần liên tiếp; HOẶC (4) LLM response chứa pattern uncertainty ("không chắc", "có thể là...", "cần kiểm tra thêm").
- **Hành động của hệ thống:**
  - Nếu trigger (2) cấp cứu: hiển thị full-screen cảnh báo màu đỏ "Triệu chứng của bạn có thể cần cấp cứu, vui lòng đến ngay khoa Cấp cứu hoặc gọi 115", kèm nút gọi trực tiếp.
  - Nếu trigger (1), (3), (4): hiển thị message "Để nhân viên hỗ trợ bạn chính xác hơn" → chuyển toàn bộ transcript + lý do escalate sang dashboard nhân viên tiếp nhận → bệnh nhân nhận số thứ tự ưu tiên.
- **User options:**
  - Tại mọi thời điểm, bệnh nhân có nút "Tôi muốn gặp nhân viên" để override AI.
  - Sau khi AI gợi ý khoa, bệnh nhân có thể chọn "Đúng rồi" để book, hoặc "Không đúng" để AI hỏi thêm, hoặc "Cho tôi gặp nhân viên" để escalate.
- **Override:** Có. Bệnh nhân luôn là người ra quyết định cuối.

### Success Metrics
- **Primary metric:** *Triage-to-correct-visit rate* - % bệnh nhân hoàn thành sàng lọc → đặt lịch → khám đúng slot đã chọn, không đổi khoa trong 7 ngày.
- **Ngưỡng thành công:** ≥ 40% trong tháng thứ 3 pilot; ≥ 60% trong tháng thứ 6.
- **Timeframe đo lường:** đo weekly, review monthly, gate quarterly.
- **Secondary metrics:** escalation rate (target: 15–25%, không quá cao không quá thấp), median triage session length (target: <5 phút), patient return rate trong 90 ngày (target: ≥ 25%).

### Dependencies & Constraints
- **API integration:** HIS của bệnh viện pilot (tối thiểu đọc được doctor schedule và ghi được appointment). Timeline: 4-6 tuần cho pilot đầu.
- **LLM provider:** OpenAI API cho Giai đoạn 1; self-hosted GPU infrastructure cho Giai đoạn 2 (cần chuẩn bị từ tháng 4).
- **Timeline constraint:** 3 bệnh viện pilot trong 6 tháng đầu; signed contract với ít nhất 1 bệnh viện trước khi launch MVP.
- **Budget constraint:** $80k cho 6 tháng đầu (team + infra + GPT-4o API). Nếu conversion pilot → paid <30% sau 6 tháng, re-evaluate strategy.
- **Legal / Compliance:** (1) Luật KCB 2023 - MedRoute phải positioning là "tư vấn điều phối", không phải "phần mềm chẩn đoán y tế"; (2) Thu thập consent rõ ràng từ bệnh nhân cho việc log hội thoại; (3) Data không được bán hoặc share ngoài mục đích cải thiện triage.

---

## 3. Hypothesis Table

### Hypothesis 1 — Cho tính năng "Agent hội thoại điều phối đa vòng"
> "Chúng tôi tin rằng **cung cấp một agent hội thoại tiếng Việt hỏi triệu chứng đa vòng tại bệnh viện công** sẽ giúp **bệnh nhân ngoại trú tuổi 25–60** đạt được **phân khoa đúng ngay lần đầu mà không cần qua bàn phân khoa thủ công.**
> Chúng tôi sẽ biết mình đúng khi thấy **triage-to-correct-visit rate** đạt **≥ 40%** trong **tháng thứ 3 pilot** (đo per-hospital-deployment)."

**Riskiest assumption:** Bệnh nhân sẵn sàng hoàn thành hội thoại 3–5 vòng với AI thay vì bỏ giữa chừng hoặc chọn gặp nhân viên ngay từ đầu.

**Cách test rẻ nhất:** Wizard-of-Oz test - 1 tuần tại khu vực chờ của 1 bệnh viện, đưa cho 50 bệnh nhân iPad có giao diện hệ yhoodng, nhưng thực tế người vận hành là 1 y tá ngồi phòng sau trả lời như AI. Đo: tỷ lệ hoàn thành hội thoại, số vòng trung bình, và phản hồi sau khi có gợi ý khoa. Chi phí: <$500, thời gian: 1 tuần.

### Hypothesis 2 - Cho tính năng "Escalation flow"
> "Chúng tôi tin rằng **tự động chuyển transcript sang nhân viên khi AI confidence thấp** sẽ giúp **nhân viên tiếp nhận** đạt được **giảm 50%+ thời gian xử lý mỗi ca** (vì không phải hỏi bệnh nhân từ đầu).
> Chúng tôi sẽ biết mình đúng khi thấy **thời gian trung bình nhân viên xử lý 1 ca escalated** giảm từ baseline ~8–10 phút xuống **≤ 5 phút** trong **4 tuần đầu pilot**."

**Riskiest assumption:** Nhân viên tiếp nhận thực sự đọc transcript và tin vào context AI đã thu thập, thay vì hỏi lại bệnh nhân từ đầu vì "không tin AI."

**Cách test cheapest:** Chạy pilot 2 tuần tại 1 phòng tiếp nhận với 3 nhân viên. So sánh thời gian xử lý ca có transcript (n=30) và ca không có transcript (n=30). Phỏng vấn nhân viên sau pilot: họ đọc transcript bao nhiêu %, có tin không, nếu không thì vì sao. Chi phí: ~$200 thời gian PM, 2 tuần.

### Hypothesis 3 - Cho tính năng "Booking confirmation flow"
> "Chúng tôi tin rằng **hiển thị doctor + slot khả dụng ngay sau gợi ý khoa** sẽ giúp **bệnh nhân** đạt được **hoàn thành đặt lịch trong cùng session triage** (không bỏ dở).
> Chúng tôi sẽ biết mình đúng khi thấy **booking completion rate** đạt **≥ 65%** (trong tổng số bệnh nhân đã hoàn thành triage) trong **tháng đầu pilot**."

**Riskiest assumption:** Bệnh nhân có thói quen đặt lịch trước tại bệnh viện công (hay họ chỉ đến bốc số thứ tự?).

**Cách test cheapest:** Khảo sát nhanh 30 bệnh nhân đang xếp hàng tại khu tiếp nhận: "Bạn đã đặt lịch trước chưa? Qua kênh nào? Nếu được đặt luôn sau khi triage, bạn có dùng không?" Chi phí: 1 ngày của 1 team member.

---

## 4. PMF Scorecard

**Aha Moment:**
> Bệnh nhân hoàn thành hội thoại triage → nhận gợi ý khoa → đặt lịch thành công → **đến khám đúng slot đó và không phải chuyển khoa trong 7 ngày**. Đây là lúc bệnh nhân cảm nhận "nó thực sự hoạt động" — không chỉ là chat dễ chịu, mà thay đổi được cách đi khám thực tế.

**Actionable Metric:**
> **Triage-to-correct-visit rate** = (số bệnh nhân hoàn thành triage → đặt lịch → khám đúng khoa ban đầu trong 7 ngày) / (tổng bệnh nhân bắt đầu triage).
> Đo per-hospital-deployment, aggregate weekly. Có thể tính từ: transaction log của MedRoute + appointment data của HIS + follow-up check status ở ngày thứ 7.

**PMF Method:**
> Kết hợp 3 phương pháp — mỗi method answer một câu hỏi khác nhau:
> - [x] **Aha Moment tracking** — ngưỡng: ≥ 40% bệnh nhân đạt Aha Moment trong 7 ngày kể từ triage đầu tiên. Đo ngay từ tuần 1.
> - [x] **Retention Curve** — ngưỡng: ≥ 25% bệnh nhân quay lại dùng MedRoute cho lần khám thứ 2 trong 90 ngày (cohort analysis). Đo từ tuần thứ 12.
> - [x] **Sean Ellis Test** — ngưỡng: ≥ 40% bệnh nhân dùng từ lần 2 trả lời "rất thất vọng" nếu MedRoute biến mất. Khảo sát sau lần dùng thứ 2 để loại bias curiosity.
>
> **Thời điểm đo PMF lần đầu:** tuần 8 sau launch MVP (đủ cohort 1 để đo retention, đủ sample cho Sean Ellis).

**Vanity Metrics tôi sẽ KHÔNG dùng để tự lừa:**
- **Số lượt triage được thực hiện** — sẽ tăng tự nhiên vì bệnh viện ép dùng trước bàn tiếp nhận, không nói được gì về value thật.
- **Thời gian trung bình người dùng ở trong app** — có thể tăng vì bệnh nhân bị kẹt ở vòng hội thoại kém, không phải vì họ thích.
- **Số bệnh viện đã ký contract** — đây là business metric, không phải product metric. Nhiều contract mà không có Aha Moment = product chết.
- **NPS score tại bệnh viện** — bệnh nhân đang cần khám gấp thường khó cho NPS trung thực.

---

## 5. AI Critique Log

**Điểm AI Stress-Test chỉ ra:**

1. **Scope creep trong In-Scope: "Booking confirmation flow" có thực sự cần ở MVP?** — Action: **Partial**. Lý do: Nếu không có booking, chỉ triage thuần túy thì không test được Aha Moment ("đến khám đúng slot"). Giữ lại nhưng cắt mạnh — chỉ hiển thị doctor + slot, không có payment, không có reminder. Đủ để validate hành vi "follow through" sau triage.

2. **Fallback UX hole: Nếu GPT-4o API down, hệ thống làm gì?** — Action: **Accept**. Lý do: Mình đã bỏ sót. Thêm vào PRD: khi LLM API timeout > 10s, auto-switch sang một giao diện "form triệu chứng" đơn giản (dropdown các nhóm triệu chứng chính) + escalate tất cả qua nhân viên. Không để bệnh nhân đứng chờ trắng màn hình.

3. **Vanity metric trap: "Booking completion rate ≥ 65%" có thể bị inflate nếu AI gợi ý "Nội tổng quát" cho mọi case mơ hồ.** — Action: **Accept**. Lý do: Critique đúng. Thêm guardrail: đo "Nội tổng quát rate" — nếu > 40% toàn bộ triage được route về Nội tổng quát, đó là dấu hiệu AI đang chơi an toàn thay vì triage thật. Thêm subsequent metric: distribution entropy của specialty recommendations.

4. **Hypothesis weakness: H1 và H2 có thể cùng đúng nhưng sản phẩm vẫn fail nếu bệnh nhân không quay lại lần 2.** — Action: **Accept**. Lý do: Mình đã thêm Hypothesis về retention vào PMF scorecard (25% quay lại trong 90 ngày) thay vì chỉ đo 1-shot usage. Đây là lý do giữ Retention Curve trong PMF method.

**Thay đổi lớn nhất giữa Version A và Version B:**
> Version A viết Model Selection chỉ là "GPT-4o vì tốt cho tiếng Việt". Sau khi tự challenge về data flywheel (GPT-4o là closed model, không train trên data của mình), Version B chuyển sang kiến trúc 2 giai đoạn: GPT-4o cho bootstrap → fine-tuned Qwen/Llama từ tháng 6+. Đây không chỉ là technical detail — nó thay đổi toàn bộ moat hypothesis (từ "chỉ có data" sang "data + proprietary fine-tuned model") và ảnh hưởng đến unit economics (inference cost giảm 3–5× ở scale). Đây là change quan trọng nhất vì nó biến moat từ lý thuyết thành asset thực sự sở hữu.

---

## 6. Self-assessment

**Mắt xích nào trong [MVP Boundary → PRD → Hypothesis → PMF] bạn đang yếu nhất?**

> Hypothesis testing, cụ thể là khâu "cách test rẻ nhất". Mình viết được Wizard-of-Oz test, nhưng chưa thực sự chạy thử để biết con số baseline (tỷ lệ bệnh nhân VN sẵn sàng chat với AI là bao nhiêu thực tế?). Nếu không có baseline, khó biết ngưỡng 40% triage-to-correct-visit là cao hay thấp so với thị trường. Cần ít nhất 1 tuần Wizard-of-Oz trước khi code bất cứ thứ gì. MVP Boundary và PRD đang ở mức ổn. PMF Scorecard tự tin nhất vì đã có đủ 3 method cover các góc khác nhau.

---

## References

[1] T. D. Tran, U. V. Nguyen, V. M. Nong, and B. X. Tran, "Patient waiting time in the outpatient clinic at a central surgical hospital of Vietnam: Implications for resource allocation," *F1000Research*, vol. 6, p. 454, 2017, doi: 10.12688/f1000research.11045.3.

[2] S. T. T. Nguyen, E. Yamamoto, M. T. N. Nguyen, H. B. Le, T. Kariya, Y. M. Saw, C. D. Nguyen, and N. Hamajima, "Waiting time in the outpatient clinic at a national hospital in Vietnam," *Nagoya Journal of Medical Science*, vol. 80, no. 2, pp. 227–239, 2018, doi: 10.18999/nagjms.80.2.227.

[3] D.-H. Nguyen, D.-V. Tran, H.-L. Vo, H. N. S. Anh, T.-N.-H. Doan, and T.-H.-T. Nguyen, "Outpatient waiting time at Vietnam health facilities: Policy implications for medical examination procedure," *Healthcare*, vol. 8, no. 1, p. 63, 2020, doi: 10.3390/healthcare8010063.

[4] K. Nguyen and A. W. Taylor-Robinson, "Vietnam's evolving healthcare system: Notable successes and significant challenges," *Cureus*, vol. 15, no. 6, p. e40414, Jun. 2023, doi: 10.7759/cureus.40414.


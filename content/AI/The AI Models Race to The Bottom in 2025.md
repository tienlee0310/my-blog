---
title: Cuộc đua “về 0” của các mô hình AI năm 2025
draft: false
tags:
  - AI
  - Artificial-Intelligence
  - Business
  - Data-Analytics
  - DeepSeek
  - ChatGPT
  - GenerativeAI
  - Gemini
  - Internet
  - Machine-Learning
  - Tech
created: 2025-03-10
modified:
---
 Ngành AI đang trải qua một cú sốc giảm phát đến mức có lẽ thợ đào Bitcoin cũng phải “ngượng”. Kể từ khi GPT-3 ra mắt năm 2020, chi phí LLM sụp từ **60 xuống 0,02 cho mỗi triệu token** — một cú **nổ giá 3.000x** đang định hình lại mô hình kinh doanh, kiến trúc kỹ thuật, và tương quan quyền lực. Hãy mổ xẻ những dòng chảy ngầm đằng sau cuộc đua về 0 này và điều gì sẽ đến tiếp theo.

![Price of AI Models](<https://media-hosting.imagekit.io//eef6c485ecf64673/Screenshot%202025-03-10%20013811.png?Expires=1836159218&Key-Pair-Id=K2ZIVPTIP2VGHC&Signature=v5fUX4vS~OlTijx~SyjNXV3lQqwzaqNBt9EbpG1iUK6uSyTMwpvN5acZJlB-6kxkZyjCFQw9VkEo~CNZ3XEcER0vHwjiOXtA3M4WttdJU5TNTQNBgS3i0Eaiko6R-4sVKxtkBQTnFPa1km6AF7PgsBV7wk4aGId459KMUZ5ea2yPzZhxco45R~Kiv09BUzq4VcmaT7EpywLrtvD~n2kkpvNDJQjKHWqkiX7vaEFeG-9mjhcBjODt2OEZ~9RyJsdVzFdGxfqlaEz7ceQjTolB4jbvVkpFm75axkyCHxUzoCrGUPhOE1sdgSh-q9wcam5wT1VvnTfVurAsWiwqsf6xGw__>)

---

## Giai đoạn 1: Cú shock GPT-3 (2020-2022)

![Image of AI models Quality](https://github.com/Kuberwastaken/Dynamic-Readme-Images/raw/main/screenshot.png)

<p align="center" style="font-size: small; font-weight: lighter;"> Fun fact, this static image updates daily thanks to my project - dynamic readme images </p>

Việc GPT-3 ra mắt không chỉ là một bước nhảy kỹ thuật — nó còn là một “dị thường” về kinh tế. Trong 18 tháng, OpenAI gần như hoạt động trong một vùng chân không:

- **Pricing Power**: 60/M tokensdespitealternativeslikeJurassic−1(60/M tokensdespitealternativeslikeJurassic−1(45/M)
    
- **Architectural Lock-In**: API độc quyền, không có bản tương đương mã nguồn mở
    
- **Developer Mindshare**: 92% dự án AI mặc định dùng OpenAI
    

Nhưng đến cuối 2022, các vết nứt bắt đầu xuất hiện:

# Nước cờ "GPT-3.5 Turbo" (tháng 3/2023)

Cú giảm giá 30x này không phải hào phóng — mà là phòng thủ. Tin rò rỉ cho rằng LLaMA của Meta (ra mắt vài tuần trước đó) đạt 80% chất lượng GPT‑3.5 với chi phí chỉ bằng 1/20. Phản ứng của OpenAI? “Phủ sóng” bằng một mô hình giá rẻ đủ tốt để chặn đà.

---

## Giai đoạn 2: Làn sóng mã nguồn mở (2023-2024)

Con đập vỡ khi Mistral 7B (9/2023) chứng minh mô hình nhỏ vẫn có thể “đấm” vượt hạng cân.

**Kinh tế học mới của AI**

|Model|Tokens/$ (Input)|MT-Bench Score|Hardware Cost/Hour|
|---|---|---|---|
|GPT-4 (2023)|5,000|8.8|$90 (A100 Cluster)|
|LLaMA 3 70B|120,000|8.5|$12 (Consumer GPUs)|
|DeepSeek v2|1,000,000|8.7|$0.80 (LoRA Fine-Tuned)|

Ba chuyển dịch “kiến tạo mảng” đã xảy ra:

1. **Yếu tố Trung Quốc**: đội DeepSeek được đồn là đạt 99% chất lượng GPT‑4 với 1/50 chi phí bằng cách kết hợp:
    
    - Quantization-aware training
        
    - Dynamic sparse attention
        
    - State-sponsored GPU access
    
        
2. **Chênh lệch phần cứng (hardware arbitrage)**: mã nguồn mở cho phép developer tận dụng phần cứng rẻ hơn:
    
    - Consumer GPUs (RTX 4090s @ 0.12/kWhvscloudA100s@0.12/kWhvscloudA100s@1.10/kWh)
        
    - CPU inference nhờ tối ưu GGUF
        
    - Shared GPU pools (Petals, Together)
        
3. **Cuộc cách mạng Mixture-of-Experts**: các mô hình như Mixtral 8x7B dùng kích hoạt tham số có điều kiện để giảm chi phí suy luận 4–6x mà không mất chất lượng.
    

---

## Giai đoạn 3: Thời kỳ “hàng hoá hoá” (2024-nay)

Thị trường hôm nay giống “cloud wars” thập niên 2010 — biên lợi nhuận bị bóp nghẹt đã trở thành chuyện sống còn:

```javascript
// Switching costs dropped to near-zero
const providers = [openai, anthropic, google, deepseek];
const cheapestProvider = providers.sort((a,b) => a.pricePerToken - b.pricePerToken)[0];

// Developers now route traffic algorithmically
app.post('/chat', async (req, res) => {
  const response = await cheapestProvider.generate(req.body.prompt);
  res.send(response);
});
```

**Oligopoly Under Siege**

- **Thế khó của OpenAI**: GPT-4o Mini với giá $0.02/M được đồn là đang chạy ở mức **-35% margin** để giữ thị phần
    
- **Sai lầm của Anthropic**: giá Claude 3 ($15/M input tokens) dẫn tới 72% developer rời bỏ theo dữ liệu Artificial Analysis
    
- **“Phương án hạt nhân” của Google**: Gemini 1.5 Flash hạ giá tất cả xuống $0.0075/M nhờ hiệu quả TPU v5e
    

Startup đang tận dụng sự hỗn loạn này bằng:

- **Model Roulette**: tự động chuyển API như Unify.ai
    
- **Siêu tối ưu suy luận (inference)**:
    
```rust
// Techniques squeezing 2-3x more tokens/sec
    fn optimize_inference(model: &mut Graph) {
      model.apply(operator_fusion()); // Combine GPU ops
      model.apply(kv_cache_quantization(8bit)); 
      model.apply(speculative_decoding(5x));
    }
```

- **Vùng xám pháp lý**: các mô hình NSFW/tài chính né lệnh cấm TOS của cloud
    
---

## Kỷ nguyên hậu-mô-hình (post-model)

Khi LLM trở thành “tiện ích” (utility), bốn chiến trường mới nổi lên:

1. **Chiến tranh độ trễ (latency)**
    
    - Phản hồi dưới 100ms cho ứng dụng thời gian thực
        
    - Xử lý theo lô ở mức $0.0001/trang
        
2. **Sụp đổ ngữ cảnh (context collapse)**
    
    - Cửa sổ 10M token cho phép “cả công ty làm context”
        
    - Mô hình tích hợp truy hồi (RAG 3.0)
        
3. **Hệ sinh thái agent**
    
    - “Công nhân” AI giá $0.01/giờ:
        

4. **Chiếm lĩnh bằng quy định (regulatory capture)**
    
    - Lobbying cho tiêu chuẩn “Safety Compliance” có lợi cho kẻ dẫn đầu
        
    - Hosting mô hình đạt chuẩn HIPAA/GDPR
        

_“OpenAI đang pivot sang sản phẩm vì dẫn đầu mô hình trở thành gánh nặng. Nhưng khi mọi sản phẩm chỉ là một React frontend bọc quanh cùng 10 mô hình, ‘moat’ nằm ở đâu?”_

---

**Playbook mới cho developer**

1. Coi LLM là hàng hoá có thể thay thế
    
2. Thiết kế kiến trúc để linh hoạt theo mô hình (load balancer, fallback provider)
    
3. Tận dụng chênh lệch giá theo vùng (chi phí GPU ở Ấn Độ thấp hơn Silicon Valley ~40%)
    
4. Chuẩn bị cho suy luận ở mức $0.000001/token nhờ điện toán quang học dựa trên photon (Lightmatter, Luminous)
    

Thời kỳ tôn thờ “mô hình càng to càng tốt” đã qua rồi. Biên giới tiếp theo? Xây công cụ sống khỏe trong một hệ sinh thái nơi trí tuệ rẻ hơn cả RAM :P


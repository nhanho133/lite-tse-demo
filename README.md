# LiteTSE demo

Live demo: https://nhanho133.github.io/lite-tse-demo/

Phát audio thật từ 4 case tách giọng mục tiêu (target speaker extraction) bằng
[LiteTSE](https://github.com/nhanho133) — mixture nhiễu, output sau TSE, và groundtruth sạch,
nghe trực tiếp cạnh nhau. Kèm WER đo bằng whisper `tiny.en`.

## Cách đo WER

Không có transcript gốc đi kèm 4 case test này (tên case theo dạng
`<target-utt>_<interferer-utt>_sN_coreN`, khớp format LibriSpeech `speaker-chapter-utterance`).
Vì vậy `2_target_groundtruth.wav` của mỗi case — giọng mục tiêu sạch, không trộn nhiễu, không
qua TSE — được đưa qua whisper `tiny.en` để lấy transcript dùng làm tham chiếu. WER của
`mixture` và `tse-output` được tính so với transcript đó.

Đây là **proxy, không phải WER chuẩn tuyệt đối**: lỗi của whisper trên chính groundtruth cũng
lẫn vào cả hai phía, nhưng phần so sánh tương đối (TSE cải thiện bao nhiêu so với mixture thô)
vẫn phản ánh đúng câu hỏi đang được trả lời.

| case | WER mixture (không TSE) | WER sau LiteTSE |
|---|---|---|
| 237-134500-0000 / 7021-79740-0002 (core0) | 105.88% | **47.06%** |
| 7176-88083-0010 / 121-127105-0022 (core0) | 18.75% | **12.50%** |
| 7729-102255-0042 / 121-121726-0005 (core0) | 133.33% | **66.67%** |
| 7729-102255-0042 / 121-121726-0005 (core20) | 133.33% | **66.67%** |
| **gộp (pooled, 39 ref words)** | **74.36%** | **35.90%** |

## CPU-only stability (laptop thật, AMD Ryzen 7 7735HS, 16 lõi)

- RTF trung bình **0.147** (8 thread) — nhanh hơn real-time ~6-7 lần.
- Không NaN/Inf, không memory leak qua 300 vòng enroll+stream+finish liên tục (RSS/thread/fd
  không có xu hướng tăng).
- Idle CPU **0%** khi model đã load nhưng không xử lý — phù hợp chạy nền kiểu app desktop.
- Ở xung nhịp thấp hơn (2.2GHz, tiêu biểu cho big-core mobile tầm trung, đo ghim cứng tần số
  bằng `cpupower`, không qua cgroup để tránh nhiễu governor): RTF trung bình vẫn <1 (0.54),
  nhưng **peak latency một chunk chạm 826ms** — vượt ngưỡng 500ms của model, đáng chú ý khi
  tích hợp vào thiết bị không có xung nhịp cao như desktop.

## Nguồn

Audio và số liệu lấy từ `test/case_*` và `test/package/` trong repo chính (không public ở đây).
Trang này chỉ là bản showcase tĩnh, không chạy inference trực tiếp trên browser.

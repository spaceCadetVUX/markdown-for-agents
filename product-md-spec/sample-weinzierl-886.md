---
type: product
title: Cổng giao tiếp KNX Modbus RTU 886 Weinzierl
description: Weinzierl 886 là gateway KNX Modbus RTU 250 kênh, chạy được master/slave qua RS-485, cách ly galvanic, cấu hình qua ETS. Tư vấn kỹ thuật tại KNXStore.vn.
canonical: https://knxstore.vn/products/weinzierl-cong-giao-tiep-he-thong-knx-va-modbus-rtu
lang: vi
brand: Weinzierl
sku: "5256"
image: https://api.knxstore.vn/media/55420abe515413c5f962c003236fb769af67d19e769cc05a64b9ea9921819833.jpg
published: 2026-07-17
price: 13880000
currency: VND
---
# Cổng giao tiếp KNX Modbus RTU 886 Weinzierl

[Trang chủ](https://knxstore.vn/) > [Phụ kiện cho hệ thống](https://knxstore.vn/thiet-bi-he-thong) > Cổng giao tiếp KNX Modbus RTU 886 Weinzierl

| | |
|---|---|
| Thương hiệu | [Weinzierl](https://knxstore.vn/brands/weinzierl) |
| Danh mục | [Phụ kiện cho hệ thống](https://knxstore.vn/thiet-bi-he-thong), [Thiết bị mở rộng](https://knxstore.vn/thiet-bi-he-thong-knx-bo-mo-rong) |
| Giải pháp | [KNX](https://knxstore.vn/solution/knx) |
| Giá | 13.880.000 đ (giá đã bao gồm VAT) |
| Xuất xứ | Germany |

## Mô tả

### Gateway KNX Modbus RTU Weinzierl 886: Tích Hợp RS-485 Vào Hệ KNX

Bạn có một tủ điện với biến tần, PLC hay đồng hồ đo năng lượng chỉ nói được Modbus RTU, trong khi toàn bộ hệ điều khiển tòa nhà đã chạy trên KNX. Viết driver riêng thì tốn thời gian, mà bỏ qua dữ liệu Modbus thì màn hình giám sát KNX của khách hàng thiếu mất một mảng thông tin quan trọng. Đây đúng là bài toán mà Weinzierl thiết kế KNX Modbus RTU Gateway 886 để giải quyết.

![](https://knxstore.vn/media/ab0267e97342871ef1adff6d431e191ca2ffc093f44d51c507e0f028c1871b88.jpg)

### Vì Sao Cần Một Gateway Riêng Cho KNX Và Modbus RTU

KNX và Modbus RTU là hai thế giới khác nhau ở tầng vật lý lẫn tầng dữ liệu. KNX TP truyền theo cơ chế event-based với group address, còn Modbus RTU là giao thức master/slave polling theo register, phổ biến trên biến tần, PLC, DDC và thiết bị đo lường công nghiệp. Hai giao thức không tự nói chuyện được với nhau nếu không có thiết bị trung gian dịch địa chỉ và kiểu dữ liệu.

Weinzierl 886 đóng vai trò lớp dịch đó. Thiết bị lắp DIN rail 1 module 18mm, có cách ly galvanic hoàn toàn giữa hai phía KNX và Modbus, nghĩa là sự cố điện hay nhiễu ở phía Modbus không lan sang bus KNX và ngược lại. Cấu hình toàn bộ 250 kênh thực hiện trực tiếp trong ETS, không cần cài thêm phần mềm hay tool trung gian nào khác.

### Kiến Trúc Master Và Slave Của Weinzierl 886

Điểm dễ gây nhầm lẫn nhất khi mới tiếp cận thiết bị này là vai trò Modbus của nó không cố định, mà được chọn khi cấu hình tham số KNX Gateway trong ETS.

#### Chế độ Modbus Master

Ở chế độ master, gateway 886 chủ động gửi request đọc/ghi đến tối đa 25 thiết bị slave trên đường RS-485. Đây là chế độ dùng phổ biến nhất trong thực tế: gateway polling dữ liệu từ biến tần, đồng hồ điện, cảm biến Modbus, rồi đẩy giá trị lên KNX bus, hoặc nhận lệnh từ KNX rồi ghi xuống register của slave.

Ở chế độ này có hai tham số quan trọng cần hiểu đúng trước khi lên kế hoạch polling: Time till next request (khoảng cách tối thiểu giữa các request, dùng để làm chậm tốc độ hỏi của master) và Time till next cycle (thời gian tối đa cho một vòng polling toàn bộ kênh). Nếu Time till next cycle đặt quá ngắn so với số kênh cấu hình, vòng polling không kịp hoàn thành, LED RTU sẽ báo đỏ nhấp nháy để cảnh báo.

#### Chế độ Modbus Slave

Ở chiều ngược lại, gateway có thể đóng vai trò slave, nghĩa là một hệ thống Modbus master khác (ví dụ SCADA, PLC trung tâm) sẽ chủ động gửi request đến 886 để đọc hoặc ghi giá trị KNX thông qua register. Chế độ này phù hợp khi hệ BMS cấp cao hơn dùng Modbus làm giao thức tích hợp chính, và KNX chỉ là lớp điều khiển hiện trường.

Dù chọn vai trò nào, việc gán slave address, function code và địa chỉ register vẫn thực hiện hoàn toàn qua tham số ETS trên từng trang cấu hình, mỗi trang gộp 10 kênh.

### 250 Kênh Cấu Hình Qua ETS, Không Cần Phần Mềm Riêng

Toàn bộ việc mapping giữa group object KNX và register Modbus nằm trong tham số ETS, chia thành các trang, mỗi trang 10 kênh.

#### Datapoint Type hỗ trợ

Mỗi kênh gán một Datapoint Type xác định kiểu dữ liệu và group object tương ứng:

| DPT | Mô tả | Kiểu register Modbus |
|---|---|---|
| DPT 1 | Binary, 1 bit | Bit register / bit trong word register / value trong word register |
| DPT 3 | Dimming, 4 bit | Bit register / value trong word register |
| DPT 5 | Percent hoặc unsigned, 1 byte | Word register (low/high byte hoặc configured) |
| DPT 6 | Signed, 1 byte | Word register |
| DPT 7 | Unsigned hoặc configured, 2 byte | Bit register / bit trong word register / value trong word register |
| DPT 8 | Signed, 2 byte | Word register (high/low byte) |
| DPT 9 | Float, 2 byte | Word register, hỗ trợ scaling min/max |
| DPT 14 | Float, 4 byte | Word register hoặc double word register (2 register liên tiếp) |

Với DPT 14 dùng double word register, cần chọn đúng Word order (Hi word trước hay Lo word trước tại địa chỉ) và Type Register value (integer unsigned, 2's complement, hoặc float IEEE), tùy theo cách slave Modbus của bạn công bố trong datasheet.

#### Multi read request, tối ưu tải bus

Khi gateway chạy chế độ master, tham số Multi read requests cho phép gộp tối đa 16 kênh liền địa chỉ, cùng slave address và cùng function code, thành một request duy nhất thay vì gửi từng request riêng lẻ. Ví dụ 5 kênh đọc holding register liên tiếp từ địa chỉ 0 đến 4 sẽ gộp thành một request read-multi-holding-register duy nhất.

Đây là điểm đáng chú ý khi thiết kế mapping: sắp xếp các kênh cùng slave theo địa chỉ liên tục ngay từ đầu sẽ giúp gateway tự động gộp request, giảm đáng kể thời gian một vòng polling, đặc biệt quan trọng khi số slave gần chạm mức tối đa 25.

### Ví Dụ Cấu Hình Thực Tế Một Kênh Trong ETS

Để hình dung rõ hơn, đây là một ví dụ cấu hình thực tế trên trang tham số Datapoints 1-10 của ETS, dùng cho Channel 1.

![](https://knxstore.vn/media/49d2153591898b0a9e75ea33d51f5c452220f998f41f70974648965a684d70ca.jpg)

Trước tiên, mục Slave address type ở đầu trang cho chọn Common (dùng chung slave address đã khai báo ở Modbus settings) hoặc For this page (khai riêng một slave address chỉ áp dụng cho 10 kênh của trang này). Với dự án chỉ có một slave duy nhất, để Common là đủ; với dự án nhiều slave khác baudrate hoặc địa chỉ khác nhau, chọn For this page cho từng nhóm kênh tương ứng.

![](https://knxstore.vn/media/694ecd0f377373ef28144fd81835f6a992c5301f953b60e198e08ff95e0f148a.jpg)

Với Channel 1 trong ảnh, cấu hình như sau:

![](https://knxstore.vn/media/43497157f6f1482d451f84c00cc1b5131786cb36480b0d9076c45ce5990dac68.jpg)

- Datapoint type: DPT 01 - binary - 1 bit, nghĩa là group object KNX của kênh này chỉ có 2 trạng thái 0/1
- Direction: Modbus to KNX, tức chiều đọc dữ liệu từ Modbus rồi đẩy lên KNX bus (không phải điều khiển ngược lại)
- Send condition: On change, group object chỉ gửi telegram khi giá trị đổi, không gửi liên tục theo chu kỳ, giúp giảm tải bus KNX
- Type: Bit register, tức gateway đọc thẳng 1 bit từ Modbus, không phải bit nằm trong word register
- Value inverted: No, giá trị coil giữ nguyên, không đảo ngược khi map sang KNX
- Function: Read coils - 01, function code Modbus chuẩn để đọc coil (register 1 bit, read/write)
- Address: 0, tức coil đầu tiên trên slave (tương ứng địa chỉ Modbus 00001 nếu slave công bố theo kiểu 1-based)
- Polling interval: Every cycle, gateway hỏi giá trị này ở mọi vòng polling, không bỏ cách chu kỳ nào

Với cấu hình này, mỗi khi coil đầu tiên của slave đổi trạng thái (ví dụ tiếp điểm báo chạy/dừng của biến tần), gateway sẽ phát hiện ở vòng polling gần nhất và gửi ngay giá trị 0/1 tương ứng lên KNX bus. Đây là kiểu mapping đơn giản và phổ biến nhất, phù hợp cho tín hiệu trạng thái nhị phân như chạy/dừng, báo lỗi, cảnh báo ngưỡng.

### Lưu Ý Lắp Đặt Và Commissioning Thực Tế

Gateway 886 không có điện trở terminate tích hợp sẵn. Với đường truyền Modbus RS-485, bắt buộc lắp điện trở 120 Ohm / 0.25W tại thiết bị cuối cùng của tuyến truyền, đấu trực tiếp giữa hai dây tín hiệu trước khi vào input của thiết bị đó. Bỏ qua bước này thường không gây lỗi ngay lập tức, nhưng khiến communication không ổn định khi tuyến dài hoặc có nhiều slave, dẫn đến lỗi timeout ngắt quãng rất khó debug tại hiện trường.

Một số điểm khác cần lưu ý khi ra công trình:

- Chỉ dùng cáp xoắn có chống nhiễu (shielded and twisted) cho đường Modbus, không dùng dây thường
- Nếu cấu hình register address = 0 với chế độ "1 based", đây là lỗi tĩnh khiến kênh bị vô hiệu hóa ngay, LED RTU sẽ báo đỏ
- Cơ chế retry mặc định: nếu slave không phản hồi trong 1 giây, gateway thử lại 2 lần, sau đó bỏ qua toàn bộ kênh của trang tham số đó cho đến vòng polling kế tiếp
- Nếu mất nguồn KNX bus, gateway ngừng hoạt động hoàn toàn, kể cả phía Modbus, vì logic điều khiển nằm trên vi xử lý được cấp nguồn từ bus KNX

Về phần đồng bộ thủ công, hai nút bấm Button KNX và Button RTU trên mặt thiết bị cho phép ép gateway gửi lại toàn bộ giá trị hiện tại theo một chiều nhất định, rất hữu ích khi cần đồng bộ lại hệ thống sau khi mất điện hoặc sau khi thay thế một slave.

### So Sánh Với Các Gateway KNX Hoặc Modbus Sang Hệ Khác

|  | Weinzierl 886 | [Kanonbus KBA1002/KBA1004](https://knxstore.vn/products/bo-chuyen-doi-rs485-sang-bacnet-ip-modbus-tcp-kanonbus-kba1002) | [Kanonbus KBA2001](https://knxstore.vn/products/bo-chuyen-doi-knx-sang-bacnet-ip-modbus-tcp-kanonbus-kba2001) |
|---|---|---|---|
| Hướng tích hợp | Modbus RTU vào KNX (và ngược lại) | RS485 vào BACnet/IP, Modbus TCP | KNX ra BACnet/IP, Modbus TCP |
| Kết nối vật lý | RS-485 | RS-485 (2 hoặc 4 cổng) | RJ45 |
| Có KNX | Có | Không | Có |
| Vai trò Modbus | Master hoặc Slave | Master hoặc Slave | Master hoặc Slave |
| Phù hợp khi | Thiết bị Modbus RTU cần vào cùng hệ điều khiển KNX | Tích hợp RS485 vào BMS lớn, không cần KNX | Đưa dữ liệu KNX ra BMS qua Ethernet |

Nếu bài toán là mang biến tần, PLC hay đồng hồ đo Modbus RTU vào chung một hệ điều khiển KNX, Weinzierl 886 là lựa chọn trực tiếp nhất. Nếu ngược lại, dự án cần đưa toàn bộ hệ KNX ra một BMS cấp cao hơn qua Ethernet, KBA2001 phù hợp hơn vì có sẵn cổng RJ45 và không giới hạn ở RS-485.

### Nên Chọn Weinzierl 886 Khi Nào

Nên chọn khi:

- Dự án có thiết bị Modbus RTU (biến tần, PLC, DDC, đồng hồ đo năng lượng) cần điều khiển hoặc giám sát chung với hệ KNX
- Cần một gateway chạy được cả hai chiều, vừa polling slave Modbus vừa cho phép hệ Modbus khác đọc dữ liệu từ KNX
- Muốn cấu hình hoàn toàn trong ETS, không muốn phụ thuộc thêm một phần mềm hay license riêng
- Số điểm dữ liệu không vượt quá 250 kênh và không quá 25 slave trên một gateway

Không nên chọn khi:

- Thiết bị đích dùng Modbus TCP thay vì Modbus RTU, khi đó cần gateway có cổng Ethernet như Kanonbus KBA1002/KBA1004
- Dự án cần đưa KNX ra BACnet/IP cho BMS cấp cao, trường hợp này Kanonbus KBA2001 phù hợp hơn
- Số điểm dữ liệu vượt 250 kênh hoặc cần polling hơn 25 slave trên một gateway, khi đó nên chia thêm gateway thứ hai hoặc cân nhắc giải pháp PLC trung gian

### Kết Luận

Weinzierl KNX Modbus RTU Gateway 886 giải quyết gọn bài toán tích hợp thiết bị Modbus RTU vào hệ điều khiển KNX mà không cần viết driver riêng hay thêm phần mềm trung gian. 250 kênh cấu hình linh hoạt, vai trò master/slave chọn được theo nhu cầu dự án, và cơ chế multi read request giúp tối ưu tải bus khi số slave tăng lên.

Nếu bạn đang spec gateway KNX Modbus RTU cho dự án cụ thể hoặc cần tư vấn cách mapping DPT phù hợp với slave Modbus hiện có, liên hệ team kỹ thuật KNXStore.vn để được hỗ trợ trực tiếp.

## Thông số kỹ thuật

| Thông số | Giá trị |
|---|---|
| Nguồn cấp | KNX Power 21V~30V DC |
| Nguồn cấp phía Modbus | 12...24V DC, cách ly hoàn toàn với KNX bus |
| Baud rate | 1200 / 2400 / 4800 / 9600 / 19200 / 38400 / 56000 / 115200 bps |
| Số slave tối đa (chế độ master) | 25 |
| Số Datapoint cấu hình | 250 |
| Vai trò Modbus | Master hoặc Slave |
| Parity | Even, Odd, None (1 hoặc 2 stop bit) |
| Register addressing | 0 based hoặc 1 based |
| Function code | 01, 02, 03, 04, 05, 06, 15, 16 |
| Địa chỉ vật lý mặc định | 15.15.255 |
| Cách ly an toàn | Galvanic isolation KNX - Modbus |

## Tài liệu

- [weinzierl-886-knx-modbus-rtu-gateway-5256-manual-en.pdf](https://knxstore.vn/media/docs/3ed9a04c2db48cef2c6ee9d46b39d315eebcceacad345f9dbaec4db0f60093fb.pdf) — PDF, 1,3 MB

## FAQ

- **Weinzierl 886 có cần nguồn ngoài không?** Có, bắt buộc. Phía Modbus của gateway cần nguồn 12-24V DC riêng, cách ly hoàn toàn với KNX bus. Không được dùng điện áp chưa lọc (unchoked output) từ nguồn KNX cho mục đích này, vì có thể gây nhiễu hoặc hỏng thiết bị.
- **Gateway 886 chạy được cùng lúc cả master và slave không?** Không. Vai trò Modbus (master hoặc slave) được chọn một lần trong tham số ETS cho toàn bộ thiết bị, không thể chạy song song cả hai vai trò trên cùng một gateway.
- **Cần điện trở terminate ở đâu khi lắp Weinzierl 886?** Điện trở 120 Ohm / 0.25W lắp tại thiết bị cuối cùng trên tuyến truyền RS-485, đấu trực tiếp giữa hai dây tín hiệu A và B trước input của thiết bị đó. Bản thân gateway 886 không có sẵn điện trở terminate.
- **Nếu mất kết nối Modbus, KNX bus có bị ảnh hưởng không?** Không. KNX bus và Modbus cách ly galvanic hoàn toàn. Mất kết nối phía Modbus chỉ khiến các kênh liên quan báo lỗi "No communication" qua diagnostic object, không ảnh hưởng đến hoạt động của các thiết bị KNX khác trên bus.
- **Có thể đọc giá trị đo lường dạng thập phân (nhiệt độ, điện năng) qua gateway này không?** Được. Gateway hỗ trợ DPT 9 (float 2 byte) và DPT 14 (float 4 byte) với cơ chế scaling tuyến tính giữa giá trị register và giá trị KNX, phù hợp cho dữ liệu đo lường analog như nhiệt độ, điện áp, công suất từ đồng hồ đo hoặc cảm biến Modbus.

## Hình ảnh

- [Ảnh 1 trên 4](https://knxstore.vn/media/55420abe515413c5f962c003236fb769af67d19e769cc05a64b9ea9921819833.jpg) — 1000×1000
- [Ảnh 2 trên 4](https://knxstore.vn/media/c33a212a9842ebf4689bb7de7ac710407857f1073afb32725add81c13305bbe7.png) — 600×200
- [Ảnh 3 trên 4](https://knxstore.vn/media/a9f702604ad16a1f2fcc613faba8dedf240b4bc585757dad863d1fd40f721eed.jpg) — 1000×834
- [Ảnh 4 trên 4](https://knxstore.vn/media/098ff204d6b99ee9b6f4e23144947faed599e852390695386d5c76a76350f2c3.jpg) — 1000×563

## Sản phẩm đề xuất

- [Bộ mở rộng DALI-2 3 sub-line nguồn 50mA Lunatone DALI-2 Expander 3](https://knxstore.vn/products/bo-mo-rong-dali-2-3-sub-line-nguon-50ma-lunatone-dali-2-expander-3) — Lunatone · Art.Nr. 89452937-3 · Liên hệ
- [Nguồn Cấp Bus DALI-2 Lắp Trần Lunatone DALI PS2](https://knxstore.vn/products/nguon-cap-bus-dali-2-lap-tran-lunatone-dali-ps2) — Lunatone · Art.Nr. 24033444-2 · Liên hệ
- [Bộ ghép nối KNX Line Coupler / Repeater Kanonbus KTELC](https://knxstore.vn/products/bo-ghep-noi-knx-line-coupler-repeater-kanonbus-ktelc) — Kanonbus · KTELC · Liên hệ
- [Bộ định tuyến KNX IP Router Kanonbus KTEIPC](https://knxstore.vn/products/bo-dinh-tuyen-knx-ip-router-kanonbus-kteipc) — Kanonbus · KTEIPC · 11.639.000 đ
- [Bộ chuyển đổi giao thức điều khiển DALI - DMX512 64 địa chỉ Dalcnet DGD Gateway](https://knxstore.vn/products/bo-chuyen-doi-giao-thuc-dieu-khien-dali-dmx512-64-dia-chi-dalcnet-dgd-gateway) — DALCNET · DGD · 5.666.760 đ
- [Gateway điều khiển điều hòa VRF đa thương hiệu tích hợp BMS qua RS485 Micro Air](https://knxstore.vn/products/gateway-dieu-khien-dieu-hoa-vrf-da-thuong-hieu-tich-hop-bms-qua-rs485) — Micro Air · MicroAir B25 · Liên hệ
- [Module Công Tắc KNX EA 36.32 Chuyển Đổi Tín Hiệu Hugo Müller](https://knxstore.vn/products/module-cong-tac-knx-ea-36-32-chuyen-doi-tin-hieu-hugo-muller) — HUGO MULLER · EA 36.32 · Liên hệ
- [Gateway điều khiển điều hòa VRF đa thương hiệu tích hợp BMS qua RS485 Micro Air](https://knxstore.vn/products/gateway-dieu-khien-dieu-hoa-vrf-da-thuong-hieu-tich-hop-bms-qua-rs485-micro-air) — Micro Air · B25 · Liên hệ

## Đặt hàng / báo giá

- Gửi yêu cầu báo giá: hotline [0918.918.755](tel:0918918755)
- Zalo: https://zalo.me/0918918755
- Liên hệ tư vấn dự án: https://knxstore.vn/contact

CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH · MST 0311846236 · support@knxstore.vn

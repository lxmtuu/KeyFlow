# Nhật ký thay đổi · Keyflow

> **English version: [CHANGELOG.en.md](CHANGELOG.en.md)** · Bản dưới đây là bản gốc tiếng Việt. Hai tệp ghi
> cùng một nội dung và **cùng một danh sách phiên bản**; `tools/check_sources.py` (`scan_release_version`)
> đối chiếu danh sách đó của cả hai với `<Version>` trong `PianoPath.csproj`, nên không bản nào lệch khỏi
> bản kia và một mục mới không thể nằm trong repo mà không nằm trong bản phát hành.

Mọi thay đổi người dùng nhìn thấy được ghi ở đây, theo [Keep a Changelog](https://keepachangelog.com/vi/1.1.0/)
và [semantic versioning](https://semver.org/lang/vi/).

Số phiên bản chỉ được viết xuống **một** chỗ trong mã nguồn — `<Version>` của `PianoPath.csproj` — và ứng
dụng tự đọc nó qua `AppInfo.Version` (nhãn phiên bản của menu khởi động, hộp thoại About, báo cáo
`--bench`). Những bản sao không đọc được assembly lúc chạy (`installer\Keyflow.iss`, ví dụ `git tag` và
`/DAppVersion=` trong hai README, mục mới nhất của hai tệp này) được `scan_release_version` ghim vào cùng
một số: quên một bản sao là kiểm tra tĩnh đỏ, không phải một bản phát hành nói hai số phiên bản.

Giới hạn kỹ thuật của sản phẩm nằm ở mục *[Giới hạn hiện tại](README.md#giới-hạn-hiện-tại)* của README phát hành
và không được lặp lại ở đây; tài liệu kế hoạch phát triển nội bộ không nằm trong kho phát hành công khai.

## Unreleased

## 2.1.1 — 2026-10-16

### Phát hành gói bất biến đúng quy trình
- **Bản tải xuống đầu tiên của đợt 2.1:** bao gồm toàn bộ tính năng hiệu ứng/AR/Settings liệt kê ở 2.1.0 bên dưới. Release nguồn 2.1.0 đã bị publish khi chưa có asset và bị immutable khóa; không có gói tải, không đổi/xóa/tái sử dụng tag đó.
- Dùng duy nhất sự kiện tạo tag để tránh hai lượt publish. Workflow tạo draft, upload đủ gói rồi publish; không tắt immutability và không cố ghi đè release bất biến.

## 2.1.0 — 2026-10-16

### Settings, hiệu ứng hạt và xuất AR
- **Settings mới gồm 12 nhóm** theo bố cục đã duyệt: Nốt & màu → Nhấn phím → Giữ phím → Thả phím → Ánh sáng; thêm Nâng cao, tìm VI/EN không dấu và alias cho 13 trang cũ. Preset/share code, camera/calibration, Audio/MIDI, luyện tập/lịch sử, ghi hình, theme và hồ sơ vẫn giữ handler/ID cũ. Stage/AR trên header là ngữ cảnh cài đặt, không tự đổi camera hay output.
- **Hiệu ứng hạt nâng cấp** là opt-in, mặc định tắt cho look cũ. Wisps/Flames và vật liệu nhả Float Up/Dissolve/Smoke dùng chung cho GPU Live Stage và AR; AR chia cap 192 theo vai trò/nhóm nốt. Thêm preset Prism Flow - Wisps / Prism Flow - Flames. Renderer phần mềm giữ classic.
- **Xuất AR offline giữ vòng đời hạt liên tục** trong mỗi take thay vì tạo lại simulation mỗi frame; dùng clock đã lập kế hoạch và substep tối đa 50 ms. Chính sách camera-file nửa tốc độ khung hình cũ vẫn giữ; không tuyên bố sửa mọi trường hợp đồng bộ A/V hoặc đã thử webcam/hardware ARM.

### AR overlay
- **AR render-to-video — giao diện xuất (S7 trọn vẹn)**: thẻ mới **AR RENDER‑TO‑VIDEO** ở trang Recording — một nút RENDER AR TAKE… render offline bài hát đang mở trên video camera đã hiệu chỉnh: không đồng hồ live, bước thời gian cố định, đúng look máy đang dùng (lớp AR, watermark kind‑16, watermark… mọi lớp), tiếng piano mix khớp tick ra `.wav` cạnh AVI (độ phân giải/fps theo lựa chọn OUTPUT; Record audio + Watermark đều tôn trọng), kèm dòng ffmpeg ghép khi xong. Song → script sự kiện làm code thuần (`ArVideoExportScript`, xUnit `SongExportScriptTests`); verify chạy đường thật với MP4 do chính app ghi + reader decode: sọc sáng của file tăng đều theo từng tick của engine.
- **AR render-to-video — phần watermark**: dải «KEYFLOW» của toggle **Watermark** giờ do chính GPU in lên khung hình (sprite kind 16 + texture `MarkTex`), nên bản ghi GPU và take offline đều mang đúng dải mà live stage đang hiển thị; placement (giữa màn, đúng 18 DIP trên đường bàn phím, chỉ co lại khi khung hình hẹp) là code thuần có xUnit. Verify so byte: cùng một tick có/không watermark khác đúng và chỉ ở vùng strip.
- **AR render-to-video — phần giọng**: mỗi take offline giờ có thêm file `.wav` nằm cạnh AVI/PNG — engine mix piano theo đúng tick của video (sự kiện nét phát ra đúng khung hình nó xuất hiện, các block 44.1 kHz pace theo từng tick nên tổng số mẫu khớp sample-exact kể cả ở fps lẻ như 13), take không nét cho file im lặng đúng dài. Kến trúc: hook tuỳ chọn `audio` (synth + script sự kiện + `WavWriter`) đi qua engine `Run`, caller sở hữu và đóng writer/synth. Verify dựng SFZ tổng hợp: 22050 mẫu cho đúng 6 khung 12 fps, âm lớn ở các tick giữ phím và gần-im ngay mục onset.
- **Rà soát phát hành:** toolbar bắt đầu tính idle từ frame đầu thay vì thời gian dựng cửa sổ; nút MORE của live-bar coverage không bị Nâng cao chặn. Pipeline public giữ nguyên workspace/gói build, dùng Git index riêng và lệnh PowerShell append hợp lệ; các cổng test/native của release vẫn bắt buộc.

- **Render-to-video AR — nửa khung** (roadmap §14.10, AR‑S7 phần đầu): móng cho việc xuất cảnh AR ra video offline. `Video/ArVideoRenderProfile.cs` (thuần) lập kế hoạch kích thước/fps/số khung với mọi clamp an toàn — hai cạnh chẵn đúng chuẩn AVI, trần 60 fps khớp cả hai writer hiện có, trần cạnh dài để camera quá nét không đốt GPU vô ích — kèm lựa chọn downsample 1/2/4 bằng trung bình hộp 2×2 (`Video/BgraPack.cs`, thuần). `Video/ArRenderToVideo.cs` là engine quét offline: với một script feed cho từng tick, mỗi khung xuất tiến cảnh đúng `1/fps` với seed cố định — không đồng hồ của riêng nó — nên hai lần quét cùng script ra **byte-identical**, và tick nào cũng khớp từng byte với một lượt render giống-hệt-loop-cứng-cùng-dt-cùng-seed (đúng ý «so khung live ↔ render» của roadmap). Writer dùng lại `AviVideoRecorder`/`PngSequenceRecorder` hiện có ở đúng kích thước/fps đã lên kế hoạch. Kiểm chứng: hai bộ xUnit thuần (`VideoRenderProfileTests`, `BgraPackTests`) và block `VerifyArRenderToVideo` trong `--verify` — take AVI 200×112@12fps đọc lại với sọc sáng bên trái camera cùng ánh phím C4 sáng đúng tick nhấn, take PNG hạ 320×180→160×90 sáu khung kèm manifest. Âm thanh bài hát (mixdown) và watermark phía GPU + giao diện xuất file ở phần sau của AR‑S7, chưa nằm trong mảng này.
- **Mã chia sẻ diện mạo lên bản 2 — look đi cùng graph** (roadmap §14.9): COPY CODE giờ phát hành mã `KEYFLOW-LOOK-2:…`, báo hiệu payload mang cùng look toàn bộ graph AR — Độ rộng nốt theo velocity, Fade-in nốt theo velocity, Độ mờ vệt nhấn theo vòng đời — nên máy nhận vẽ lại đúng các đường cong đó chứ không chỉ màu sắc/tỉ trọng. APPLY CODE **vẫn đọc được mọi mã bản 1** (`KEYFLOW-LOOK-1:…`) từng được chia, nguyên từng bit, vì phong bì JSON vốn đã trọn vẹn từ đầu; mã của phiên bản khác vẫn bị từ chối với đúng lỗi «được tạo bởi một phiên bản Keyflow khác». Ảnh nền và camera vẫn ở lại máy gửi như trước. Phần vận chuyển mã tách ra `Stage/ShareCodeEnvelope.cs` (thuần, không settings) để xUnit kiểm trên mọi máy: `ShareCodeEnvelopeTests` sáu fact (phát hành v2 URL-safe, mã bản 1 mở nguyên bit, lý do cho mọi mã hỏng, mã bị chat wrap vẫn mở, zip-bomb dừng ở mức giới hạn payload). Mức look/graph được block `VerifyPresetShareCodes` mở rộng khóa trong `--verify`.
- **Ngân sách hiệu năng AR: lớp phủ tự cắt họ hiệu ứng theo thứ tự cố định khi GPU quá ngân sách** (roadmap §14.8): `Camera/ArQualityBudget.cs` cắm vào bộ adaptive-quality sẵn có vốn đã đang theo dõi thởi gian hình của GPU. Tier xấu đi (Full → BloomReduced → BloomMinimal → AmbientReduced → AmbientMinimal), mỗi nấc mất đúng thêm một họ theo câu roadmap — hạt → pulse → note light → light bars → style nốt (note 3D sụp về lối vẽ phẳng; bản thân note vẫn còn) → impact glow — và trả lại khi áp lực giảm, không động đến setting nào. Phím, đốt và keypress nằm ngoài bảng, nên không hình nào quá ngân sách mà mất được lớp phủ. Khóa bằng bộ xUnit `ArQualityBudgetTests`, block `VerifyArQualityBudget` cùng một trận diễn tập WARP trong `VerifyArNoteProjection` (cut sâu: beam/pulse/impact/particle/note light đều vắng, key tint và vạch damper vẫn còn; về Full là đèn quay lại nguyên vẹn).
- **Hình ảnh AR tự tinh chỉnh theo khung nhìn camera** (roadmap §14.7, Thêm 5): `Camera/ArViewPresets.cs` — bảng thuần khóa theo id khung nhìn máy đã biết, không phải một setting mới. Camera nghiêng (45°, đánh dấu tự do, legacy) giữ nguyên độ cao siêu mỏng cứ catalogue (flare impact 0,2 cm trên phím, pulse tối thiểu 0,3 cm trên mặt phẳng phím) và các quad đứng dựng — hold arc, beam sáng — vẫn vẽ; khung nhìn **90° trực đỉnh** thì lift thu còn 0,05 cm / 0,1 cm (nhìn thẳng xuống, mặt phẳng phím chính là ảnh) còn các quad đứng bị giấu hẳn, vì từ trên cao chúng ép thành sợi nhiễu mỏng như tóc. Id khung nhìn lạ hoặc rỗng quay về hàng nghiêng, đúng cách calibration store chuẩn hóa cùng id đó, và preset/mã chia sẻ vẫn không mang theo gì — id khung nhìn thuộc máy. Khóa bộ xUnit `ArViewPresetTests` và block `VerifyArNoteProjection` mở rộng trong `--verify` (overhead: beam/arc bị giấu, impact light/pulse/particle vẫn chiếu nguyên vẹn trên WARP).
- **Mẹo đặt camera ngay trong cửa sổ Calibrate, kèm canh tay trong khung hình** (roadmap §14.7, Thêm 4): cửa sổ calibrate giờ nói luôn hai điều ai cũng quên lần đầu — *treo camera qua vai, nhìn dọc hàng phím* (đặt ngay trên đầu thì đọc được mọi phím; đặt ngang phím thì mất phím xa) và *tránh ống kính quá rộng* (ultra-wide làm hàng phím cong về mép ảnh, đánh dấu trên hàng cong không bao giờ bám phím thật) — kèm nhắc đánh dấu khi tay ở ngoài khung hình. Và cửa sổ tự canh: một trạm quan sát màu da theo lobe Y′CbCr quét still đang hiển thị, khi tỉ lệ màu da vượt ngưỡng — khắp ảnh trước khi đặt đủ góc, chỉ bên trong vùng phím sau khi đủ góc — dòng note khẽ nhắc đưa tay ra; đây là lời khuyên chứ không phải cửa chặn, và gỗ sồi nắng vẫn có thể mượn màu da mà không hại ai. Khóa bộ xUnit `CalibrationAdviceTests` và block `VerifyCalibrationAdvice` trong `--verify`.
- **Bản sao căn chỉnh — xuất/nhập mọi calibration của camera đang dùng** (roadmap §14.7, Thêm 3): hai nút mới trong thẻ KEYBOARD ALIGNMENT. **EXPORT CALIBRATION** gom mọi calibration đã lưu của camera (mọi khung nhìn, kích thước ảnh và cỡ phím) ra một tệp JSON `keyflow-ar-calibration` mang đi được, kèm ảnh PNG chụp frame hiện tại của camera đặt cạnh tệp để sau này nhìn là biết của camera nào (khôi phục không đọc ảnh). **IMPORT CALIBRATION** đọc một tệp như vậy, khôi phục từng hàng về đúng cameraLink của tệp (rỗng thì về camera đang dùng) và cho mỗi hàng đi qua đúng các kiểm tra của một lần calibrate mới — hàng không đạt được đếm và bỏ qua chứ không bao giờ được vẽ, còn tệp không đúng loại hay tệp của Keyflow mới hơn bị từ chối kèm lý do rõ ràng. Không thêm setting nào: preset và mã chia sẻ vẫn không hề mang calibration như trước. Khóa bằng bộ xUnit `ArCalibrationArchiveTests` và block `VerifyArCalibrationArchive` trong `--verify`.
- **Đo trôi tự động — AR tự phát hiện camera xê và nhắc đánh dấu lại** (roadmap §14.7, Thêm 2): toggle **Canh khung camera có trôi** trong thẻ KEYBOARD ALIGNMENT (mặc định bật, thuộc máy). Khi lớp phủ đang chạy với một calibration, cứ khoảng hai giây app chụp mẫu vùng phím trên ảnh và so với khung lúc bắt đầu canh: tìm dịch chuyển hay nhất theo ô 3 pixel có bù độ sáng và cắt phần nhiễu (tay chơi, bóng đổ không tính; đổi sáng ±18 % không báo nhầm). Phím trôi quá ngưỡng (~6 pixel ở khung 1280) trong 3 lần đo liền thì dòng trạng thái của thẻ AR nhắc *«Khung phím đã trôi khoảng N pixel — mở Calibrate và đánh dấu lại»* — đúng triết lý «nhắc ngưỡng dùng, không tự sửa»; canh cũng tự tạm nghỉ khi đang đo độ trễ vì hai bên dùng chung cột sọc khung camera. Bảo chứng bởi xUnit `ArDriftMeterTests` (frame tổng hợp: trôi 9 px đo được ±3 px, ổn định/bàn tay/ánh sáng không báo động) cùng block `VerifyArDriftWatch` trong `--verify`.
- **Một cú bấm, một ánh sáng — và ánh sáng sống động hơn**: trước đây, ở hướng nốt rơi xuống, một lần nhấn phím phát sáng ngay lúc tay chạm phím *rồi lặp lại lần nữa* khi đầu nốt live rơi tới hàng phím — ánh lặp đôi dù tay không nhấn thêm. Nay hiệu ứng cháy đúng một lần theo ngón tay ở cả hai hướng nốt (nốt bài hát trượt qua vẫn lóe nhẹ lúc chạm phím như trước, và chế độ Nghe vẫn có nhịp đánh). Cùng đợt đánh sáng được đánh bóng: flash impact nở vào mượt (~55 ms) và tắt dần êm thay vì lóe/cắt; độ sáng và kích thước ánh sáng đi theo velocity rõ rệt hơn — chạm nhẹ là tàn lửa, đánh mạnh bung rộng (dải nhạc động rộng gấp ~3 lần); phím giữ lâu "thở" nhẹ — tint sáng dao động chậm vài phần trăm sau khoảng một giây, phím mới bấm vẫn sáng tĩnh. Không thêm cài đặt mới: vẻ mới áp ngay cho Live Stage lẫn lớp phủ AR vì cả hai đọc chung một pipeline sáng; bảo chứng bởi bộ kiểm `GpuStageSimulationLightCurvesTests` (xUnit) và ba block khẳng định mới trong `--verify` (một cú bấm = một impact, dải nhạc động velocity + bao đường flash, độ "thở" của tint phím giữ).
- **Vạch damper: tắt/bật tuỳ ý và màu tuỳ chọn; khung Alignment check chọn được màu**: thẻ **AR PLAYBACK** có thêm toggle **Damper strip** (tắt hẳn vạch sáng mép sau phím khi muốn hình dịu) và mục **Màu vạch damper** — *Theo look (halo)* giữ vạch trong gia đình màu halo của look như trước, *Tự chọn* tô vạch bằng một màu riêng (hữu ích khi màu halo của preset chìm trên nền phím thật); toggle và màu này thuộc look nên đi cùng preset/chia sẻ. Riêng khung **Alignment check** (thẻ KEYBOARD ALIGNMENT) chọn được một trong năm màu tương phản cao — Trắng (mặc định), Xanh lơ, Vàng, Hồng cánh sen, Xanh lá non — để khung luôn nổi trên ảnh camera sáng hay tối; màu khung thuộc máy giống chính chế độ kiểm tra. Cả ba công tắc ăn cho **mọi góc AR** (45°/90°/tự do) vì chung một đường vẽ qua calibration.
- **Alignment check — phát hiện camera xê dịch bằng một cái nhìn**: toggle mới trong thẻ **KEYBOARD ALIGNMENT** vẽ lại khung phím đã lưu (khung 4 cạnh cùng vạch mờ ở mỗi quãng tám) ngay trên ảnh camera trực tiếp, ở mọi bậc chất lượng, và pack sau cùng nên đọc được trên cả nốt lẫn hiệu ứng. Chân máy bị đụng hay bàn phím bị xê — các vạch không còn khớp phím thật là biết ngay phải calibrate lại. Chế độ này thuộc máy, đi cùng khung nhìn và calibration, nên preset hay mã chia sẻ không bao giờ bật nó trên máy khác; chỉ vẽ khi đã calibrated. Cùng đợt, bảng **độ cao catalogue** được khóa trong `ArElementHeights`: tâm lóe sáng impact nổi đúng 0,2 cm trên phím vừa đánh và xung pulse không bao giờ xuống dưới 0,3 cm — đủ để lóe sáng không lẩn sau mép phím ở góc máy nghiêng, mà mỏng tới mức khung nhìn 90° không thấy khác biệt (dưới mức một pixel, xUnit `ArAlignmentGuidesTests`/`ArElementHeightsTests` khóa hình học, WARP kiểm riêng block alignment).
- **Vạch damper phát sáng ôm mép sau phím** (catalogue `Dampers`, vẽ từ bậc AR Standard trở lên): một vạch sáng mềm dính liền mép sau của hàng phím — đúng «vạch đánh» sáng rực trong các video đàn quay thẳng từ trên xuống. Vạch luôn mỏng quanh mép; pedal sustain nhấn xuống (mức 0..1, không nhị phân) làm nó vươn sâu ra thân đàn và sáng dậy. Màu lấy **màu halo** của look (cùng gia đình với Pedal Glow trên sân khấu) nên tự hợp với mọi preset, bloom vẽ hào quang quanh lõi nóng, bậc Lite không vẽ — đúng catalogue. Cùng lúc, khung nhìn **90° overhead** (góc quay phổ biến nhất cho hình kiểu này) được `VerifyArNoteProjection` chiếu lại toàn bộ trên WARP — vệt nhấn, tint velocity, vạch damper — để CI khóa góc quay này theo từng build; kiểm xUnit `ArDampersTests` chạy trên mọi máy.

## 2.0.1 — 2026-10-12

### AR overlay

- **Vệt nhấn trực tiếp có thể chạy vào phím** (AR‑6.2, phần coverage): trong Cài đặt → Camera live, chevron **THÊM · ĐỘ PHỦ CỦA VỆT TRÊN PHÍM** (chỉ hiện khi bật vệt nhấn) mở ra thanh trượt **Độ phủ của vệt lên phím** (25–300 % chiều sâu phím): 100 giữ vẻ ngoài cũ — vệt dừng đúng mép phím; 200 chạm tới mép trước của phím trắng — **độ phủ toàn bộ** kiểu Keysight; lớn hơn thì tràn qua phím, nhỏ hơn thì vệt tách khỏi phím. Áp dụng được cho cả vệt phẳng lẫn vệt tự do trong không gian 3D (Standard/Ultra) và phần nào nằm ngoài phím vẫn đi theo hướng nốt đã chọn; mặc định 100 nên không ai mất vẻ ngoài quen thuộc. Bảo chứng bởi fact mới trong `ArNotesTests`/`Ar3DNoteProjectionTests` và bởi `--verify` (mở rộng `VerifyArGraphVariable`); thanh trượt thuộc máy, áp giao diện không mang theo.
- **Vệt nhấn trực tiếp mờ theo vòng đời** (AR‑6.2): trong Cài đặt → Camera live → "Thanh khi nhấn phím", mục mới **Độ mờ vệt nhấn theo vòng đời** với hai thanh trượt **Độ đậm lúc vừa nhấn** và **Độ đậm khi vươn hết tầm** cho vệt nhấn thay đổi độ đậm theo quãng nó đã vươn, thay vì chỉ mờ dần theo chiều dài như cũ. Khi phím còn giữ, độ mờ đi xuôi từ vạch bắt đầu tới vạch kết thúc (60 % → 100 % mặc định — vệt sáng dần khi dài ra); khi nhả phím, độ mờ đi ngược lại suốt pha nhả, và độ mờ pha nhả vốn có vẫn đảm bảo vệt biến mất hẳn. Áp dụng cho cả vệt phẳng lẫn vệt tự do trong không gian 3D (Standard/Ultra); mặc định **tắt**, nên hình ảnh cũ không đổi gì. Bảo chứng bởi `ArLiveFadeTests` cùng các fact mới trong `ArNotesTests`/`Ar3DNoteProjectionTests` và bởi `--verify` (mở rộng `VerifyArGraphVariable`).
- **Thẻ LIVE KEY RESPONSE nói đúng điều nó làm và giấu đúng hàng**: dòng mô tả đầu thẻ (Cài đặt → Camera live) từng gom chung tint phím, note bay và vệt nhấn thành một câu, rồi nói bừa rằng các núm velocity \"không đổi tint phím\"; nay nó kể rõ ba kênh độc lập — tô màu phím theo vài giây, note bay theo đồ hoạ đã chọn, vệt nhấn riêng — và ghi chú vệt theo tint velocity của look. Hàng **Độ mờ vệt nhấn theo vòng đời** cùng hai thanh trượt của nó nay chỉ hiện khi bật **Thanh khi nhấn phím**, và hai thanh trượt cũng chỉ hiện khi toggle độ mờ đang bật, để không còn điều khiển \"chết\" nào trong dock.
- **Độ trễ camera ghi nhớ theo từng camera** (AR‑3.5): `Camera delay` từng là một con số cho cả máy, trong khi mỗi camera trễ một lượng riêng (webcam USB ≈ 60–150 ms; camera ảo hay điện thoại làm webcam khác hẳn). Nay độ trễ được lưu theo định danh camera — cùng khoá với file calibration — trong `ar-delay.json` cạnh thư mục cài đặt: rỗi khỏi một camera thì giá trị thanh trượt được ghi lại cho camera đó, mở lại camera thì độ trễ của chính nó tự nạp lại cho cả thanh trượt lẫn engine, và kéo thanh trượt hay chấp nhận kết quả **MEASURE DELAY** đều ghi vào camera đang mở. Giá trị mới được kiểm bởi `ArDelayStoreTests` và `ArDelaySessionTests` trong project test xUnit chạy trên cả Linux lẫn Windows.

## 2.0.0 — 2026-10-10

### Giao diện & đồ hoạ

- **Lớp phủ AR (đang phát triển, chưa thử trên webcam thật)**: dùng camera làm nền toàn màn hình, đánh dấu 4 góc dải phím và chọn khung nhìn phù hợp. Calibration nay lưu FOV và pose đàn 3D pitch/yaw/roll; trong Calibrate có gizmo kiểu Unity Move/Rotate/Scale cùng thanh trượt số cho translation và scale đồng đều. Định dạng version 5 lưu đầy đủ pose/FOV/baseline; version 1–4 vẫn đọc với translation 0 và scale 1 (v3/v4 giữ FOV, v1/v2 dùng 70°; v4 giữ góc xoay, v1–v3 dùng 0°). Bật **Notes in perspective** để nốt thành quad camera-facing theo pose; hướng chạy nốt có mũi tên riêng trên preview, nút căn screen-up/theo chiều sâu phím và hai núm **Note direction tilt / azimuth** độc lập với transform bàn phím, đặt ngay sau toggle và trước **Minimum length, share of width**. **Live press bars** là toggle độc lập, mặc định tắt, không tắt key tint; vệt nốt phẳng của sân khấu bị suppress trên camera cả khi chưa calibrate. Ambient như cánh hoa và sao băng tiếp tục vẽ phẳng screen-space, không bị chiếu phối cảnh. Learn / Perform, nhìn trước, opacity và key tint vẫn có; Quality chọn Lite / Standard / Ultra. AR camera-facing quads gọi chung `ShadeNoteMaterial` cho các style đã biết; WARP regression mới sample sáu style trên calibrated quads và Gem/Crystal trên flat AR path, đang chờ Windows CI xác nhận. Không tuyên bố full setting/value parity; camera thật vẫn **Unverified** cho tới khi thử webcam. Camera delay (0–300 ms) bù độ trễ; `--bench-ar` đo phần việc của app. Sandbox thiếu .NET SDK nên chưa build/test tại chỗ; ambient vẫn phẳng trên camera, ghi lựa chọn ambient vào mô tả PR khi thử hardware.
- **Danh sách camera trên máy**: mỗi camera một dòng bằng tên thật (không bao giờ "Camera 1"), loại, cổng, mã VID/PID; Automatic ưu tiên camera màu, xếp camera ảo sau và camera hồng ngoại (Windows Hello) cuối. Cắm/rút camera, hay bật/tắt OBS Virtual Camera, tự cập nhật danh sách và mở lại camera khi cần; việc đó nằm yên khi đang ghi, đang đo độ trễ hoặc đang đánh dấu phím. Camera là của máy: áp preset hay dán mã chia sẻ không đổi camera đã chọn.
- Sửa hai lỗi đọc camera thật: khoá Media Foundation sai nên mọi camera YUY2/MJPG báo `0xC00D5212`, và reader tạo/đọc khác luồng gây `E_NOINTERFACE`; mỗi lần quét danh sách camera không còn để lọt một tham chiếu COM.
- Camera trực tiếp: app chọn chế độ gần 720p nhất ở tốc độ cao nhất camera có (tối đa 60 fps), đọc theo nhịp camera thay vì trần 30 fps, và hiện chế độ thật (`720p · 60 fps · MJPG`) ở dòng trạng thái. Khung hình đi qua bộ đệm 3 ô, không còn bị xé khi vẽ chậm.
- Bàn phím **88/76/61/49 phím**: chọn cỡ ở Settings → Keyboard; sân khấu, nốt rơi, phím sáng và động cơ GPU đều vẽ đúng cỡ đã chọn, nốt ngoài tầm phím không được vẽ. Preset không đổi cỡ đàn. Trang đó còn gợi ý cỡ vừa với các nốt đã chơi trên đàn thật.
- Khôi phục trang chủ hoà nhạc, ba giao diện có sẵn, nhãn đã dịch và các nút icon từ `arena/01a107ec-pianopath` khi giải quyết xung đột hợp nhất với `main`.
- Giữ artwork **Keyflow Visualizer Piano** và icon EXE Windows/bộ cài từ `main`. Mã nguồn và tài nguyên nhúng của shader Cosmic Nocturne vẫn được giữ nhưng không bật trên trang chủ đã khôi phục; bố cục và dấu hiệu thương hiệu trong app theo bản hoà nhạc.

### Âm thanh & trải nghiệm thiết lập

- Thay toàn bộ nhạc cụ SF2 đóng gói và engine SF2 bằng **Salamander Grand Piano V3 duy nhất**, phát qua SFZ; bỏ chức năng nhập SF2 và chọn nhạc cụ ngoài.
- Salamander không còn nằm trong Git, ZIP hoặc installer: người dùng chủ động tải một lần khoảng **748 MB** từ trang Audio. Downloader ghim commit, xác minh Git blob SHA-1 từng mẫu FLAC, hỗ trợ huỷ/tiếp tục và ghi mapping/attribution sau khi xác minh xong.
- Menu khởi động tách **Piano & Audio Setup** và **MIDI Devices** để vào đúng trang cài đặt nhanh hơn; trang Audio có trạng thái rõ ràng, tiến độ tải, nạp/kiểm tra lại đàn và lựa chọn đầu ra.
- Thêm công tắc **Tiếng đàn** ở Quick Adjust và Audio. Tắt tiếng chỉ ngắt đầu ra âm thanh của ứng dụng; hình ảnh, MIDI ra và âm thanh ghi vào bản thu vẫn giữ nguyên. Tuỳ chọn được lưu trên máy và preset không ghi đè.
- Cập nhật README, hướng dẫn phát hành, kiểm tra nguồn và trang phát hành công khai theo mô hình tải đàn riêng; bỏ hướng dẫn Git LFS, thư mục `Assets/` và kích thước SF2 cũ.

### Giao diện & hộp thoại

- Nút icon thay cho hàng chữ hoa toàn bộ trong dock cài đặt: hàng nút preset (lưu, xoá, nhập, xuất, đặt lại), hàng mã chia sẻ (sao chép), trang History (xuất HTML, xoá lịch sử), các thao tác tệp Nền/Camera/Hồ sơ và tạo/sửa/xoá theme giờ là nút icon kèm tooltip đã dịch. Hành động chính vẫn giữ nhãn (“Áp dụng preset”, “Áp mã”, “Lưu”).
- Nút **SETTINGS** trên header thành nút bánh răng icon; nhãn “Change” của chip LOOK thành icon gọn; năm liên kết của menu khởi động (đàn & âm thanh, MIDI, phím tắt, giới thiệu, thoát) có icon tương ứng.
- Bỏ ký tự Unicode trong nhãn nút (⬇, ⟳, ●, ↺, ×) — chúng không theo màu theme và render không nhất quán; tải đàn, làm mới thiết bị, ghi hình, xoá vòng lặp và reset từng slider dùng bộ icon riêng của app.
- Nhãn nút phụ thống nhất về chữ thường có hoa đầu; mọi nút chỉ có icon giữ tooltip — đồng thời là tên đọc cho trình đọc màn hình.
- Công tắc lớp trong Quick Adjust co theo độ dài nhãn thay vì giữ mỗi mục rộng 150 px, đưa nút gạt sát chữ hơn.

### Đóng gói & phát hành

- **Gói chưa được ký mã số**: SignPath Foundation từ chối đơn đăng ký mã nguồn mở, nên bản 2.0.0 phát hành các gói **chưa ký** (`PianoPath.exe` ×3 và `Keyflow-Setup-*.exe`). Windows SmartScreen có thể hiện bảng *"Windows protected your PC"* khi cài đặt hay chạy lần đầu: bấm *More info → Run anyway*, rồi đối chiếu tệp với `SHA256SUMS.txt`. `release.yml` không còn bước ký; hướng ký về sau nằm ở `docs/CODE-SIGNING.md`.
- Ba gói ZIP được đóng sau khi bộ cài đã biên dịch xong; tên tệp và `SHA256SUMS.txt` giữ nguyên dạng cũ.

## 1.0.0 — 2026-10-03

Bản phát hành chính thức đầu tiên. Keyflow là ứng dụng desktop Windows (C# · WPF · .NET 10) để **chơi đàn,
luyện tập và làm video piano theo MIDI** với chất lượng trình diễn hoà nhạc, giao diện **hai ngôn ngữ
English và Tiếng Việt** đổi ngay trong lúc chạy. Toàn bộ phạm vi v0.5–v0.8 của `docs/ROADMAP.md` đã vào;
mục này mô tả sản phẩm ở trạng thái phát hành thay vì liệt kê từng commit (trước 1.0.0 repo chưa cắt bản
phát hành nào, nên lịch sử chi tiết nằm trong log commit và các pull request).

### Sân khấu & đồ hoạ

- **Engine GPU Direct3D 11** (Vortice) vẽ sân khấu trên luồng riêng: HDR 16-bit, bloom nhiều tầng, mục
  tiêu khung 60/120/144/240 FPS hoặc không giới hạn, và không bao giờ chặn đầu vào MIDI. Shader HLSL nhúng
  trong assembly và được biên dịch lúc khởi động bằng `d3dcompiler_47.dll` có sẵn trong Windows.
- Đổ bóng bàn phím 88 phím theo mô hình Unreal: Cook-Torrance GGX + softbox + tonemap filmic ACES; tia lửa
  nóng sáng nguội dần theo bức xạ nhiệt, sóng cộng hưởng âm học, lửa tại điểm phím gõ, bụi acoustic, cánh
  hoa, đèn sân khấu — mỗi lớp bật/tắt riêng.
- **Bộ dựng hình WPF** (shader CPU đa luồng) là đường dự phòng khi Direct3D không khởi động được, và là
  đường vẽ của bản ghi PNG trong suốt; trang General nói rõ engine nào đang chạy và vì sao.
- **14 preset có sẵn** và **3 preset cộng đồng** nhúng trong assembly (`presets/*.json`); preset người dùng
  giữ ảnh thu nhỏ do chính sân khấu render; **mã chia sẻ diện mạo** `KEYFLOW-LOOK-1:…` (JSON nén gzip trong
  base64url) gửi một look sang máy khác, bỏ đường dẫn ảnh nền và `Clamp()` lại khi giải mã.
- Ba theme giao diện (Concert Grand, Concert Noir, Velvet Gold) cộng **xưởng theme** của người dùng: năm
  màu gốc suy ra hai mươi token, lưu trong `<cài đặt>/themes/*.json`; high contrast của Windows được nhận
  diện và vẽ lại ngay khi hệ thống đổi.

### Âm thanh

- Synthesizer **SoundFont 2** viết tay, không thêm dependency: preset/zone/sample, ngữ nghĩa generator
  (instrument ghi đè, preset cộng dồn), loop, giai đoạn release, giới hạn đa âm, sustain/sostenuto/soft.
- SoundFont đi kèm: **YDP Grand Piano** của FreePats (đa mẫu Yamaha Disklavier Pro), giấy phép **CC BY 3.0**,
  ~113 MiB qua Git LFS; `Assets/ATTRIBUTION.txt` ghi công và nằm trong mọi gói phân phối.
- Đầu ra `waveOut`; máy không mở được thiết bị âm thanh vẫn chạy đầy đủ và báo đúng lý do.

### Nhập bài & khuông nhạc

- **MIDI** định dạng 0/1/2, cả hai kiểu độ chia (PPQ lẫn SMPTE), tempo map, kênh percussion, tên track,
  lưới phách theo ô nhịp (`Meter.Of`: 4/4, 3/4, 2/2, 3/8 đập theo đơn vị đã ghi; 6/8, 9/8, 12/8 đập theo
  nhóm ba; 7/8 vẫn bảy), từ chối tệp hỏng.
- **MusicXML** `.musicxml`/`.xml`/`.mxl` (nén đọc qua `META-INF/container.xml`), giữ con trỏ theo
  `divisions` trong từng part/measure, `backup`/`forward`, đổi tempo giữa bài, hợp âm dùng chung điểm vào,
  điểm chia tay đọc từ `<staff>`.
- **Lớp khuông nhạc** vẽ trên sân khấu: tự suy ra hoá biểu của bài (tương quan Krumhansl–Kessler), viết dấu
  hoá theo từng ô nhịp, nối đuôi nốt ngắn theo phách, viết dấu nghỉ cho tay đang im lặng, nối dấu luyến cho
  nốt vắt qua vạch nhịp, ghép hợp âm thành một cột chung đuôi, nối đường cong khi dòng nhạc đổi tay, và
  dòng kẻ phụ chỉ khi nốt ra ngoài khuông.
- **Suy luận điểm chia hai tay** từ chính bài: gom hai cụm theo thời lượng vang, chỉ nhận khi giữa hai tay
  có ≥5 semitone trống và mỗi tay chiếm ≥10%, ưu tiên nốt giữa C4; bài một tay hoặc chồng lấn thì giữ lựa
  chọn của người dùng, và kết quả được nhớ theo từng bài.

### Luyện tập

- Chấm điểm theo nốt (đúng/sai, độ chính xác, chuỗi dài nhất) và **giữ từng nốt đã chấm điểm** của lượt
  chơi (tối đa 256 nốt) để vẽ **bóng (ghost)**: lượt tốt nhất ở trên, lượt mới nhất ở dưới.
- **Biểu đồ 14 ngày** theo ngày địa phương (ngày không luyện vẫn là một hàng 0), trong dock và trong báo
  cáo HTML.
- **Tempo luyện tập tự động**: sai liên tiếp quá ngưỡng (1–6, mặc định 3) thì mỗi bước giảm 5% tới sàn
  50%, đúng 4 nốt liên tiếp thì tăng 2% và dừng ở 100% — mỗi bước đi qua chính slider tempo.
- **Lịch sử luyện tập** append mỗi lượt một dòng JSON vào `history/practice.jsonl` (giữ 200 lượt gần nhất,
  bỏ qua dòng hỏng), trang **History** in các lượt gần nhất và lượt tốt nhất của bài đang mở, kèm
  **EXPORT HTML** và **CLEAR HISTORY**.

### Ghi hình

- **MP4 (H.264 + AAC)** ghi trực tiếp bằng Media Foundation sink writer sẵn có trong Windows, tiếng nằm
  trong tệp: khung BGRA của sân khấu đổi sang NV12, bitrate theo kích thước × fps, PCM của engine âm thanh
  mã hoá AAC qua cùng giao kèo `IAudioTrack`.
- **AVI** (MJPEG, hoặc RGB không nén khi máy thiếu codec; trần 2 GB) với tiếng ở **WAV** stereo 16-bit cạnh
  đó, kèm dòng lệnh ffmpeg để ghép.
- **Chuỗi PNG 32-bit có alpha** cho hậu kỳ nền trong, kèm `sequence.json` và lệnh ffmpeg `libvpx-vp9` +
  `yuva420p` dựng thành WebM alpha.
- Việc mã hoá chạy trong **tiến trình con** (`--encode-take`) với phép thử codec trước khi ghi và phép thử
  đường ống khi lượt ghi không ra tệp, nên codec làm sập tiến trình thì kết luận vẫn còn; cửa sổ mở bộ ghi
  được bọc `HangGuard` hạn 10 giây rồi **tự lùi về AVI** kèm câu báo trên máy có media stack treo.
- **Lớp phủ camera** (camera trực tiếp hoặc tệp video chạy lặp, đặt theo bốn góc, mirror, opacity, key màu
  xanh lá) và **theo dõi bàn tay** bằng thị giác cổ điển: luật màu da YCbCr có độ nhạy chỉnh được, lưới
  32×24, đếm ngón theo biên dạng cột, vẽ dải sáng trên phím bàn tay đang ở.
- **Hết đơ và hết tệp hỏng khi ghi** — ba bản vá sau lượt cắt đầu tiên: mỗi khung và mỗi khối âm thanh trả
  lại bộ đệm COM của Media Foundation ngay trong `finally` (trước đây chúng bị giữ tới lúc GC, RAM phình lên
  rồi cả cửa sổ đứng khi chúng được giải phóng cùng lúc); khung hình và âm thanh đi qua một hàng đợi có
  biên tới luồng công nhân làm phần chuyển NV12 và ghi tệp, khung mà bộ mã hoá theo không kịp được gộp vào
  khung mới nhất thành khung lặp nên tệp vẫn giữ đúng thời gian thực; cả lượt ghi MP4 nằm trên **một luồng
  duy nhất** (Media Foundation từ chối sink writer tạo ở luồng này rồi gọi ở luồng khác — đó là lý do những
  lượt ghi trước ra tệp rỗng hoặc thiếu index) và lỗi lúc đóng tệp được báo ra thay vì bỏ qua; AVI cũng ghi
  trên luồng riêng, khung lặp ghi bằng null frame thay vì nén lại, và lượt ghi bị chặn ở **1080p** theo cả
  hai chiều.
- **MP4 nay là định dạng mặc định**, và tệp nó ghi ra mở được ở mọi trình phát: `MF_MT_FRAME_SIZE` xếp
  chiều rộng vào word cao nhưng bộ ghi lại truyền chiều cao trước, nên bộ mã hoá đọc mọi khung với stride
  sai và một lượt 1920×1080 bị khai thành 1080×1920 — hình ra thành nhiễu ngang. Kích thước nay được khai
  đúng `rộng × cao`, `--soak-record` so kích thước tệp tự khai với kích thước đã yêu cầu, và hồ sơ mới mặc
  định ghi MP4 thay vì AVI (AVI không có codec MJPEG là khung thô, đầy trần 2 GB trong vài giây ở 1080p).

### Thư viện, hồ sơ & bàn thiết kế

- Hộp thoại Play có **RECENT** (12 bài gần nhất kèm số nốt, số track, tempo, điểm chia tay, tốc độ rơi,
  tempo phát và preset — mở lại là khôi phục qua đúng các slider) và **LIBRARY** theo thư mục (quét BFS
  tối đa 3 tầng/500 tệp bằng chính reader của app, cache theo kích thước + giờ sửa trong
  `library-index.json`, thẻ theo từng bài, tìm kiếm khớp tiêu đề–tên tệp–thẻ, và theo dõi thư mục nên danh
  sách tự cập nhật).
- **Hồ sơ cài đặt** `Keyflow.profile.json` gom cài đặt sân khấu + ngôn ngữ + theme, nhập/xuất ở trang
  General; kéo-thả `.json` (hồ sơ), `.mid`/`.midi` (mở bài) và ảnh (đặt nền) vào cửa sổ.
- **Undo/redo 32 bước** cho bàn thiết kế (`Ctrl+Z` / `Ctrl+Shift+Z` / `Ctrl+Y`), một lần kéo slider là một
  bước vì ảnh chụp được commit ở nhịp idle của bộ tự lưu.
- **Tìm kiếm trong dock** có từ đồng nghĩa theo từng setting, khớp theo token (giao), tên setting là lưới
  an toàn cuối, và phần khớp được tô màu accent.

### Giao diện, đa ngôn ngữ & trợ năng

- Dock **13 trang chia 4 nhóm** (STAGE DESIGN, SOUND & INPUT, SESSION, APP); catalogue trang nằm ở
  `Ui/SettingsPages.cs` và được đối chiếu với dải tab trong XAML cả lúc tĩnh lẫn lúc chạy.
- **Song ngữ**: 1.266 khoá chuỗi, bảng tiếng Việt đầy đủ, đổi ngôn ngữ trên cửa sổ đang mở và mọi nhãn vẽ
  lại ngay; id ngôn ngữ/theme/preset luôn ở dạng tiếng Anh trong tệp lưu nên đổi ngôn ngữ không đổi dữ
  liệu trên đĩa.
- **Trợ năng**: `AutomationProperties.Name` cho mọi nút chỉ có glyph và mọi hàng sinh tự động, Tab đi vòng
  trong dock theo đúng thứ tự các hàng, palette `SystemColors` khi Windows bật high contrast, và ở cửa sổ
  1080×700 mọi trang vẫn giữ điều khiển trong khung dù một từ ở ngôn ngữ khác dài hơn.
- Thẻ phím tắt (F1), Escape ưu tiên lớp đang thấy, menu khởi động có chọn ngôn ngữ và xem nhanh diện mạo
  sân khấu.

### Đóng gói & phân phối

- **`publish.ps1`**: một lệnh ra bản single-file **self-contained** (ReadyToRun) hoặc
  **framework-dependent**, cho `win-x64` hoặc `win-arm64`, kèm `-Zip`; script từ chối publish khi
  `Assets\ConcertGrand.sf2` còn là con trỏ Git LFS (bản build sẽ chạy mà không có tiếng đàn).
- **Bộ cài Inno Setup** song ngữ (`installer\Keyflow.iss` + bản dịch một phần 108 câu trong
  `installer\Languages\Vietnamese.isl`): tự chọn ngôn ngữ theo Windows, tạo shortcut Start Menu/Desktop và
  mục gỡ cài đặt; `tools/build_installer.ps1` biến mọi cảnh báo lạ của ISCC thành lỗi.
- **`release.yml`**: đẩy tag `v*` là có GitHub Release kèm **ba ZIP** (self-contained `win-x64`,
  framework-dependent `win-x64`, self-contained `win-arm64`), **bộ cài `.exe` `win-x64`** dựng từ chính thư
  mục vừa publish, và **`SHA256SUMS.txt`** của từng tệp để người tải tự kiểm gói (các gói chưa ký số).
  Bản publish được smoke test bằng `--verify` trước khi đính kèm.
- **`LICENSE.txt` (MIT) và `Assets/ATTRIBUTION.txt` (CC BY 3.0 của SoundFont) nằm trong mỗi gói** — ZIP lẫn
  thư mục bộ cài — chứ không chỉ trong repo, đúng yêu cầu "đi kèm bản sao" của cả hai giấy phép.
- `build.yml` chạy **một lượt publish thật** (framework-dependent, `-AllowLfsPointer` vì CI không tải LFS)
  mỗi lần push và kiểm bản publish có đủ `PianoPath.exe`, `LICENSE.txt`, `Assets\`: lỗi của đường đóng gói
  nay đỏ ngay lúc push thay vì đợi tới lần cắt bản phát hành.

### Kiểm chứng

- **Ba lớp kiểm chứng**:
  - `tools/check_sources.py` — tĩnh, chạy trên mọi máy, không cần .NET SDK. Đợt này thêm **hai luật**:
    `scan_release_version` ghim số phiên bản vào một nguồn duy nhất (`<Version>` của `PianoPath.csproj`)
    và `scan_project_files` parse mọi file MSBuild (`.csproj`, `.pubxml`) đúng cách MSBuild sẽ parse, vì
    một comment XML chứa hai dấu gạch nối là cả project không nạp được.
  - `--verify` — **58 hàm kiểm chứng** trên Windows: toán shader, MIDI, SoundFont, AVI/MP4/PNG, cài đặt,
    dock, ngôn ngữ, lịch sử, hồ sơ, trợ năng, high contrast, engine GPU trên WARP và cổng hiệu năng.
  - **Ảnh README do chính ứng dụng render trong CI** (mỗi cảnh một ảnh cho mỗi ngôn ngữ, cộng gallery
    preset), không phải ảnh dàn dựng.
- **`tests/PianoPath.Tests`**: 133 test case xUnit trên `net10.0` thường (link 10 tệp nguồn, chạy trên
  Linux trong ~1 phút) cho MIDI, MusicXML, chia tay, lưới phách, WAV, ngân sách khung hình và nửa thuần
  của `Loc`.
- **Cổng hiệu năng `--bench`**: đo thời gian từng khung ở 1920×1080 cho hai cảnh (cài đặt mặc định
  p95 < 8 ms, preset có sẵn nặng nhất p95 < 16 ms) trên máy có card đồ hoạ thật; trên adapter phần mềm
  (mọi runner CI) chỉ so tương đối với lượt trước và in `NOTE`, vì cùng một code mà WARP đo chênh nhau gần
  hai lần giữa hai lượt chạy.

### Đã biết, để sau 1.0.0

- **Chưa ký mã**: Windows SmartScreen vẫn hiện "Windows protected your PC" ở lần chạy đầu (README ghi cách
  vượt qua). `SHA256SUMS.txt` là để người tải kiểm tệp, không thay được chữ ký.
- **Chưa có MSIX, manifest winget và kiểm tra cập nhật trong ứng dụng** — cả ba cần tài khoản/chứng chỉ
  hoặc một quyết định về mạng, trong khi sản phẩm cam kết offline và không telemetry
  (`docs/ROADMAP.md` §4 #5).
- **Một tệp MP4 thật vẫn chờ một máy Windows bình thường xác nhận**: runner CI treo trong
  `IMFSample::SetSampleTime` nên mục đó luôn `SKIP` ở đó; app đã tự lùi về AVI khi gặp một máy như vậy.
- **Ngân sách 8/16 ms chưa được xác nhận bằng số đo trên card đồ hoạ thật** — sandbox và runner đều chỉ có
  WARP. Gói `win-arm64` cũng vậy: được publish trong CI nhưng chưa chạy thử trên máy ARM.
- Chưa có bản *lite* không kèm SoundFont (~113 MiB là khoảng một phần ba tổng tải) — `docs/ROADMAP.md` §4 #4.

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

Giới hạn kỹ thuật của sản phẩm nằm ở mục *[Giới hạn hiện tại](README.md#giới-hạn-hiện-tại)* của README và
không được lặp lại ở đây; việc còn mở nằm ở [`docs/ROADMAP.md`](docs/ROADMAP.md).

## 1.0.0 — 2026-10-02

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

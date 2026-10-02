# Đa ngôn ngữ: kiến trúc, quy ước và cách thêm một ngôn ngữ mới

Tài liệu này mô tả cách Keyflow dịch giao diện: **khoá là câu chữ tiếng Anh trong mã nguồn**, mỗi
ngôn ngữ là **một tệp bảng**, và mọi nhãn trên màn hình **tự vẽ lại khi đổi ngôn ngữ** mà không khởi
động lại ứng dụng. Phần danh mục trang và nhóm điều hướng đi kèm nằm ở
`docs/DOCK-NAVIGATION-AUDIT.md`; các hướng đi tiếp theo nằm ở `docs/ROADMAP.md`.

> Môi trường: sandbox phát triển **không có .NET SDK** và WPF chỉ biên dịch trên Windows. Vì vậy tính
> đúng đắn của bảng chuỗi được chứng minh bằng `tools/check_sources.py` (tĩnh, mọi máy) và
> `VerificationSuite` (`--verify`, runner Windows) chứ không bằng mắt người.

## 1. Bốn nguyên tắc

| # | Nguyên tắc | Vì sao |
|---|---|---|
| 1 | **Khoá là văn bản nguồn tiếng Anh** (`"Falling notes"`, `"Preset “{0}” applied"`), không phải id do con người đặt | Mã nguồn đọc như giao diện thật; một bản dịch thiếu tự rơi về tiếng Anh thay vì nhãn trống; dịch giả `grep` đúng chuỗi trong `Ui/`, `Stage/`, `Audio/` là thấy ngay ngữ cảnh |
| 2 | **Id không bao giờ dịch.** Giá trị cài đặt, tên preset, id theme, tên thiết bị Windows và đường dẫn vẫn là tiếng Anh; chỉ *caption* hiển thị là dịch | `visual-settings.json` và `presets\*.json` ghi ra từ một máy tiếng Việt phải mở được trên máy tiếng Anh; đổi ngôn ngữ không làm hỏng thứ đã lưu |
| 3 | **Nhãn đăng ký với `Loc` thì tự vẽ lại.** Không có "đổi ngôn ngữ rồi khởi động lại" | Ứng dụng là công cụ biểu diễn: người dùng đang mở bàn thiết kế, đang quay video, không được mất phiên làm việc vì đổi thứ tiếng |
| 4 | **Bảng chuỗi là dữ liệu, không phải logic.** Thêm ngôn ngữ = thêm một tệp bảng + một dòng đăng ký, không sửa renderer | Nhánh `if (language == ...)` trong code vẽ là con đường dẫn tới giao diện lệch nhau giữa các ngôn ngữ |

## 2. File và vai trò

| Tệp | Vai trò |
|---|---|
| `Localization/Localizer.cs` | `AppLanguage`, `Languages` (danh sách ngôn ngữ + cách giải một giá trị đã lưu), và `Loc`: tra bảng, đăng ký nhãn sống, marker XAML, chuẩn đoán cho `--verify` |
| `Localization/Strings.English.cs` | **Inventory**: mọi khoá ứng dụng có thể in ra, ánh xạ tới chính nó. Là danh sách việc của dịch giả |
| `Localization/Strings.Vietnamese.cs` | Bảng tiếng Việt, đúng cùng tập khoá, theo cùng thứ tự |
| `Ui/DeviceOption.cs` | Bản ghi `(Id, Display)`: hai hàng "Computer keyboard only" / "No MIDI output" dịch được, thiết bị thật do Windows đặt tên thì giữ nguyên |
| `Ui/MainWindow.Language.cs` | Lối vào phía giao diện: `ApplyLanguage`, `RetranslateSurfaces`, tiêu đề nhóm điều hướng, caption combo thiết bị |
| `Ui/MainWindow.Settings.cs` | Trang **General** (nhóm APP) với các chip ngôn ngữ; ô tìm kiếm của dock khớp cả tiếng Anh lẫn tiếng Việt |
| `Ui/MainWindow.xaml` | `local:Loc.Localize="True"` trên mọi phần tử mang chữ tĩnh; thẻ **INTERFACE LANGUAGE** ở menu khởi động |
| `tools/check_sources.py` | Kiểm tĩnh: hai bảng phải cùng tập khoá, placeholder còn nguyên, **mọi chuỗi mã nguồn in ra phải là một khoá của inventory**, và **mọi khoá của inventory phải còn được một tệp mã nguồn giữ** (`scan_dead_keys`: một khoá không nguồn nào in ra là câu dịch vô ích — nó chỉ còn sống trong hai bảng, và lần đổi chữ sau sẽ để lại bên cạnh nó một bản sao thứ hai của cùng câu đó) |

## 3. API trong mã C#

```csharp
Loc.T("Falling notes");                                   // tra một chuỗi tĩnh
Loc.F("Preset “{0}” applied", name);                       // bản mẫu + tham số (CurrentCulture)
Loc.Set(caption, "Falling notes");                         // nhãn sống: tự đổi khi đổi ngôn ngữ
Loc.Set(tooltip, "Reset fall speed", FrameworkElement.ToolTipProperty);
Loc.Format(status, "Fall speed {0}", value);               // bản mẫu sống, có tham số
Loc.Bind(label, () => Loc.T(name) + suffix);               // khi văn bản phải tính ra
Loc.Page(SettingsPages.Theme);                             // tên trang/nhóm qua cùng một bảng
Loc.OnChanged(RefreshChoiceCaptions);                      // mặt dựng lại từ đầu (không phải label)
Loc.Apply("vi");  Loc.Refresh();                           // đổi ngôn ngữ rồi vẽ lại
```

Ba điều đáng nhớ khi dùng:

* **`Set`/`Format`/`Bind` đăng ký, chúng không in ngay.** Nhãn được giữ bằng `ConditionalWeakTable`
  + danh sách `WeakReference`, nên một element bị gỡ đi không bao giờ được vẽ lại và cũng không giữ
  đối tượng sống. Vì vậy code tạo control *luôn* gọi `Loc.Set`, kể cả khi ngôn ngữ hiện tại là tiếng Anh.
* **Chỗ nào văn bản được *ghép* thì phải dùng renderer** (`Bind`/`Format`), không dùng `T` lên chuỗi
  đã ghép. `Loc.T(preset.Description)` hợp lệ vì `Description` là khoá tiếng Anh;
  `Loc.T(description + path)` thì không, và bộ kiểm tĩnh sẽ báo khoá lạ.
* **Đừng so sánh văn bản đã dịch.** Muốn biết trạng thái, so sánh giá trị đã lưu
  (`_visualSettings.BackgroundMode == "Image"`), đừng so sánh `label.Text == "Image"`.
  `VerificationSuite` đã được viết lại theo quy tắc này: nó so với `Loc.T("NO SOUNDFONT · SILENT")`
  và `Loc.T("Style")` chứ không so với chuỗi tiếng Anh cứng.

### Marker trong XAML

```xml
<TextBlock Text="LOOK" local:Loc.Localize="True" Style="{StaticResource EyebrowTextStyle}"/>
```

`Loc.Localize` là attached property: khi gán `True`, `Loc.Track` tìm literal theo thứ tự
`TextBlock.Text` → `HeaderedContentControl.Header` → `ContentControl.Content` →
`FrameworkElement.ToolTip` → `Window.Title` và đăng ký từng tính chất tìm thấy. Vì vậy
`<Button Content="A" ToolTip="Set the loop start at the playhead" local:Loc.Localize="True"/>`
dịch *tooltip* và để nguyên chữ `A`; một ký tự đơn (glyph, dấu, `↺`) được bỏ qua có chủ ý.
Literal tiếng Anh vẫn nằm trong XAML nên markup đọc tự nhiên và thiếu bản dịch chỉ in tiếng Anh.

## 4. Thêm một ngôn ngữ (ví dụ: tiếng Nhật)

1. **Tạo bảng** — sao chép `Strings.English.cs` thành `Localization/Strings.Japanese.cs`, đổi tên lớp
   thành `StringsJapanese`. Inventory đã là danh sách khoá; giữ nguyên thứ tự từ điển ordinal.
2. **Đăng ký** — trong `Localization/Localizer.cs`:

   ```csharp
   internal static readonly AppLanguage Japanese = new("ja", "Japanese", "日本語", StringsJapanese.Table);
   internal static readonly AppLanguage[] All = [English, Vietnamese, Japanese];
   ```

   Xong phần code: bộ chọn ngôn ngữ, `--lang=ja`, `SettingsPages` và mọi trang đều lấy danh sách từ
   `Languages.All`.
3. **Dịch theo checklist** — `Localization/Strings.English.cs` chính là danh sách việc. Chạy
   `python tools/check_sources.py` sau mỗi lượt dịch: nó liệt kê đúng những khoá bảng mới còn thiếu và
   những bản mẫu làm mất `{0}`/`{1}`. Giá trị để trống bị coi là chưa dịch; `--verify` báo danh sách đó
   dưới dạng `UntranslatedKeys`.
4. **Kiểm tra** — ba lớp, theo thứ tự rẻ nhất:

   ```powershell
   python tools/check_sources.py                                   # parity + phủ khoá (mọi máy)
   dotnet run --project .\PianoPath.csproj -c Release -- --lang=ja --show-settings --settings-tab=general
   dotnet run --project .\PianoPath.csproj -c Release -- --verify   # UntranslatedKeys phải rỗng
   ```

5. **Ảnh preview (tuỳ chọn)** — `build.yml` render ảnh README bằng `--lang=en`; thêm một mục
   `$shots` với `--lang=ja` nếu muốn có ảnh cho ngôn ngữ mới.

Không có bước nào sửa renderer, `App.xaml`, `PianoVisualSettings` hay `visual-settings.json`.

## 5. Quy ước dịch tiếng Việt

| Loại | Cách xử lý |
|---|---|
| Thuật ngữ kỹ thuật | Giữ nguyên: MIDI, SMF, SoundFont (`.sf2`), AVI, OBS, FPS, GPU, RGB, HEX, pedal, preset, theme, slider, timeline, note, hit/miss/streak |
| Tên riêng của sản phẩm | Không dịch: Keyflow, Concert Grand, Concert Noir, Velvet Gold, Neon Violet, Concert Gold, Sakura Nocturne…; `Moonlight Sonata` → `Ánh Trăng`, `Galaxy Voyage` → `Du Hành Thiên Hà` cho nhãn mô tả, còn *id/tên đã lưu* vẫn là tiếng Anh |
| Tiêu đề in hoa | Viết hoa tiếng Việt có dấu: `KEYBOARD & SHORTCUTS` → `BÀN PHÍM & PHÍM TẮT`, `SESSION & CAPTURE` → `BUỔI TẬP & GHI HÌNH` |
| Xưng hô | Trung tính, gần: "bạn" cho người dùng, không "quý khách"; câu mệnh lệnh ngắn ("Chọn MIDI File", "Giữ phím để note ngân dài") |
| Bản mẫu | Giữ đúng số lượng và số thứ tự `{0}`, `{1}`; giữ nguyên `\n`; giữ ngoặc kép cong “ ” và dấu `·` làm máy phân cách |
| Số và đơn vị | Định dạng theo `CultureInfo.CurrentCulture` (`0,5` chứ không `0.5`); đơn vị `px`, `px/s`, `fps`, `%` giữ nguyên |
| Chuỗi lọc hộp thoại tệp | Giữ pattern phần mở rộng: `AVI video (*.avi)\|*.avi` |
| Nhãn trên sân khấu | Watermark `KEYFLOW` và tên nốt (`C4`) là **một phần của hình ảnh**, không dịch; `KEYS` / `PARTICLES` trong HUD thì có |

## 6. Cố ý không dịch

| Thứ | Lý do |
|---|---|
| Tên thiết bị MIDI/âm thanh | Windows trả về; hai hàng placeholder đã dịch là đủ |
| Tên tệp preset, đường dẫn ảnh nền | Đó là dữ liệu người dùng, không phải văn bản giao diện |
| `A` / `B` / `↺` / `F1` / `Esc` / `Space` | Phím và glyph in nguyên văn; chip trong thẻ F1 cũng vậy vì nó mô tả phím thật trên bàn phím |
| Giá trị liệt kê trong JSON (`"ChromaGreen"`, `"PerHand"`, `concert-noir`) | Id bền vững, đã có bảng `LegacyIds`/migration riêng |
| `KEYFLOW` watermark, tên nốt `C4` | Là nội dung khung hình người dùng mang đi làm video |
| Nhật ký `--verify` và thông báo exception nội bộ | Dành cho người phát triển, không phải cho màn hình |

## 7. Vì sao không dùng `.resx` / `ResourceManager`

`.resx` + satellite assembly là lựa chọn đúng cho bộ máy dịch lớn, nhưng ở đây nó đổi ba thứ lấy một
lợi ích nhỏ: phải có id cho mọi chuỗi (mất tính "nguồn là khoá"), phải truyền `ResourceManager` vào
mọi nơi sinh text, và bản dịch lọt sang các assembly `net10.0-windows` trong khi bản publish dùng
`-p:SatelliteResourceLanguages=en` để bỏ tài nguyên ngôn ngữ của WPF. Một tệp `Dictionary` static thì
không có vòng đời riêng, không cần tải chậm, biên dịch cùng mã nguồn nên **không có khoá nào biến mất
im lặng** — `check_sources.py` thấy mọi literal tại build time.

Ngược lại, hai việc `.resx` từng giúp thì dự án này đã tự làm: parity bắt buộc (mục 8) và một file
danh sách việc cho dịch giả (chính `Strings.English.cs`).

## 8. Ba lớp kiểm tra

| Lớp | Chạy ở đâu | Bắt được gì |
|---|---|---|
| `python tools/check_sources.py` | Mọi máy, **trước** bước build trong CI | Hai bảng có đúng cùng tập khoá; English map khoá tới chính nó; thứ tự ordinal; placeholder và `\n` còn nguyên; **mọi literal mà mã nguồn in ra phải là khoá** (đi theo cả đường `Loc.*`, các row builder, bảng phím tắt, theme, preset, trang điều hướng và marker XAML) |
| `--verify` → `VerifyLanguageSwitching` | runner Windows | Bản dịch không thiếu key nào; `Languages.Find/Normalize` chấp nhận id, culture tag, tên bản ngữ và rơi về tiếng Anh; **đổi ngôn ngữ trên cửa sổ đang mở** làm đổi nhãn thật; ô tìm kiếm dock tìm hàng bằng tiếng Việt; không để lại key lạ |
| Ảnh README do CI render | runner Windows | Ngôn ngữ và bố cục hiện ra trong ảnh, nên một nhãn trống hay chuỗi chưa dịch lộ ra ngay ở trang chủ |

## 9. Còn mở

* **Chuỗi lạ đã là lỗi, không còn là `NOTE`.** `VerifyLanguageSwitching` khẳng định
  `Loc.UnknownKeys` rỗng: một câu mà giao diện in ra nhưng không phải khoá của inventory thì không
  bảng nào dịch được, nên nó là lỗi hồi quy chứ không phải thông tin môi trường. Muốn thêm câu mới
  thì thêm khoá vào `Strings.English.cs` + `Strings.Vietnamese.cs` (đúng thứ tự ordinal) — bộ kiểm
  tĩnh và `--verify` sẽ cùng báo nếu quên. Tên do người dùng đặt (thiết bị, tệp, preset tự lưu) đi
  thẳng ra màn hình nguyên văn chứ không đi qua `Loc`, nên không bị tính là chuỗi lạ.
* Chip phím tắt trong thẻ F1: nhãn mô tả (`Click the keys`, `MIDI keyboard`, `Pointer idle`,
  `Drag the timeline`, `Practice modes`) **đã** đi qua `Loc.Set` và có khoá trong inventory; chỉ cột
  phím (`A W S …`, `Space`, `A · B · ×`) là in nguyên văn theo đúng quy ước "phím thật không dịch".
* `VisualPresets.SanitizeName` trả `"My preset"` làm **tên tệp** khi người dùng lưu mà không đặt tên;
  nhãn hiển thị của nó thì đã dịch (`Preset của tôi`) vì `DisplayName` đi qua bảng. Hệ quả phụ: một preset
  do người dùng tự đặt tên trùng hẳn một khoá của bảng (ví dụ `Custom`) sẽ bị dịch khi in — vô hại nhưng
  nên giới hạn `DisplayName` cho đúng các preset có sẵn (hiện đã làm vậy).
* Mô tả preset *do người dùng lưu* là một chuỗi ghép (`User preset · x.json`); nó đã theo ngôn ngữ
  lúc dựng danh sách, nhưng một preset tự đặt tên tiếng Việt thì không nên bị `T()` chạm vào.
* Tài liệu có hai bản: `README.md` (tiếng Việt, bản gốc) và `README.en.md` (tiếng Anh). `scan_readme`
  trong `tools/check_sources.py` quét **cả hai** (ảnh, anchor, liên kết nội bộ) và bắt buộc mỗi bản
  phải trỏ sang bản kia; `scan_cli_and_samples` kiểm bảng tham số dòng lệnh của từng bản, nên bản
  tiếng Anh không thể thiếu một switch mà bản tiếng Việt đã có.
* **Bộ cài (`installer/`)** là mặt thứ ba có chữ hiển thị, và nó không đi qua `Loc`: Inno Setup đọc
  `installer/Languages/Vietnamese.isl` (UTF-8 **có BOM** để trình biên dịch biết ngay là Unicode),
  liệt kê **sau** `compiler:Default.isl` trong `MessagesFile` nên tệp chỉ *ghi đè* 108 câu mà trình
  cài đặt thật sự hiện — phần còn lại theo tiếng Anh của `Default.isl`. Ba thứ dễ sai đều bị chặn:
  tên câu không tồn tại (Inno Setup chỉ **cảnh báo** rồi bỏ dòng, tức là âm thầm hiện tiếng Anh),
  câu đặt nhầm `[Messages]`/`[CustomMessages]`, và placeholder bị rơi mất. `tools/inno_messages.py`
  sinh `installer/Languages/messages.txt` (tên hợp lệ + placeholder, lấy từ `Default.isl` của ba mốc
  Inno 6.3.3/6.7.3/7.1.0), `scan_installer` trong `tools/check_sources.py` đối chiếu bản dịch với tệp
  đó, và `tools/build_installer.ps1` (chạy trong cả hai workflow) biến mọi cảnh báo khác của ISCC
  thành lỗi build. Quy ước dịch vẫn theo §5 (`Huỷ`, `tệp`, `thư mục`, `shortcut`, `Start Menu`).
* RTL (Ả Rập, Do Thái, Ba Tư) cần thêm `FlowDirection`, đảo `Margin`/`Grid` cột và đường rơi của nốt;
  bảng cho một ngôn ngữ RTL phải đi kèm đợt việc đó, không nên thêm bảng suông.
* `installer/Keyflow.iss` vẫn dùng một tệp ngôn ngữ Inno; bản dịch installer tiếng Việt là việc riêng
  (xem `docs/ROADMAP.md`).

## 10. Ảnh giao diện: chỉ ảnh do ứng dụng render

`docs/previews/` **chỉ chứa ảnh do chính `PianoPath.exe` render trong CI** (`--snapshot`), chia làm hai bộ:
`docs/previews/vi/` cho `README.md` và `docs/previews/en/` cho `README.en.md`, mỗi bộ chụp cùng danh sách
chủ đề (`$shots` trong `build.yml`) bằng đúng ngôn ngữ của bản README đọc nó (cộng `docs/previews/presets.jpg`,
gallery mọi preset do một bước riêng của cùng workflow dựng, dùng chung cho cả hai bản). Screenshot của người dùng — kể cả ảnh rất đẹp cóảnh nền tự chọn — không commit vào đây, vì hai lý do: ảnh sẽ lệch khỏi UI thật ngay lần sửa giao diện
kế tiếp, và ảnh nền trong screenshot hầu như luôn là artwork của bên thứ ba, không kèm giấy phép cho repo MIT.

Muốn tài liệu chiếu được một tính năng thì làm cho CI render được tính năng đó. Ảnh nền là ví dụ đã làm
xong: `tools/make_stage_background.py` **tự sinh** `docs/samples/stage-backdrop.png` (thuần stdlib Python,
chạy dưới một giây, ảnh tất định vì sao trời là một danh mục có seed), `--background-image=<file.png>`
treo ảnh đó lên sân khấu **chỉ trong lần chạy này**, và `build.yml` chụp `background-image.png`. Cờ này
gọi `MainWindow.PreviewBackgroundImage`, vốn đặt thẳng vào stage thay vì đi qua handler của các hàng cài
đặt: không bật cờ "đã sửa", không mồi đồng hồ tự lưu, và ảnh đọc không được thì chỉ để stage tự báo lỗi
trong `BackgroundLoadError` chứ không mở hộp thoại — một lượt chụp không có ai để bấm OK. `VerificationSuite`
kiểm đúng hợp đồng đó, nên hành vi "chỉ xem, không ghi" không thể lệch trong im lặng.

`tools/check_sources.py` giữ ba đầu mối thẳng hàng:

* mọi `![...]()` trỏ vào `docs/previews/` phải hoặc đã tồn tại, hoặc có tên trong `$shots` của workflow —
  được phép chậm hơn README đúng một commit, vì chính commit render ảnh sẽ bắt kịp — **và** bản `README.md`
  chỉ được trỏ vào `docs/previews/vi/`, bản `README.en.md` chỉ được trỏ vào `docs/previews/en/`, còn
  workflow phải thật sự render cả hai ngôn ngữ (vòng `foreach ($lang in @('en', 'vi'))`);
* mọi tham số dòng lệnh mà `App.xaml.cs` hoặc `VerificationSuite` đọc phải có trong bảng *Tham số dòng
  lệnh* của README và ngược lại, còn mọi tham số + đường dẫn `build.yml` truyền cho `PianoPath.exe` phải
  thật sự tồn tại — một cờ gõ sai không làm hỏng build, nó chỉ im lặng cho ra một ảnh xấu;
* `docs/samples/stage-backdrop.png` phải khớp kích thước với `W, H` của script sinh ra nó: nộp ảnh sửa
  tay vào `docs/samples` sẽ bị báo lỗi thay vì âm thầm chia rẽ khỏi script.

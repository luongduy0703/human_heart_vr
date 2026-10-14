# Bài thực hành: Trái tim sống – AR trên web bằng webcam laptop

*Tài liệu dành cho trợ giảng. Sinh viên dùng cùng tài liệu này để làm theo, còn các phần "Gợi ý cho trợ giảng" và "Đáp án" chỉ dành cho người hướng dẫn.*

---

## Thông tin buổi học

| Mục | Nội dung |
|---|---|
| Chủ đề | Thực tế tăng cường (AR) dựa trên ảnh, chạy trên trình duyệt |
| Thời lượng | 120 phút (có thể chia 2 buổi × 60 phút: buổi 1 gồm phần 1–3, buổi 2 gồm phần 4–5) |
| Đối tượng | Sinh viên đã biết HTML, CSS, JavaScript cơ bản (biến, hàm, `async/await`). Không cần biết trước three.js hay AR |
| Sản phẩm | Trang web: webcam nhìn vào tờ giấy in marker thì trái tim 3D trồi lên từ mặt giấy; kéo chuột để xoay, lăn chuột để zoom, bấm chấm màu để xem tên bộ phận |
| Mã nguồn | Thư mục `web/` của repo: `index.html`, `main.js`, `heart-data.js`. Cách chạy chi tiết xem [README.md](README.md) |
| Thư viện | [MindAR](https://hiukim.github.io/mind-ar-js-doc/) 1.2.5 (nhận diện ảnh), [three.js](https://threejs.org/) 0.160.0 (vẽ 3D) |

## Mục tiêu

Sau buổi học, sinh viên có thể:

1. Giải thích quy trình của AR dựa trên ảnh: nhận diện marker → tính tư thế của tờ giấy → đặt vật thể 3D lên đó.
2. Đọc hiểu một cảnh three.js: `Scene`, `Camera`, `Renderer`, `Group`, `Mesh`, `Material` và ma trận biến đổi.
3. Giải thích kỹ thuật **occluder** (vật che chỉ ghi độ sâu) tạo hiệu ứng "trái tim chui ra từ tờ giấy".
4. Viết hoạt ảnh theo thời gian, xử lý sự kiện chuột, chọn vật thể 3D bằng tia (raycasting).
5. Tự sửa và mở rộng chương trình: đổi thông số, làm tim đập, thêm bộ phận mới.

## Chuẩn bị

**Trợ giảng (trước buổi học):**
- Tải repo về và chạy thử trên máy mình theo [README.md](README.md).
- Kiểm tra mạng phòng lab **không chặn** `cdn.jsdelivr.net`: mở được trang là được.
- In ảnh `Assets/Markers/heart_marker2.jpg` trên giấy A4 mờ (không bóng), mỗi nhóm một tờ, in dư vài tờ.
- Chuẩn bị máy chiếu để demo phần mở đầu.
- Kiểm tra máy phòng lab đã có Python 3 (gõ `python --version` trong Command Prompt).

**Sinh viên:**
- Laptop có webcam, Chrome hoặc Edge, Python 3, một trình soạn code (VS Code hoặc tương tự).
- Tải project: https://github.com/luongduy0703/human_heart_vr → **Code → Download ZIP**, rồi giải nén.

> **Gợi ý cho trợ giảng:** chia nhóm 2–3 sinh viên, mỗi nhóm một tờ marker. Nếu thiếu giấy in, sinh viên có thể mở ảnh marker trên điện thoại (chỉnh sáng tối đa) rồi giơ trước webcam.

## Kế hoạch buổi học

| Thời gian | Hoạt động | Hình thức |
|---|---|---|
| 0–10 phút | Mở đầu: demo sản phẩm, đặt câu hỏi "làm sao máy biết tờ giấy nằm ở đâu?" | Trợ giảng demo trên máy chiếu |
| 10–25 phút | **Phần 1:** chạy bản demo trên máy mình | Sinh viên làm, trợ giảng đi hỗ trợ |
| 25–45 phút | **Phần 2:** kiến thức nền | Trợ giảng giảng, có hình minh họa |
| 45–75 phút | **Phần 3:** đọc hiểu mã nguồn | Trợ giảng dẫn dắt, sinh viên trả lời câu hỏi nhanh |
| 75–110 phút | **Phần 4:** bài tập thực hành | Làm theo nhóm |
| 110–120 phút | **Phần 5:** tổng kết, câu hỏi ôn tập | Thảo luận chung |

---

## Phần 1 – Chạy thử bản demo (15 phút)

1. Vào thư mục `web` của project, **nháy đúp `start.bat`**. Một cửa sổ đen hiện ra (web server) và trình duyệt tự mở `http://localhost:8000/web/`.
2. Trình duyệt hỏi quyền dùng camera: bấm **Cho phép**.
3. Đợi khung góc dưới báo *"Đưa tờ giấy in marker vào trước webcam."* (lần đầu mất vài giây để chuẩn bị marker).
4. Giơ tờ giấy vào **giữa khung hình**, cách webcam 30–50 cm: tim trồi lên trong 4 giây.
5. Thử: kéo chuột để xoay, lăn chuột để zoom, bấm vào một chấm màu để xem tên, bấm nút **Đặt lại góc nhìn**.
6. Che tờ giấy hơn 2 giây rồi đưa lại: tim trồi lên lại từ đầu.

**Sinh viên đạt khi:** thấy tim trồi lên từ tờ giấy, xoay được, zoom được và bấm hiện được tên ít nhất 3 bộ phận.

> **Gợi ý cho trợ giảng – lỗi hay gặp ở phần này:**
> - Mở trực tiếp file `index.html` (địa chỉ `file:///…`): trang báo lỗi webcam. Phải mở qua `http://localhost:8000/web/`.
> - Không hiện hộp hỏi quyền camera hoặc đã lỡ bấm Chặn: bấm biểu tượng camera trên thanh địa chỉ, chọn Cho phép rồi tải lại trang.
> - Webcam đang bị Zoom, Teams hay ứng dụng Camera dùng: đóng các ứng dụng đó.
> - Tim không hiện: tờ giấy nằm sát mép khung hình, quá xa, phòng tối hoặc giấy bị lóa. Nhắc sinh viên giơ giấy vào giữa khung hình.
> - Cửa sổ đen báo cổng 8000 đang bận: sửa `set PORT=8000` trong `start.bat` thành `8080`.

---

## Phần 2 – Kiến thức nền (20 phút)

### 2.1. AR dựa trên ảnh hoạt động thế nào

```
 Khung hình webcam
        │
        ▼
 Tìm các điểm đặc trưng (góc, nét chữ, hoa văn…)
        │
        ▼
 So khớp với dữ liệu của ảnh marker (.mind)  ──►  không khớp: chưa thấy marker
        │ khớp
        ▼
 Tính tư thế (vị trí + hướng) của tờ giấy so với camera: một ma trận 4×4
        │
        ▼
 three.js vẽ trái tim theo ma trận đó, chồng lên hình webcam
```

- MindAR làm toàn bộ phần nhận diện. Trước khi chạy, ảnh marker phải được **chuyển** thành dữ liệu đặc trưng (định dạng `.mind`). Trang tự làm việc này ở lần chạy đầu rồi lưu trong trình duyệt.
- Ảnh marker tốt là ảnh **nhiều chi tiết, tương phản cao, không lặp họa tiết**. Ảnh trơn (một màu, ít nét) rất khó nhận diện.
- Tư thế đo được mỗi khung hình hơi "rung". MindAR làm mượt bằng bộ lọc **One Euro** (`FILTER_MIN_CF`, `FILTER_BETA` trong `main.js`).

### 2.2. Hệ tọa độ của tờ giấy (anchor)

```
              +y  (về phía đầu ảnh)
               ▲
               │      +z: vuông góc với tờ giấy, hướng ra phía camera
               │
   ────────────O────────────►  +x
               │
   O = tâm ảnh marker; 1 đơn vị = chiều rộng ảnh (19.6 cm khi in A4)
```

- Mặt giấy là mặt phẳng `z = 0`. Tim nằm ngửa: đáy chạm `z = 0`, mặt trước hướng về camera, các mạch máu lớn hướng về phía đầu ảnh.
- Tim dài khoảng 1.02 đơn vị (20 cm khi marker in rộng 19.6 cm) và dày 0.56 đơn vị (khoảng 11 cm).
- Vì đơn vị là "chiều rộng ảnh", **in marker to hay nhỏ thì tim to hay nhỏ theo**. MindAR không cần biết kích thước in thật.

### 2.3. Cây cảnh (scene graph) của bài

```
scene
├── HemisphereLight, DirectionalLight    ánh sáng
└── anchor.group      MindAR cập nhật ma trận theo tờ giấy mỗi khung hình
    ├── occluder      mặt giấy tàng hình (xem 2.4)
    └── heartRoot     hoạt ảnh trồi lên: dời theo trục z, to dần
        └── pivot     xoay và zoom quanh tâm tim
            └── content   dời ngược lại để tâm tim trùng pivot
                ├── heartMesh      lưới trái tim
                └── hotspotGroup   12 × (chấm màu + vùng bấm vô hình)
```

Mỗi `Group` lo **một việc**: `heartRoot` lo hoạt ảnh, `pivot` lo xoay/zoom. Nhờ vậy hai việc không ghi đè lên nhau. Con của một group thì đi theo group đó: hotspot nằm trong `content` nên xoay và zoom cùng trái tim.

### 2.4. Mẹo "chui ra từ tờ giấy": bộ đệm độ sâu và occluder

Khi vẽ, card đồ họa giữ một **bộ đệm độ sâu** (depth buffer): mỗi điểm ảnh nhớ khoảng cách tới vật gần camera nhất đã vẽ ở đó. Một điểm của vật mới chỉ được vẽ nếu nó **gần camera hơn** giá trị đang lưu.

Occluder là một mặt phẳng lớn đặt đúng mặt giấy (`z = 0`), với ba thiết lập:

| Thiết lập | Tác dụng |
|---|---|
| `colorWrite: false` | Không vẽ màu: nhìn xuyên qua vẫn thấy hình webcam |
| ghi độ sâu (mặc định) | Đánh dấu "ở đây đã có vật, cách camera bấy nhiêu" |
| `renderOrder = -1` | Được vẽ **trước** trái tim |

```
            camera
              │   nhìn vào tờ giấy
              ▼
        ┌── phần tim trên giấy ──┐    gần camera hơn occluder  →  được vẽ
  ══════╪════════════════════════╪══  mặt giấy z = 0 (occluder: chỉ ghi độ sâu)
        └── phần tim dưới giấy ──┘    xa hơn occluder  →  bị loại, chỗ đó vẫn là hình webcam
```

Dời trái tim từ dưới giấy lên dần (`heartRoot.position.z` từ âm về 0) thì ta thấy nó "chui ra" từ tờ giấy.

### 2.5. Vì sao phải chạy web server

- Trình duyệt chỉ cho dùng webcam (`getUserMedia`) khi trang mở qua `https://` hoặc `http://localhost`.
- Mở file trực tiếp (`file:///…`) thì không nạp được JavaScript module và không tải được mô hình.
- Server chạy ở **thư mục gốc của repo**, vì trang đọc `../Assets/Models/heart.glb` và `../Assets/Markers/heart_marker2.jpg`.

> **Gợi ý cho trợ giảng:** trước khi giảng mục 2.4, hỏi sinh viên: "Nếu chỉ dời tim từ dưới lên mà không có occluder thì ta sẽ thấy gì?" (Trả lời: thấy cả phần tim nằm dưới tờ giấy, như tim lơ lửng xuyên qua giấy.) Bài tập 2 ở phần 4 sẽ kiểm chứng điều này.

---

## Phần 3 – Đọc hiểu mã nguồn (30 phút)

Mở `web/main.js` trong trình soạn code. Luồng chạy chính:

```
main()
 ├─ loadTargets()          chuẩn bị marker (lần đầu) hoặc đọc lại từ IndexedDB
 ├─ GLTFLoader             tải mô hình heart.glb
 ├─ new MindARThree(...)   tạo bộ nhận diện + cảnh three.js
 ├─ buildScene()           dựng cảnh, gắn sự kiện tìm thấy / mất dấu marker
 ├─ setupInput()           chuột và cảm ứng
 ├─ mindar.start()         mở webcam, bắt đầu nhận diện
 └─ mỗi khung hình:  update(dt)  →  render  →  updateLabel()
```

### 3.1. Nạp thư viện (`index.html`)

`index.html` dùng **importmap** để đặt tên ngắn cho các thư viện tải từ CDN:

```html
<script type="importmap">
  {
    "imports": {
      "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
      "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/",
      "mindar-image": "https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-image.prod.js",
      "mindar-image-three": "https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-image-three.prod.js"
    }
  }
</script>
```

Nhờ đó `main.js` chỉ cần viết `import * as THREE from 'three'`. Phải giữ three.js ở 0.160.0, vì MindAR 1.2.5 dùng một hằng số (`sRGBEncoding`) đã bị xóa từ three.js r162.

### 3.2. Chuẩn bị marker – `loadTargets()`

```js
const cached = await cacheGet(CACHE_KEY);
if (cached) return cached;

const img = new Image();
img.src = MARKER_URL;
await img.decode();
const compiler = new Compiler();
await compiler.compileImageTargets([img], onProgress);
const data = compiler.exportData();
await cacheSet(CACHE_KEY, data);
return data;
```

`Compiler` của MindAR chuyển ảnh thành dữ liệu đặc trưng, rồi `cacheSet` lưu vào IndexedDB (kho dữ liệu của trình duyệt). Những lần sau `cacheGet` trả về ngay, không phải chuyển lại.

**Câu hỏi nhanh:** nếu đổi sang ảnh marker khác mà giữ nguyên `CACHE_KEY` thì chuyện gì xảy ra?

<details><summary>Đáp án</summary>

Trang vẫn đọc dữ liệu cũ đã lưu, tức dữ liệu của ảnh marker cũ, nên không nhận ra marker mới. Đổi ảnh thì phải đổi tên `CACHE_KEY`.
</details>

### 3.3. Tạo bộ nhận diện – `new MindARThree(...)`

```js
const mindar = new MindARThree({
  container,
  imageTargetSrc: URL.createObjectURL(new Blob([targets])),
  maxTrack: 1,
  uiError: 'no',
  filterMinCF: FILTER_MIN_CF,
  filterBeta: FILTER_BETA,
  missTolerance: MISS_TOLERANCE,
});
```

- `MindARThree` tạo sẵn `scene`, `camera`, `renderer`. Nó đặt video webcam phía sau canvas 3D trong suốt.
- **Camera của MindAR đứng yên ở gốc tọa độ.** Thứ di chuyển là `anchor.group`, nhóm đi theo tờ giấy (tạo bằng `mindar.addAnchor(0)` trong `buildScene`).
- `missTolerance: 10`: mất marker 10 khung hình liên tiếp mới coi là mất dấu, để tim không chớp tắt khi nhận diện hụt một vài khung.

### 3.4. Đặt trái tim lên giấy – `buildScene()`

```js
let heartMesh = null;
gltf.scene.traverse((o) => { if (o.isMesh && !heartMesh) heartMesh = o; });
heartMesh.removeFromParent();
heartMesh.position.set(0, 0, 0);
heartMesh.quaternion.identity();
heartMesh.scale.set(1, 1, 1);
heartMesh.geometry.applyMatrix4(new THREE.Matrix4().fromArray(HEART_MESH_MATRIX));
content.add(heartMesh);
```

- File glb tải từ Sketchfab có nhiều node trung gian. Ta chỉ lấy lưới tim (`isMesh`) và bỏ phần còn lại.
- `HEART_MESH_MATRIX` (trong `heart-data.js`) là phép **phóng đều 0.514 lần cộng một phép dời**. Mô hình gốc dài 1.986 đơn vị, ta cần dài 1.02 đơn vị (20 cm): 1.02 / 1.986 ≈ 0.514. Phép dời đặt tâm tim vào giữa ảnh và đáy tim chạm `z = 0`.
- `applyMatrix4` trên **geometry** "nướng" phép biến đổi vào các đỉnh, nên bản thân lưới không còn phép biến đổi nào.

### 3.5. Occluder

```js
const occluder = new THREE.Mesh(new THREE.PlaneGeometry(4, 4), new THREE.MeshBasicMaterial({ colorWrite: false }));
occluder.renderOrder = -1;
anchor.group.add(occluder);
```

Occluder là con của `anchor.group`, **không** phải con của `heartRoot`: nó phải đứng yên ở mặt giấy trong khi tim di chuyển. Kích thước 4 × 4 đơn vị (khoảng 78 cm) đủ che quanh tờ giấy.

**Câu hỏi nhanh:** vì sao cần `renderOrder = -1`?

<details><summary>Đáp án</summary>

Occluder phải ghi độ sâu **trước** khi vẽ tim. Nếu tim được vẽ trước, các điểm của tim đã nằm sẵn trên ảnh, occluder vẽ sau cũng không xóa được chúng.
</details>

### 3.6. Hoạt ảnh trồi lên

```js
function applyEmerge(s) {
  heartRoot.position.z = THREE.MathUtils.lerp(-SINK_DEPTH, 0, s);
  heartRoot.scale.setScalar(THREE.MathUtils.lerp(START_SCALE, 1, s));
}
```

```js
update(dt) {
  if (state.emergeT >= 1) return;
  state.emergeT = Math.min(1, state.emergeT + dt / EMERGE_SECONDS);
  const t = state.emergeT;
  applyEmerge(t * t * (3 - 2 * t));  // SmoothStep giống bản Unity
  if (t >= 1) hotspotGroup.visible = true;
},
```

- `dt` là số giây giữa hai khung hình. Cộng dồn `dt / EMERGE_SECONDS` thì tiến độ `t` đi từ 0 đến 1 trong đúng 4 giây, **không phụ thuộc máy nhanh hay chậm**.
- `t * t * (3 - 2 * t)` (SmoothStep) làm chuyển động "chậm – nhanh – chậm" tự nhiên hơn đi đều.
- Tim bắt đầu ở độ sâu `SINK_DEPTH` (12 cm dưới giấy) với kích thước 30%, rồi lên tới mặt giấy với kích thước thật.
- Khi tim lên hết thì mới bật `hotspotGroup` (các chấm màu).

Phát lại khi thấy lại marker:

```js
anchor.onTargetFound = () => {
  state.anchorVisible = true;
  const lostFor = performance.now() / 1000 - state.lostAt;
  if (!state.everFound || lostFor > REPLAY_AFTER_LOST) replay();
  state.everFound = true;
  setStatus('Đã thấy marker.');
};
```

**Câu hỏi nhanh:** vì sao chỉ phát lại khi mất dấu **quá 2 giây**, không phát lại mỗi lần mất dấu?

<details><summary>Đáp án</summary>

Nhận diện thỉnh thoảng hụt trong chốc lát (tay che, nghiêng giấy, ánh sáng đổi). Nếu lần nào mất dấu cũng phát lại thì tim cứ chui xuống rồi trồi lên liên tục, rất khó chịu.
</details>

### 3.7. Chọn hotspot bằng tia – `pick()`

```js
const r = container.getBoundingClientRect();
ndc.set(((clientX - r.left) / r.width) * 2 - 1, -((clientY - r.top) / r.height) * 2 + 1);
raycaster.setFromCamera(ndc, app.camera);
// Tia trúng tim trước (chấm bị tim che) thì không chọn gì
const hit = raycaster.intersectObjects(app.pickTargets, false)[0];
if (hit && hit.object.userData.hotspot) app.select(hit.object.userData.hotspot);
```

1. Đổi vị trí chuột (pixel) sang **tọa độ chuẩn hóa** từ -1 đến 1 (`ndc`).
2. `setFromCamera` tạo một tia từ camera đi qua điểm đó.
3. `intersectObjects` trả về danh sách vật bị tia cắt, **sắp theo khoảng cách**. Phần tử `[0]` là vật gần nhất.
4. `pickTargets` gồm lưới tim và 12 quả cầu vùng bấm. Nếu vật gần nhất là tim, tức chấm màu nằm khuất sau tim, thì không chọn gì.

Mỗi hotspot có hai quả cầu: **chấm màu** đường kính 0.8 cm để nhìn, và **vùng bấm** bán kính 1.5 cm không vẽ ra (`visible: false` trong material) để bấm cho dễ trúng. Tia vẫn cắt được vật có material không hiển thị.

`select()` chỉ cho **một nhãn** mỗi lúc: bỏ phóng to chấm cũ, phóng to chấm mới 1.6 lần, đổi chữ trên nhãn. Bấm lại chấm đang chọn thì bỏ chọn.

### 3.8. Nhãn tên – `updateLabel()`

```js
spot.marker.getWorldPosition(projected).project(camera);
const x = (projected.x * 0.5 + 0.5) * container.clientWidth;
const y = (-projected.y * 0.5 + 0.5) * container.clientHeight;
labelEl.style.transform = `translate(${x}px, ${y}px) translate(-50%, calc(-100% - 16px))`;
```

Nhãn là một thẻ HTML (`#label`), không phải chữ 3D, nên luôn nét và luôn quay về người xem. Mỗi khung hình, vị trí 3D của chấm được **chiếu** (`project`) ra tọa độ màn hình (từ -1 đến 1), đổi sang pixel, rồi đặt nhãn ngay phía trên chấm. Đây là phép ngược với bước 1 của mục 3.7.

### 3.9. Xoay và zoom – `rotate()`, `applyZoom()`

```js
inv.copy(app.anchor.group.matrixWorld).invert();
axis.setFromMatrixColumn(app.camera.matrixWorld, 1).transformDirection(inv);  // trục dọc của camera
q.setFromAxisAngle(axis, dx * radPerPixel);
app.pivot.quaternion.premultiply(q);
```

- Kéo ngang thì tim xoay quanh **trục dọc của màn hình**, kéo dọc thì xoay quanh **trục ngang**, bất kể tờ giấy đang nghiêng thế nào.
- Trục của camera nằm trong hệ tọa độ thế giới, còn `pivot` nằm trong hệ tọa độ tờ giấy. Vì vậy phải đổi trục sang hệ tờ giấy bằng ma trận nghịch đảo (`invert`) của `anchor.group`.
- `premultiply` áp phép xoay mới **chồng lên** góc xoay hiện có, theo trục của hệ cha.

```js
pivot.scale.setScalar(state.zoom);
// pivot nằm ở tâm tim: nâng lên theo tỉ lệ để đáy tim luôn nằm trên mặt giấy
pivot.position.set(HEART_CENTER[0], HEART_CENTER[1], HEART_BOTTOM + (HEART_CENTER[2] - HEART_BOTTOM) * state.zoom);
```

Zoom quanh tâm tim thì phần dưới tim sẽ lún xuống giấy và bị occluder che. Nâng `pivot` lên theo tỉ lệ zoom thì đáy tim luôn nằm đúng trên mặt giấy.

> **Hoạt động gợi ý:** mở `http://localhost:8000/web/?debug`, nhấn **F12** → tab **Console**, gõ `heartApp.app.state` rồi Enter. Vừa lăn chuột vừa gõ lại để xem `zoom` thay đổi; bấm một chấm rồi xem `selected`.

---

## Phần 4 – Bài tập thực hành (35 phút)

**Cách làm chung:** sửa file trong `web/` bằng trình soạn code, lưu lại, rồi tải lại trang bằng **Ctrl+F5** (tải lại bỏ qua bộ nhớ đệm). Nếu chỉ nhấn F5, trình duyệt có thể vẫn chạy bản code cũ.

> **Gợi ý cho trợ giảng:** yêu cầu sinh viên **đoán kết quả trước** khi sửa, rồi mới chạy để kiểm chứng. Sau mỗi bài, cho sinh viên hoàn tác (Ctrl+Z) để bài sau bắt đầu từ code gốc. Bài 1–2 bắt buộc, bài 3–5 chọn tối thiểu hai bài, bài 6 dành cho nhóm làm nhanh.

### Bài 1 (dễ) – Đổi thông số hoạt ảnh

Ở đầu `main.js`, đổi:
- `EMERGE_SECONDS` từ `4` thành `2`,
- `SINK_DEPTH` từ `0.12 / MARKER_WIDTH_M` thành `0.06 / MARKER_WIDTH_M`,
- `START_SCALE` từ `0.3` thành `1`.

Đổi lần lượt từng thông số rồi chạy thử, sau đó đổi cả ba cùng lúc. Trả lời: mỗi thay đổi ảnh hưởng thế nào tới cảnh tim trồi lên? Khi đổi cả ba, lúc vừa thấy marker, tim có còn bị che hết dưới mặt giấy không? Vì sao?

<details><summary>Đáp án</summary>

- `EMERGE_SECONDS = 2`: tim trồi lên nhanh gấp đôi.
- Chỉ đổi độ sâu thành 6 cm: lúc đầu tim vẫn nhỏ bằng 30% (dày khoảng 3.3 cm) nên vẫn bị che hết; tim chỉ trồi lên một quãng ngắn hơn.
- `START_SCALE = 1`: tim không lớn dần, chỉ trồi lên với kích thước thật.
- Đổi cả ba: tim dày khoảng 11 cm mà chỉ nằm sâu 6 cm, nên ngay lúc đầu đã có khoảng 5 cm tim nhô khỏi mặt giấy, như xuất hiện đột ngột một nửa. Muốn tim bị che hết lúc bắt đầu thì độ sâu phải lớn hơn chiều dày của tim lúc đó (chiều dày nhân `START_SCALE`).
</details>

### Bài 2 (dễ) – Tìm hiểu vai trò của occluder

Trong `buildScene()`:
1. Đổi `new THREE.MeshBasicMaterial({ colorWrite: false })` thành `new THREE.MeshBasicMaterial({ color: 0x00aaff })` rồi chạy.
2. Trả lại như cũ, rồi thêm dòng `occluder.visible = false;` ngay sau `anchor.group.add(occluder);` rồi chạy.

Mô tả điều quan sát được ở mỗi lần. Dựa vào mục 2.4, giải thích.

<details><summary>Đáp án</summary>

1. Thấy một hình vuông xanh lớn (khoảng 78 cm) nằm đúng mặt giấy, che mất hình webcam xung quanh tờ giấy. Phần tim dưới giấy vẫn bị che. Đây chính là occluder, bình thường nó "vô hình" nhờ `colorWrite: false`.
2. Không còn occluder: lúc tim trồi lên, thấy cả phần tim nằm dưới mặt giấy, như tim xuyên qua tờ giấy. Hiệu ứng "chui ra" mất hẳn.
</details>

### Bài 3 (trung bình) – Trái tim đập

Làm trái tim **đập khoảng 72 nhịp/phút**: mỗi nhịp, tim phồng lên vài phần trăm rồi xẹp dần.

Gợi ý:
- Sửa hàm `update(dt)` trong `buildScene()`. Phần làm tim đập phải đặt **trước** dòng `if (state.emergeT >= 1) return;`, vì dòng đó kết thúc hàm khi tim đã trồi lên xong.
- Phóng to group `content`: đáy tim nằm ở gốc tọa độ của `content`, nên tim phồng lên mà đáy vẫn nằm trên giấy, và các chấm màu đi theo bề mặt.
- Một nhịp kéo dài 60 / 72 giây. Dùng `Math.exp(-phase * 10)` để có dạng "phồng nhanh, xẹp chậm".

<details><summary>Đáp án</summary>

```js
update(dt) {
  // Bài 3: tim đập 72 nhịp/phút, mỗi nhịp phồng lên 4% rồi xẹp dần
  state.beatTime = (state.beatTime ?? 0) + dt;
  const phase = (state.beatTime * 72 / 60) % 1;       // vị trí trong một nhịp, từ 0 đến 1
  content.scale.setScalar(1 + 0.04 * Math.exp(-phase * 10));

  if (state.emergeT >= 1) return;
  // … phần còn lại giữ nguyên
},
```

Thử đổi `0.04` thành `0.08` để thấy rõ hơn, đổi `72` để tim đập nhanh hoặc chậm.
</details>

### Bài 4 (trung bình) – Nhãn tiếng Việt

Sửa `heart-data.js` để nhãn hiện cả tên tiếng Việt và tiếng Anh, ví dụ `Tâm nhĩ phải (Right atrium)`.

<details><summary>Đáp án</summary>

Sửa trường `label` của từng phần tử trong `HOTSPOTS`, ví dụ:

```js
{ id: 'TamNhiPhai', label: 'Tâm nhĩ phải (Right atrium)', color: [0.25, 0.45, 1], position: [-0.219388, 0.132653, 0.432219] },
```

Tên các bộ phận: Tâm nhĩ phải/trái, Tâm thất phải/trái, Động mạch chủ, Thân động mạch phổi, Tĩnh mạch chủ trên/dưới, Tĩnh mạch phổi, Động mạch vành phải/trái, Mỏm tim. Nhãn là thẻ HTML nên hiển thị tiếng Việt có dấu bình thường.
</details>

### Bài 5 (khó) – Thêm bộ phận thứ 13

Thêm một hotspot mới, ví dụ **tiểu nhĩ trái** (Left auricle): phần "tai" nhỏ của tâm nhĩ trái, nằm cạnh thân động mạch phổi.

Khó nhất là tìm **tọa độ** của điểm mới. Gợi ý: sửa `pick()` để khi bấm lên bề mặt tim (không trúng chấm nào) thì in tọa độ chỗ bấm ra Console (F12), rồi chép tọa độ đó vào `HOTSPOTS`.

- Tọa độ cần lấy trong hệ của lưới tim: dùng `hit.object.worldToLocal(...)`.
- Các chấm có sẵn nằm cao hơn bề mặt 3 mm: dời điểm theo pháp tuyến `hit.face.normal` một đoạn `0.003 / 0.196` đơn vị.

<details><summary>Đáp án</summary>

Trong `pick()`, thêm đoạn sau ngay trước dòng `if (hit && hit.object.userData.hotspot) …`:

```js
// Bài 5: bấm lên bề mặt tim thì in tọa độ để thêm hotspot mới
if (hit && hit.object === app.pickTargets[0]) {        // pickTargets[0] là lưới tim
  const p = hit.object.worldToLocal(hit.point.clone());
  p.add(hit.face.normal.clone().multiplyScalar(0.003 / 0.196));   // nhô khỏi bề mặt 3 mm
  console.log('position:', p.toArray().map((v) => Number(v.toFixed(4))));
}
```

Chạy, đợi tim trồi lên hết, bấm lên chỗ tiểu nhĩ trái, đọc tọa độ trong Console. Sau đó thêm vào cuối mảng `HOTSPOTS` trong `heart-data.js`:

```js
{ id: 'TieuNhiTrai', label: 'Left auricle', color: [0.95, 0.85, 0.3], position: [x, y, z] },
```

với `[x, y, z]` là tọa độ vừa đọc được. Tải lại trang (Ctrl+F5) rồi kiểm tra chấm mới.
</details>

### Bài 6 (nâng cao, tùy chọn) – Đổi marker

Dùng một ảnh khác làm marker, ví dụ bìa một cuốn sách hoặc một tấm poster.

Gợi ý:
- Chép ảnh vào `Assets/Markers/`, sửa `MARKER_URL` trong `main.js`.
- **Đổi tên `CACHE_KEY`** (xem câu hỏi nhanh ở mục 3.2).
- Chọn ảnh nhiều chi tiết, tương phản cao. Tim được đặt ở tâm ảnh và dài bằng khoảng chiều rộng ảnh.

Trả lời: với ảnh ngang (rộng hơn cao), tim trông thế nào so với ảnh dọc? Vì sao?

<details><summary>Đáp án</summary>

Tim dài khoảng 1.02 lần chiều rộng ảnh, theo trục dọc của ảnh. Ảnh dọc (như marker gốc, cao 1.31 lần chiều rộng) chứa trọn trái tim. Ảnh ngang thấp hơn chiều dài tim, nên tim tràn ra ngoài mép trên và mép dưới của ảnh.
</details>

---

## Phần 5 – Tổng kết và câu hỏi ôn tập (10 phút)

**Tóm tắt:** MindAR nhận diện ảnh marker và cho tư thế của tờ giấy. three.js đặt trái tim theo tư thế đó. Occluder chỉ ghi độ sâu, nhờ đó tim như chui ra từ tờ giấy. Tia (raycasting) giúp bấm chọn vật 3D. Các `Group` lồng nhau tách riêng hoạt ảnh và thao tác xoay, zoom.

**Câu hỏi ôn tập:**

1. MindAR cần gì để nhận ra marker? Vì sao ảnh trơn khó nhận diện?
2. Trong hệ tọa độ của tờ giấy, 1 đơn vị dài bao nhiêu? In marker nhỏ bằng nửa thì tim thay đổi thế nào?
3. Occluder không vẽ màu thì làm sao che được trái tim?
4. Vì sao occluder là con của `anchor.group` mà không phải con của `heartRoot`?
5. Vì sao chia `heartRoot`, `pivot`, `content` thành ba group riêng?
6. Tia trúng trái tim trước rồi mới tới chấm màu thì chương trình xử lý thế nào? Vì sao làm vậy?
7. Vì sao phải chạy web server, không mở thẳng file `index.html`?
8. So với bản Unity trên điện thoại (thư mục `Assets/`), bản web giống và khác nhau ở đâu?

<details><summary>Gợi ý đáp án (cho trợ giảng)</summary>

1. Cần dữ liệu đặc trưng của ảnh (`.mind`) và khung hình có đủ điểm đặc trưng khớp với dữ liệu đó. Ảnh trơn có quá ít điểm đặc trưng để so khớp và tính tư thế.
2. 1 đơn vị = chiều rộng ảnh marker. In nhỏ bằng nửa thì tim cũng nhỏ bằng nửa (dài khoảng 10 cm thay vì 20 cm), vì mọi kích thước tính theo chiều rộng ảnh.
3. Nó ghi độ sâu và được vẽ trước, nên các điểm của tim nằm xa camera hơn mặt giấy không qua được phép thử độ sâu và không được vẽ.
4. Occluder phải đứng yên ở mặt giấy. Nếu là con của `heartRoot`, nó sẽ di chuyển và phóng to theo hoạt ảnh của tim.
5. Mỗi group lo một việc (trồi lên / xoay, zoom / dời về tâm), các phép biến đổi nhân với nhau theo thứ tự cha – con mà không ghi đè lên nhau.
6. Không chọn gì, vì khi đó chấm màu nằm khuất phía sau trái tim. Người dùng không nhìn thấy chấm đó nên không thể định bấm vào nó.
7. Trình duyệt chỉ cho dùng webcam qua `https` hoặc `localhost`, và không nạp module, không tải mô hình từ `file://`.
8. Giống: cùng mô hình, cùng marker, cùng 12 hotspot, cùng hiệu ứng occluder, cùng logic trồi lên và phát lại. Khác: bản Unity dùng ARCore trên điện thoại (cần biết kích thước in thật, có chế độ kính VR); bản web dùng MindAR trên trình duyệt, đơn vị theo chiều rộng ảnh, điều khiển bằng chuột.
</details>

---

## Phụ lục

### A. Bảng tra nhanh khi sinh viên gặp lỗi

| Hiện tượng | Nguyên nhân thường gặp | Cách xử lý |
|---|---|---|
| Báo "chỉ cho dùng webcam khi mở trang qua http://localhost…" | Mở file trực tiếp | Mở `http://localhost:8000/web/` |
| Báo "Không tải được mô hình hoặc ảnh marker" | Server chạy sai thư mục | Chạy server ở thư mục gốc repo (dùng `start.bat`) |
| Báo "Không mở được webcam" | Chưa cho quyền camera, hoặc ứng dụng khác đang dùng webcam | Cho phép camera trên thanh địa chỉ, đóng Zoom/Teams/Camera |
| Khung quét vẫn hiện, không thấy tim | Giấy ở mép khung hình, quá xa, tối, lóa | Đưa giấy vào giữa, lại gần, thêm ánh sáng |
| Sửa code mà không thấy thay đổi | Trình duyệt dùng bản cũ trong bộ nhớ đệm | Tải lại bằng Ctrl+F5 |
| Trang trắng, Console (F12) báo lỗi cú pháp | Gõ sai khi làm bài tập | Đọc dòng lỗi trong Console, đối chiếu với đáp án |
| Console báo lỗi tải từ `cdn.jsdelivr.net` | Không có mạng hoặc mạng chặn CDN | Kiểm tra kết nối, đổi mạng |

Xem thêm bảng xử lý sự cố đầy đủ trong [README.md](README.md#xử-lý-sự-cố).

### B. Tài liệu tham khảo

- MindAR – Image tracking với three.js: https://hiukim.github.io/mind-ar-js-doc/
- three.js – `Object3D`, `Raycaster`, `Material.colorWrite`: https://threejs.org/docs/
- Bộ lọc One Euro (làm mượt tín hiệu nhiễu): https://gery.casiez.net/1euro/
- Mô hình tim: [Human Heart 3D Model](https://sketchfab.com/3d-models/human-heart-3d-model-anatomy-medical-project-5da08bb137014f0392c69f9997f777cd) của Mesh-Magnet, giấy phép CC BY 4.0.

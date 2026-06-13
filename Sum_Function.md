# สรุป Function / Feature สำหรับนำไปพัฒนาต่อ

เอกสารนี้สรุปการทำงานของฟีเจอร์สำคัญจาก `Football-Control-Panel-V.2-stream-alone.html` และ `fcp_v2_assets/main.js` เพื่อใช้เป็น context ให้ AI หรือผู้พัฒนานำไปสร้างต่อในโปรเจกต์ใหม่

## 1) ปุ่มปรับเวลา `adjustTimeBtn`

### HTML ที่เกี่ยวข้อง

```html
<button id="adjustTimeBtn" class="btn btn-danger">ปรับเวลา<i class=""></i></button>
```

Popup ที่ถูกเปิด:

```html
<div id="presetTimePopup" class="popup-modal">
    <h3><i class="fas fa-clock"></i> เลือกเวลาครึ่งหลัง</h3>
    <p>เลือกเวลาที่ต้องการสำหรับครึ่งหลัง</p>
    <button class="btn btn-primary preset-time-btn" data-seconds="0">0 นาที</button>
    <button class="btn btn-primary preset-time-btn" data-seconds="720">12 นาที</button>
    <button class="btn btn-primary preset-time-btn" data-seconds="900">15 นาที</button>
    <button class="btn btn-primary preset-time-btn" data-seconds="1200">20 นาที</button>
    <button class="btn btn-primary preset-time-btn" data-seconds="1500">25 นาที</button>
    <button class="btn btn-primary preset-time-btn" data-seconds="2700">45 นาที</button>
</div>
```

### State / Function ที่ใช้

- `countdownStartTime`: เก็บเวลาเริ่มต้นของครึ่งหลังเป็นวินาที ค่า default คือ `2700` หรือ 45 นาที
- `openPopup(popup)`: แสดง overlay และ popup
- `closeAllPopups()`: ปิด popup ทั้งหมด รวมถึง `presetTimePopup`
- `.preset-time-btn`: ปุ่ม preset เวลาแต่ละค่า ใช้ `data-seconds` เป็นค่าที่จะบันทึก
- `localStorage.setItem('countdownStartTime', countdownStartTime)`: จำค่าที่เลือกไว้
- `startTimer2()`: เมื่อต้องเริ่มครึ่งหลัง จะนำ `countdownStartTime` ไปตั้งค่า `timer`

### Flow การทำงาน

1. ผู้ใช้กดปุ่ม `adjustTimeBtn`
2. Event listener เรียก `openPopup(elements.presetTimePopup)`
3. ผู้ใช้เลือก preset เช่น 12, 15, 20, 25, 45 นาที
4. JS อ่านค่า `data-seconds` จากปุ่มที่กด
5. อัปเดต `countdownStartTime`
6. บันทึกค่าลง `localStorage`
7. ปิด popup ด้วย `closeAllPopups()`
8. แสดง toast แจ้งว่าเปลี่ยนเวลาแล้ว
9. เมื่อกดเริ่มครึ่งหลัง `startTimer2()` จะเริ่มจากเวลาที่เลือกไว้

### หมายเหตุสำหรับพัฒนาต่อ

- ฟีเจอร์นี้เป็นการตั้งเวลาเริ่มต้นสำหรับครึ่งหลัง ไม่ได้เปลี่ยนเวลาที่กำลังเดินอยู่ทันที
- ถ้าต้องการให้กด preset แล้วอัปเดตเวลาบนหน้าจอทันที ให้เพิ่ม `timer = countdownStartTime; updateTimerDisplay();`
- ในโค้ดยังมี `timeSettingsPopup`, `saveTimeSettings()`, `saveAndUpdateTime()` สำหรับกรอกเวลาเอง แต่ปุ่ม `adjustTimeBtn` ปัจจุบันเปิด `presetTimePopup`

## 2) ปุ่มเลือกทีมจาก Excel `editBtnA`

### HTML ที่เกี่ยวข้อง

```html
<button class="btn btn-outline-danger" id="editBtnA" title="เลือกทีมจาก Excel">
    <i class="fas fa-list-ul"></i>
</button>
```

Popup ที่ใช้เลือกทีม:

```html
<div id="teamSelectPopup" class="popup-modal">
    <h3><i class="fas fa-list-ul"></i> <span id="teamSelectTitle">เลือกทีม</span></h3>
    <input type="text" id="teamSelectSearch" class="team-select-search" placeholder="ค้นหาชื่อทีม">
    <div id="teamSelectList" class="team-select-list"></div>
    <button id="closeTeamSelectBtn" class="btn-secondary">ปิด</button>
</div>
```

### State / Function ที่ใช้

- `sheetData`: ข้อมูล Excel ที่ถูกอ่านเข้ามาแล้ว โดยแถวแรกเป็น header
- `teamSelectTarget`: ระบุว่าจะเลือกทีมให้ฝั่ง `A` หรือ `B`
- `openTeamSelector(target)`: เปิด popup เลือกทีม
- `getTeamsFromExcel()`: ดึงรายชื่อทีมไม่ซ้ำจาก Excel
- `renderTeamSelectList()`: render รายชื่อทีมใน popup และ filter จากช่องค้นหา
- `applyTeamSelection(target, teamInfo)`: นำทีมที่เลือกไปอัปเดต UI และ OBS
- `updateTeamUI(team, name, logoFile, color1, color2)`: อัปเดตชื่อทีม โลโก้ สี initials และ OBS source
- `getTeamInitials(name)`: สร้างตัวย่อทีมไว้แสดงแทนโลโก้

### Column ใน Excel ที่ฟีเจอร์นี้ใช้

- `TeamA`, `TeamB`: ชื่อทีม
- `LogoA`, `LogoB`: ชื่อไฟล์โลโก้
- `ColorA`, `ColorB`: สีหลัก
- `ColorA2`, `ColorB2`: สีรอง

### Flow การทำงาน

1. ผู้ใช้ต้องโหลดไฟล์ Excel ก่อน เพื่อให้มีข้อมูลใน `sheetData`
2. ผู้ใช้กด `editBtnA`
3. Event listener เรียก `openTeamSelector('A')`
4. ถ้าไม่มีข้อมูล Excel จะแสดง toast ให้ upload/load ไฟล์ก่อน
5. ตั้ง `teamSelectTarget = 'A'`
6. เปลี่ยนหัว popup เป็น `เลือกทีม A`
7. เรียก `renderTeamSelectList()`
8. `getTeamsFromExcel()` อ่านทีมจากทั้ง column `TeamA` และ `TeamB`
9. รายชื่อทีมถูก deduplicate ด้วยชื่อทีมแบบ lower-case
10. แสดงทีมเป็นปุ่ม พร้อมโลโก้ถ้ามีไฟล์โลโก้
11. ผู้ใช้ค้นหาทีมได้จาก `teamSelectSearch`
12. เมื่อกดทีมใดทีมหนึ่ง จะเรียก `applyTeamSelection('A', teamInfo)`
13. อัปเดตชื่อ โลโก้ สี และ OBS source ของทีม A
14. ปิด popup และแสดง toast

### พฤติกรรมสำคัญ

- ปุ่ม `editBtnA` ไม่ได้เข้าโหมดพิมพ์ชื่อทีมเองแล้ว แต่เปลี่ยนเป็นเปิด popup เลือกทีมจาก Excel
- ถ้าไม่มี `teamSelectPopup` ใน HTML ฟังก์ชัน fallback จะเรียก `enterEditMode(target)`
- การเลือกทีมจาก popup จะเปลี่ยนเฉพาะทีมฝั่งที่เลือก ไม่ได้เปลี่ยน Match ID, label, score หรือข้อมูลแมตช์ทั้งหมด
- ถ้า Excel ไม่มีโลโก้ของทีมที่เลือก จะใช้โลโก้เดิมของฝั่งนั้นเป็น fallback
- สีที่แสดงจริงยังผ่าน `updateTeamUI()` ซึ่งจะเช็กสีที่ผู้ใช้เคยบันทึกไว้ใน `localStorage` ผ่าน `getTeamColors(name)` ก่อนใช้สีจาก Excel

## 3) จำไฟล์ Excel ล่าสุด และ refresh ก่อนโหลด Match ID

### HTML / Button ที่เกี่ยวข้อง

```html
<button id="loadBtn" class="btn-primary">
    <i class="fas fa-check"></i> <span data-lang="load">โหลด</span>
</button>
```

ปุ่มก่อนหน้า/ต่อไปถูกสร้างด้วย JS:

```js
prevBtn.id = 'prev-match-btn';
nextBtn.id = 'next-match-btn';
```

### State / Function ที่ใช้

- `sheetData`: ข้อมูล Excel ที่ parse แล้ว
- `selectedExcelFile`: จำไฟล์ Excel ที่เลือกไว้ล่าสุด
- `selectedExcelHandle`: จำ file handle จาก File System Access API ถ้า browser รองรับ
- `handleExcel()`: เลือกไฟล์ Excel ครั้งแรก
- `readExcelFile(file, showSuccessToast = true)`: อ่านไฟล์ด้วย `FileReader` และ parse ด้วย `XLSX.read`
- `refreshExcelData()`: อ่าน Excel ใหม่จากไฟล์/handle เดิมก่อนโหลดข้อมูล
- `loadCurrentMatch()`: refresh Excel ก่อน แล้วค่อยเรียก `applyMatch()`
- `applyMatch()`: หา Match ID จาก `sheetData` แล้วอัปเดตทีม, โลโก้, สี, label, OBS, reset timer/score

### Flow การเลือกไฟล์ Excel

1. ผู้ใช้กดปุ่ม Excel
2. `handleExcel()` ทำงาน
3. ถ้า browser รองรับ `window.showOpenFilePicker`
   - เปิด file picker
   - เก็บ handle ไว้ใน `selectedExcelHandle`
   - อ่านไฟล์ล่าสุดด้วย `selectedExcelHandle.getFile()`
   - เก็บ file ไว้ใน `selectedExcelFile`
   - parse Excel เข้า `sheetData`
4. ถ้า browser ไม่รองรับ `showOpenFilePicker`
   - ใช้ `<input type="file">` แบบเดิม
   - เก็บไฟล์ไว้ใน `selectedExcelFile`
   - parse Excel เข้า `sheetData`

### Flow เมื่อกดปุ่มโหลด

1. ผู้ใช้กด `loadBtn`
2. Event listener เรียก `loadCurrentMatch()`
3. `loadCurrentMatch()` เรียก `refreshExcelData()`
4. `refreshExcelData()` พยายามอ่าน Excel ใหม่
   - ถ้ามี `selectedExcelHandle` จะเรียก `selectedExcelHandle.getFile()` เพื่อดึงไฟล์ล่าสุดจาก disk
   - ถ้าไม่มี handle แต่มี `selectedExcelFile` จะอ่านจาก file object เดิม
   - ถ้าไม่มีไฟล์ จะ toast แจ้งให้ upload/load ไฟล์ก่อน
5. ถ้า refresh สำเร็จ จึงเรียก `applyMatch()`
6. `applyMatch()` อ่าน Match ID จาก input `matchID`
7. หา row ใน Excel ที่ column แรกตรงกับ Match ID
8. อัปเดตชื่อทีม โลโก้ สี label OBS source
9. reset timer, score และ half กลับเป็น `1st`

### Flow เมื่อกดปุ่มก่อนหน้า/ต่อไป

ปุ่ม `prev-match-btn`:

1. อ่านค่า `matchID` ปัจจุบัน
2. ถ้า ID มากกว่า 1 จะลดค่า `matchID` ลง 1
3. เรียก `loadCurrentMatch()`
4. จึง refresh Excel ก่อน แล้วค่อยโหลด Match ID ใหม่

ปุ่ม `next-match-btn`:

1. อ่านค่า `matchID` ปัจจุบัน
2. เพิ่มค่า `matchID` ขึ้น 1
3. เรียก `loadCurrentMatch()`
4. จึง refresh Excel ก่อน แล้วค่อยโหลด Match ID ใหม่

### ข้อควรระวังสำหรับนำไปใช้ต่อ

- การ refresh ไฟล์ล่าสุดจาก disk ทำงานดีที่สุดเมื่อใช้ `showOpenFilePicker` เพราะมี `selectedExcelHandle.getFile()`
- ถ้า environment ไม่รองรับ File System Access API และใช้ fallback `<input type="file">` การอ่านซ้ำจาก `selectedExcelFile` อาจไม่ได้เห็นไฟล์ที่ถูกแก้จากภายนอกเสมอไป ผู้ใช้อาจต้องเลือกไฟล์ใหม่
- `applyMatch()` คาดว่าแถวแรกของ Excel เป็น header และ column แรกคือ Match ID
- Header ที่ใช้ใน `applyMatch()` เป็นแบบ case-sensitive เช่น `TeamA`, `TeamB`, `LogoA`, `LogoB`, `ColorA`, `ColorB`, `ColorA2`, `ColorB2`, `label1`, `label2`, `label3`
- ปุ่มโหลด/ก่อนหน้า/ต่อไปควรเรียกผ่าน `loadCurrentMatch()` เสมอ ไม่ควรเรียก `applyMatch()` ตรง ๆ ถ้าต้องการให้ refresh Excel ก่อนทุกครั้ง


ETTV 南部中心排班系統 (SNG Schedule System)這是一套專為 ETTV 南部中心設計的線上排班與畫假系統。採用前端 HTML5 介面搭配 Google Sheets 作為雲端資料庫，實現輕量化、即時同步且跨裝置使用的管理工具。🚀 系統亮點動態 SNG 封面設計：具備動畫效果的衛星轉播車圖示，貼合新聞中心氛圍。跨月排班邏輯：自動以當月第一個週日作為起始，至下月第一個週日的前一天（週六），確保週次完整。雙重身分控管：員工前台：僅能選擇自己姓名進行「排休」申請，並檢視班表。主管後台：透過密碼 (12345) 登入，具備完整指派權限。今日出勤儀表板：自動偵測當日日期，列出上班、出差與休假人員。嚴謹的防呆機制：支持「單日鎖定」（禁休）。支持「整月鎖定」（發佈後禁止更改）。自動化統計：即時計算每人當月休假天數，顯示於左側固定欄位。🛠️ 技術架構前端：HTML5, Tailwind CSS, FontAwesome 6, JavaScript (Vanilla JS)。後端：Google Apps Script (GAS)。資料庫：Google Sheets (試算表)。📖 設定教學第一步：建立 Google 試算表資料庫在 Google Drive 建立一個新的試算表。不需要建立任何內容，保持空白即可。第二步：部署 Google Apps Script點擊試算表選單：擴充功能 > Apps Script。刪除原有代碼，貼入以下後端程式碼：function doGet(e) {
  const monthKey = (e && e.parameter && e.parameter.month) ? e.parameter.month : '2026-4';
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  let sheet = ss.getSheetByName(monthKey);
  if (!sheet) return ContentService.createTextOutput(JSON.stringify({ status: 'empty' })).setMimeType(ContentService.MimeType.JSON);
  const data = sheet.getDataRange().getValues();
  if (data.length < 2) return ContentService.createTextOutput(JSON.stringify({ status: 'empty' })).setMimeType(ContentService.MimeType.JSON);
  const config = JSON.parse(data[0][1] || '{}');
  const dates = data[1].slice(1);
  const scheduleData = {};
  for (let i = 2; i < data.length; i++) {
    const empName = data[i][0];
    if (!empName) continue;
    scheduleData[empName] = {};
    for (let j = 1; j < data[i].length; j++) {
      if (data[i][j]) scheduleData[empName][dates[j-1]] = data[i][j];
    }
  }
  return ContentService.createTextOutput(JSON.stringify({ status: 'success', isMonthLocked: config.isMonthLocked, dateSettings: config.dateSettings, scheduleData: scheduleData })).setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  try {
    const payload = JSON.parse(e.postData.contents);
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    let sheet = ss.getSheetByName(payload.monthKey);
    if (!sheet) sheet = ss.insertSheet(payload.monthKey);
    sheet.clear();
    sheet.appendRow(['[Config]', JSON.stringify({ isMonthLocked: payload.isMonthLocked, dateSettings: payload.dateSettings })]);
    sheet.appendRow(['姓名'].concat(payload.dates));
    for (const emp in payload.scheduleData) {
      const row = [emp];
      payload.dates.forEach(d => row.push(payload.scheduleData[emp][d] || ''));
      sheet.appendRow(row);
    }
    return ContentService.createTextOutput(JSON.stringify({ status: 'success' })).setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({ status: 'error', message: err.toString() })).setMimeType(ContentService.MimeType.JSON);
  }
}

點擊 部署 > 新增部署作業。類型選 網頁應用程式，執行身分選 我，誰可以存取選 所有人。複製產生的 URL。第三步：更新前端 HTML開啟 SNG_Schedule.html。找到 const API_URL = '...'; 並將您剛複製的網址貼上。📋 操作指南1. 員工畫假 (排休)進入頁面後，在上方選擇自己的姓名。在屬於自己的那一列中，點擊空白格子選擇「排休」。點擊右上方「儲存班表」上傳至雲端。2. 主管排班點擊右上角「主管後台」。輸入密碼：admin。點擊格子可選擇：早1、早2、夜1、夜2、▲ (大夜)、差 或各類假別。單日備註：點擊日期表頭，可輸入節日名稱或勾選「鎖定」禁止該日畫假。本月鎖定：排班完成後點擊「鎖定本月班表」，員工將無法再進行任何更動。3. 查看今日出勤頁面最上方「今日」橫幅會根據系統日期自動顯示當日上班、出差、休假名單。⚠️ 注意事項資料儲存：每張工作表的第一列為系統配置區，請勿手動刪除。網路延遲：儲存或載入時請稍候 1-3 秒，待右下角出現「成功」提示。瀏覽器相容性：建議使用 Google Chrome 或 Safari 以獲得最佳動畫與滾動效果。

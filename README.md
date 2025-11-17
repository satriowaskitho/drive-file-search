<img width="1029" height="943" alt="image" src="https://github.com/user-attachments/assets/c56f131f-e9fb-4b99-b7d0-22039bcde9c6" /><img width="1031" height="1030" alt="image" src="https://github.com/user-attachments/assets/ee8290a2-cc69-4b10-9b97-b5350a08fe26" />***

# Google Drive File Browser untuk Google Sites

Aplikasi web untuk menampilkan dan mencari file Google Drive (Docs, Sheets, Slides) dan Microsoft Office (DOCX, XLSX, PPTX) dengan antarmuka yang responsif dan modern.

## 📋 Daftar Isi

- [Fitur](#fitur)
- [Teknologi](#teknologi)
- [Prasyarat](#prasyarat)
- [Instalasi](#instalasi)
- [Konfigurasi](#konfigurasi)
- [Deployment](#deployment)
- [Cara Menggunakan](#cara-menggunakan)
- [Struktur File](#struktur-file)
- [Troubleshooting](#troubleshooting)
- [Lisensi](#lisensi)

## ✨ Fitur

- 🔍 Pencarian file berdasarkan nama dan konten
- 📄 Mendukung format Google Workspace (Docs, Sheets, Slides)
- 💼 Mendukung format Microsoft Office (DOCX, XLSX, PPTX, DOC, XLS, PPT)
- 📱 Desain responsif (Desktop, Tablet, Mobile)
- 🎨 Antarmuka modern dengan ikon dan badge warna
- 📊 Menampilkan informasi file (ukuran, tanggal modifikasi, pemilik)
- ⚡ Auto-load semua file saat halaman dimuat
- 🔒 Pencarian terbatas pada folder tertentu

## 🛠 Teknologi

- Google Apps Script
- HTML5
- CSS3 (Responsive Design)
- JavaScript (ES6+)
- Google Drive API

## 📦 Prasyarat

- Akun Google (Gmail)
- Akses ke Google Drive
- Google Sites (untuk embed aplikasi)
- Browser modern (Chrome, Firefox, Safari, Edge)

## 🚀 Instalasi

### Langkah 1: Siapkan Folder Google Drive

1. Buka [Google Drive](https://drive.google.com)
2. Buat folder baru atau gunakan folder yang sudah ada
3. Buka folder tersebut dan salin **Folder ID** dari URL
   ```
   https://drive.google.com/drive/folders/1ABC123xyz456DEF789
                                          └──────────┬──────────┘
                                                  Folder ID
   ```

### Langkah 2: Buat Google Apps Script Project

1. Buka [script.google.com](https://script.google.com)
<img width="671" height="355" alt="image" src="https://github.com/user-attachments/assets/914d0651-07a2-4eba-b293-70b728052e4c" />

2. Klik **New Project** (Projek Baru)
<img width="661" height="329" alt="image" src="https://github.com/user-attachments/assets/11576675-6088-4467-a52b-4e3ced2e5b86" />

3. Ubah nama project menjadi **"Drive File Browser"**
<img width="1018" height="520" alt="image" src="https://github.com/user-attachments/assets/d0144fbc-5cbc-406f-bee2-a18ba13feb80" />
<img width="1025" height="749" alt="image" src="https://github.com/user-attachments/assets/ba742bf7-4ee6-4908-817b-047e175b9e72" />


### Langkah 3: Buat File Code.gs

1. Hapus kode default yang ada
<img width="1025" height="447" alt="image" src="https://github.com/user-attachments/assets/b84bc26a-3349-4e73-9b06-ad01b0e2a303" />

2. Salin dan tempel kode berikut:

```javascript
// KONFIGURASI: Ganti dengan ID folder Google Drive Anda
const FOLDER_ID = 'PASTE_FOLDER_ID_ANDA_DISINI';

function doGet() {
  return HtmlService.createHtmlOutputFromFile('Index')
    .setTitle('Drive File Search')
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}

function getAllFiles() {
  try {
    const folder = DriveApp.getFolderById(FOLDER_ID);
    const results = [];
    
    // Define all document, spreadsheet, and presentation file types
    const mimeTypes = [
      // Google Workspace files
      MimeType.GOOGLE_DOCS,
      MimeType.GOOGLE_SHEETS,
      MimeType.GOOGLE_SLIDES,
      // Microsoft Office files (modern)
      MimeType.MICROSOFT_WORD,           // .docx
      MimeType.MICROSOFT_EXCEL,          // .xlsx
      MimeType.MICROSOFT_POWERPOINT,     // .pptx
      // Microsoft Office files (legacy)
      MimeType.MICROSOFT_WORD_LEGACY,    // .doc
      MimeType.MICROSOFT_EXCEL_LEGACY,   // .xls
      MimeType.MICROSOFT_POWERPOINT_LEGACY // .ppt
    ];
    
    // Get all files in folder
    const files = folder.getFiles();
    
    while (files.hasNext()) {
      const file = files.next();
      const mimeType = file.getMimeType();
      
      // Only include document, spreadsheet, and presentation files
      if (mimeTypes.includes(mimeType)) {
        results.push({
          name: file.getName(),
          url: file.getUrl(),
          type: mimeType,
          lastModified: file.getLastUpdated().toLocaleDateString(),
          owner: file.getOwner().getName(),
          id: file.getId(),
          size: formatFileSize(file.getSize())
        });
      }
    }
    
    // Sort by name
    results.sort((a, b) => a.name.localeCompare(b.name));
    
    return results;
  } catch (error) {
    return [{error: error.toString()}];
  }
}

function searchFiles(searchTerm) {
  try {
    if (!searchTerm || searchTerm.trim() === '') {
      return getAllFiles(); // Return all files if search is empty
    }
    
    const folder = DriveApp.getFolderById(FOLDER_ID);
    const results = [];
    
    // Define all file types we want to search
    const mimeTypes = [
      // Google Workspace
      MimeType.GOOGLE_DOCS,
      MimeType.GOOGLE_SHEETS,
      MimeType.GOOGLE_SLIDES,
      // Microsoft Office (modern)
      MimeType.MICROSOFT_WORD,
      MimeType.MICROSOFT_EXCEL,
      MimeType.MICROSOFT_POWERPOINT,
      // Microsoft Office (legacy)
      MimeType.MICROSOFT_WORD_LEGACY,
      MimeType.MICROSOFT_EXCEL_LEGACY,
      MimeType.MICROSOFT_POWERPOINT_LEGACY
    ];
    
    // Build search query
    const mimeQuery = mimeTypes.map(type => `mimeType = '${type}'`).join(' or ');
    const query = `(${mimeQuery}) and fullText contains "${searchTerm}" and "${FOLDER_ID}" in parents and trashed = false`;
    
    const files = DriveApp.searchFiles(query);
    
    while (files.hasNext()) {
      const file = files.next();
      results.push({
        name: file.getName(),
        url: file.getUrl(),
        type: file.getMimeType(),
        lastModified: file.getLastUpdated().toLocaleDateString(),
        owner: file.getOwner().getName(),
        id: file.getId(),
        size: formatFileSize(file.getSize())
      });
    }
    
    // Sort by name
    results.sort((a, b) => a.name.localeCompare(b.name));
    
    return results;
  } catch (error) {
    return [{error: error.toString()}];
  }
}

function formatFileSize(bytes) {
  if (bytes === 0) return '0 Bytes';
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return Math.round(bytes / Math.pow(k, i) * 100) / 100 + ' ' + sizes[i];
}

function getFileTypeLabel(mimeType) {
  // MIME type strings from Google Apps Script
  const types = {
    // Google Workspace
    'application/vnd.google-apps.document': 'Google Docs',
    'application/vnd.google-apps.spreadsheet': 'Google Sheets',
    'application/vnd.google-apps.presentation': 'Google Slides',
    // Microsoft Word
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document': 'Word (.docx)',
    'application/msword': 'Word (.doc)',
    // Microsoft Excel
    'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet': 'Excel (.xlsx)',
    'application/vnd.ms-excel': 'Excel (.xls)',
    // Microsoft PowerPoint
    'application/vnd.openxmlformats-officedocument.presentationml.presentation': 'PowerPoint (.pptx)',
    'application/vnd.ms-powerpoint': 'PowerPoint (.ppt)'
  };
  
  return types[mimeType] || 'Unknown';
}
```
<img width="1024" height="944" alt="image" src="https://github.com/user-attachments/assets/fafb9520-d386-45e7-abd9-f79c64356cdf" />

3. **Penting**: Ganti `PASTE_FOLDER_ID_ANDA_DISINI` dengan Folder ID yang Anda salin di Langkah 1


### Langkah 4: Buat File Index.html

1. Klik **+** di sebelah "Files"
<img width="1019" height="558" alt="image" src="https://github.com/user-attachments/assets/741570b2-5f0e-4ea9-aa7a-b4365492b3bd" />

2. Pilih **HTML**
<img width="1023" height="593" alt="image" src="https://github.com/user-attachments/assets/9204834b-f1a7-42ed-8a61-5232816e1ea7" />

3. Beri nama **Index**
<img width="1025" height="522" alt="image" src="https://github.com/user-attachments/assets/bd300332-793e-4e4d-9170-ee146c52809e" />

4. Pilih file **Index.html**, salin dan tempel kode HTML di bawah:
<img width="1028" height="490" alt="image" src="https://github.com/user-attachments/assets/860eae75-3129-4a44-910e-534bba881ec2" />


```javascript
<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    * {
      box-sizing: border-box;
    }
    
    body {
      font-family: 'Google Sans', Arial, sans-serif;
      padding: 20px;
      max-width: 900px;
      margin: 0 auto;
      /* background-color: #f8f9fa; */
      background-color: #fff;
    }
    
    .header {
      background-color: #fff;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.1);
      margin-bottom: 20px;
    }
    
    h2 {
      margin: 0 0 15px 0;
      color: #202124;
      font-size: 24px;
    }
    
    .search-container {
      display: flex;
      gap: 10px;
      align-items: center;
      flex-wrap: wrap;
    }
    
    #searchInput {
      flex: 1;
      min-width: 200px;
      padding: 12px 16px;
      font-size: 16px;
      border: 1px solid #dadce0;
      border-radius: 24px;
      outline: none;
      transition: box-shadow 0.2s;
    }
    
    #searchInput:focus {
      box-shadow: 0 1px 6px rgba(32,33,36,.28);
      border-color: #4285f4;
    }
    
    #searchBtn, #clearBtn {
      padding: 12px 24px;
      font-size: 16px;
      border: none;
      border-radius: 24px;
      cursor: pointer;
      font-weight: 500;
      transition: background-color 0.2s;
      white-space: nowrap;
    }
    
    #searchBtn {
      background-color: #1a73e8;
      color: white;
    }
    
    #searchBtn:hover {
      background-color: #1765cc;
    }
    
    #clearBtn {
      background-color: #fff;
      color: #5f6368;
      border: 1px solid #dadce0;
    }
    
    #clearBtn:hover {
      background-color: #f8f9fa;
    }
    
    .loading {
      display: none;
      color: #5f6368;
      margin: 10px 0;
      text-align: center;
      padding: 20px;
      background-color: #fff;
      border-radius: 8px;
    }
    
    .stats {
      background-color: #fff;
      padding: 15px 20px;
      border-radius: 8px;
      margin-bottom: 15px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.1);
      color: #5f6368;
      font-size: 14px;
    }
    
    .results {
      margin-top: 20px;
    }
    
    .file-item {
      background-color: #fff;
      border: 1px solid #dadce0;
      padding: 16px 20px;
      margin-bottom: 12px;
      border-radius: 8px;
      transition: box-shadow 0.2s;
      cursor: pointer;
    }
    
    .file-item:hover {
      box-shadow: 0 2px 8px rgba(0,0,0,0.15);
    }
    
    .file-header {
      display: flex;
      align-items: flex-start;
      gap: 12px;
    }
    
    .file-icon {
      font-size: 28px;
      flex-shrink: 0;
      line-height: 1;
    }
    
    .file-content {
      flex: 1;
      min-width: 0;
    }
    
    .file-name {
      font-weight: 500;
      color: #1a73e8;
      text-decoration: none;
      font-size: 16px;
      display: block;
      margin-bottom: 8px;
      word-wrap: break-word;
      line-height: 1.4;
    }
    
    .file-name:hover {
      text-decoration: underline;
    }
    
    .file-info {
      color: #5f6368;
      font-size: 13px;
      display: flex;
      gap: 12px;
      flex-wrap: wrap;
      align-items: center;
    }
    
    .file-type {
      display: inline-block;
      padding: 4px 12px;
      background-color: #e8f0fe;
      color: #1967d2;
      border-radius: 12px;
      font-size: 12px;
      font-weight: 500;
    }
    
    /* Google Workspace */
    .file-type.docs {
      background-color: #e6f4ea;
      color: #137333;
    }
    
    .file-type.sheets {
      background-color: #e8f5e9;
      color: #0d652d;
    }
    
    .file-type.slides {
      background-color: #fef7e0;
      color: #f9ab00;
    }
    
    /* Microsoft Office */
    .file-type.word {
      background-color: #deebff;
      color: #0052cc;
    }
    
    .file-type.excel {
      background-color: #e3fcef;
      color: #006644;
    }
    
    .file-type.powerpoint {
      background-color: #ffebe6;
      color: #bf2600;
    }
    
    .no-results {
      color: #5f6368;
      text-align: center;
      padding: 40px 20px;
      background-color: #fff;
      border-radius: 8px;
    }
    
    .error {
      color: #d93025;
      padding: 16px 20px;
      background-color: #fce8e6;
      border-radius: 8px;
      border: 1px solid #d93025;
    }
    
    /* Tablet (768px - 1024px) */
    @media (max-width: 1024px) and (min-width: 768px) {
      body {
        padding: 15px;
      }
      
      h2 {
        font-size: 22px;
      }
      
      .header {
        padding: 16px;
      }
      
      .file-item {
        padding: 14px 16px;
      }
      
      #searchInput {
        font-size: 15px;
        padding: 11px 14px;
      }
      
      #searchBtn, #clearBtn {
        padding: 11px 20px;
        font-size: 15px;
      }
    }
    
    /* Mobile (up to 767px) */
    @media (max-width: 767px) {
      body {
        padding: 10px;
      }
      
      h2 {
        font-size: 20px;
        margin-bottom: 12px;
      }
      
      .header {
        padding: 15px;
      }
      
      .search-container {
        flex-direction: column;
        gap: 8px;
        width: 100%;
      }
      
      #searchInput {
        width: 100%;
        min-width: unset;
        font-size: 16px;
        padding: 12px 16px;
      }
      
      #searchBtn, #clearBtn {
        width: 100%;
        padding: 12px 20px;
        font-size: 15px;
      }
      
      .stats {
        padding: 12px 15px;
        font-size: 13px;
      }
      
      .file-item {
        padding: 12px 15px;
        margin-bottom: 10px;
      }
      
      .file-header {
        gap: 10px;
      }
      
      .file-icon {
        font-size: 24px;
      }
      
      .file-name {
        font-size: 15px;
        margin-bottom: 6px;
      }
      
      .file-info {
        font-size: 12px;
        gap: 8px;
      }
      
      .file-type {
        font-size: 11px;
        padding: 3px 10px;
      }
    }
    
    /* Small Mobile (up to 480px) */
    @media (max-width: 480px) {
      body {
        padding: 8px;
      }
      
      h2 {
        font-size: 18px;
      }
      
      .header {
        padding: 12px;
      }
      
      .file-item {
        padding: 10px 12px;
      }
      
      .file-name {
        font-size: 14px;
      }
      
      .file-info {
        font-size: 11px;
      }
      
      .no-results {
        padding: 30px 15px;
        font-size: 14px;
      }
    }
    
    /* Landscape orientation on mobile */
    @media (max-width: 767px) and (orientation: landscape) {
      .search-container {
        flex-direction: row;
      }
      
      #searchBtn, #clearBtn {
        width: auto;
      }
      
      #searchInput {
        flex: 1;
      }
    }
  </style>
</head>
<body>
  <div class="header">
    <h2>📁 Drive File Browser</h2>
    <div class="search-container">
      <input type="text" id="searchInput" placeholder="Search files...">
      <button id="searchBtn" onclick="performSearch()">Search</button>
      <button id="clearBtn" onclick="showAllFiles()">Show All</button>
    </div>
    <div class="loading" id="loading">⏳ Loading files...</div>
  </div>
  
  <div id="stats" class="stats" style="display:none;"></div>
  <div id="results" class="results"></div>

  <script>
    // Load all files on page load
    window.onload = function() {
      showAllFiles();
    };

    document.getElementById('searchInput').addEventListener('keypress', function(e) {
      if (e.key === 'Enter') {
        performSearch();
      }
    });

    function showAllFiles() {
      document.getElementById('searchInput').value = '';
      document.getElementById('loading').style.display = 'block';
      document.getElementById('results').innerHTML = '';
      document.getElementById('stats').style.display = 'none';

      google.script.run
        .withSuccessHandler(displayResults)
        .withFailureHandler(displayError)
        .getAllFiles();
    }

    function performSearch() {
      const searchTerm = document.getElementById('searchInput').value;

      document.getElementById('loading').style.display = 'block';
      document.getElementById('results').innerHTML = '';
      document.getElementById('stats').style.display = 'none';

      google.script.run
        .withSuccessHandler(displayResults)
        .withFailureHandler(displayError)
        .searchFiles(searchTerm);
    }

    function displayResults(results) {
      document.getElementById('loading').style.display = 'none';
      const resultsDiv = document.getElementById('results');
      const statsDiv = document.getElementById('stats');

      if (results.length === 0) {
        resultsDiv.innerHTML = '<div class="no-results">📭 No files found</div>';
        return;
      }

      if (results[0].error) {
        resultsDiv.innerHTML = `<div class="error">❌ Error: ${results[0].error}</div>`;
        return;
      }

      // Show statistics
      const searchTerm = document.getElementById('searchInput').value;
      const statsText = searchTerm 
        ? `Found ${results.length} file(s) matching "${searchTerm}"`
        : `Showing all ${results.length} file(s)`;
      statsDiv.innerHTML = statsText;
      statsDiv.style.display = 'block';

      // Display files
      let html = '';
      
      results.forEach(file => {
        const fileIcon = getFileTypeIcon(file.type);
        const fileTypeLabel = getFileTypeLabel(file.type);
        const typeClass = getFileTypeClass(file.type);
        
        html += `
          <div class="file-item" onclick="window.open('${file.url}', '_blank')">
            <div class="file-header">
              <div class="file-icon">${fileIcon}</div>
              <div class="file-content">
                <a href="${file.url}" target="_blank" class="file-name" onclick="event.stopPropagation()">
                  ${escapeHtml(file.name)}
                </a>
                <div class="file-info">
                  <span class="file-type ${typeClass}">${fileTypeLabel}</span>
                  <span>📏 ${file.size}</span>
                  <span>📅 ${file.lastModified}</span>
                  <span>👤 ${escapeHtml(file.owner)}</span>
                </div>
              </div>
            </div>
          </div>
        `;
      });

      resultsDiv.innerHTML = html;
    }

    function displayError(error) {
      document.getElementById('loading').style.display = 'none';
      document.getElementById('results').innerHTML = 
        `<div class="error">❌ Error: ${error.message}</div>`;
    }

    function getFileTypeIcon(mimeType) {
      if (mimeType.includes('google-apps.document')) return '📄';
      if (mimeType.includes('google-apps.spreadsheet')) return '📊';
      if (mimeType.includes('google-apps.presentation')) return '📽️';
      if (mimeType.includes('wordprocessing') || mimeType.includes('msword')) return '📘';
      if (mimeType.includes('spreadsheetml') || mimeType.includes('ms-excel')) return '📗';
      if (mimeType.includes('presentationml') || mimeType.includes('ms-powerpoint')) return '📙';
      return '📎';
    }

    function getFileTypeLabel(mimeType) {
      if (mimeType.includes('google-apps.document')) return 'Google Docs';
      if (mimeType.includes('google-apps.spreadsheet')) return 'Google Sheets';
      if (mimeType.includes('google-apps.presentation')) return 'Google Slides';
      if (mimeType.includes('wordprocessingml.document')) return 'Word (.docx)';
      if (mimeType === 'application/msword') return 'Word (.doc)';
      if (mimeType.includes('spreadsheetml.sheet')) return 'Excel (.xlsx)';
      if (mimeType === 'application/vnd.ms-excel') return 'Excel (.xls)';
      if (mimeType.includes('presentationml.presentation')) return 'PowerPoint (.pptx)';
      if (mimeType === 'application/vnd.ms-powerpoint') return 'PowerPoint (.ppt)';
      return 'File';
    }

    function getFileTypeClass(mimeType) {
      if (mimeType.includes('google-apps.document')) return 'docs';
      if (mimeType.includes('google-apps.spreadsheet')) return 'sheets';
      if (mimeType.includes('google-apps.presentation')) return 'slides';
      if (mimeType.includes('wordprocessing') || mimeType.includes('msword')) return 'word';
      if (mimeType.includes('spreadsheetml') || mimeType.includes('ms-excel')) return 'excel';
      if (mimeType.includes('presentationml') || mimeType.includes('ms-powerpoint')) return 'powerpoint';
      return '';
    }

    function escapeHtml(text) {
      const div = document.createElement('div');
      div.textContent = text;
      return div.innerHTML;
    }
  </script>
</body>
</html>
```
<img width="1028" height="1028" alt="image" src="https://github.com/user-attachments/assets/05a1bdbe-1df3-4d0b-8afd-0d779e88a106" />


## ⚙️ Konfigurasi

### Mengubah Folder yang Dicari

Edit file `Code.gs` pada baris:

```javascript
const FOLDER_ID = 'ID_FOLDER_ANDA';
```
<img width="1027" height="415" alt="image" src="https://github.com/user-attachments/assets/33b7d928-5569-4fbc-a3b0-f24412c89407" />


Ganti dengan ID folder yang ingin Anda tampilkan.

## 🚢 Deployment

### Deploy sebagai Web App

1. Di Apps Script editor, klik **Deploy** > **New deployment**
<img width="1027" height="452" alt="image" src="https://github.com/user-attachments/assets/b3ece0d8-b017-4f2d-a4ce-c1e2dcaf7c52" />

2. Klik ikon ⚙️ > Pilih **Web app**
<img width="1027" height="1031" alt="image" src="https://github.com/user-attachments/assets/f97f022d-938c-42e9-aa52-d060eed9fc7f" />
<img width="1031" height="1030" alt="image" src="https://github.com/user-attachments/assets/510de29d-4957-4cd6-be71-c1b8d3b2cf6d" />

3. Isi konfigurasi:
   - **Description**: Drive File Browser v1.0
   - **Execute as**: Me (contoh: email@gmail.com)
   - **Who has access**: Anyone (atau pilih sesuai kebutuhan)
<img width="1029" height="1029" alt="image" src="https://github.com/user-attachments/assets/d6a48192-7bbc-4912-8094-9763845d4a82" />

4. Klik **Deploy**
<img width="1030" height="1028" alt="image" src="https://github.com/user-attachments/assets/0748dba2-71dd-443f-86c6-6b6e77119976" />

5. Klik **Authorize access**
<img width="1033" height="1030" alt="image" src="https://github.com/user-attachments/assets/979b433e-d804-49ef-a462-481e3f433fc9" />

6. Pilih akun Google Anda
<img width="656" height="796" alt="image" src="https://github.com/user-attachments/assets/c06d3116-4629-4c86-8da8-22ea2b4f18a0" />

7. Klik **Advanced** > **Go to [Project Name] (unsafe)**
<img width="660" height="800" alt="image" src="https://github.com/user-attachments/assets/ab29de12-0fbe-4ffd-af27-bfd8aea30a85" />
<img width="663" height="804" alt="image" src="https://github.com/user-attachments/assets/e6ba2ae0-c37f-48fc-b303-aa269497dce3" />

8. Klik **Continue**
<img width="659" height="946" alt="image" src="https://github.com/user-attachments/assets/6aebdf97-03a1-48a5-bdb0-6aa98e47c407" />

9. **Salin URL Web App** yang muncul
<img width="1029" height="943" alt="image" src="https://github.com/user-attachments/assets/71ff1f36-2c77-4fcb-bf37-f90f07e5768f" />


### Embed di Google Sites

1. Buka [sites.google.com](https://sites.google.com)
<img width="1022" height="364" alt="image" src="https://github.com/user-attachments/assets/686079e1-371c-43c4-b31a-6c0732cded2d" />

2. Pilih atau buat Google Site baru
3. Klik **Edit** (ikon pensil)
4. Klik di area halaman tempat Anda ingin menambahkan browser file
5. Di panel kanan, klik **Insert** > **Embed**
<img width="1028" height="1028" alt="image" src="https://github.com/user-attachments/assets/bbf2a3b8-32d3-472f-9fbf-ec787460068d" />

6. Pilih tab **By URL**
<img width="1027" height="942" alt="image" src="https://github.com/user-attachments/assets/bd36e5a8-a1f5-4d53-9100-4409311ca81d" />

7. **Paste URL Web App** yang Anda salin
<img width="1026" height="941" alt="image" src="https://github.com/user-attachments/assets/a296bef2-280b-460b-94da-8b4018903f50" />

8. Klik **Insert**
9. Sesuaikan ukuran embed
<img width="1025" height="940" alt="image" src="https://github.com/user-attachments/assets/ea7859af-1f27-40ba-9026-2a86f2491fb0" />

10. Klik **Publish** untuk mempublikasikan site

## 📖 Cara Menggunakan

### Melihat Semua File

- Saat halaman dimuat, semua file akan ditampilkan secara otomatis
- File diurutkan berdasarkan nama (A-Z)

### Mencari File

1. Ketik kata kunci di kotak pencarian
2. Tekan **Enter** atau klik tombol **Search**
3. Hasil akan menampilkan file yang mengandung kata kunci di:
   - Nama file
   - Konten file (untuk Google Docs, Sheets, Slides)

### Membuka File

- Klik pada nama file atau di mana saja pada card file
- File akan terbuka di tab baru

### Kembali Melihat Semua File

- Klik tombol **Show All**
- Atau hapus teks pencarian dan tekan Enter

## 📁 Struktur File

```
drive-file-browser/
├── Code.gs           # Server-side logic (Apps Script)
└── Index.html        # Frontend UI (HTML + CSS + JS)
```

## 🐛 Troubleshooting

### Error: "Exception: You do not have permission to call DriveApp.getFolderById"

**Solusi**: 
1. Pastikan Anda sudah memberikan izin saat deployment
2. Re-deploy aplikasi dan authorize ulang

### File Tidak Muncul

**Penyebab**:
- Folder ID salah
- File bukan tipe yang didukung
- File ada di subfolder (tidak terdeteksi)

**Solusi**:
1. Periksa kembali Folder ID di `Code.gs`
2. Pastikan file adalah Google Docs/Sheets/Slides atau MS Office
3. Pindahkan file ke folder utama (bukan subfolder)

### Embed Tidak Muncul di Google Sites

**Solusi**:
1. Pastikan deployment setting "Who has access" adalah **Anyone** atau sesuai organisasi Anda
2. Gunakan URL yang benar (harus diakhiri dengan `/exec`)
3. Clear cache browser dan refresh halaman

### Pencarian Tidak Menemukan File

**Catatan**: 
- Pencarian full-text hanya bekerja untuk Google Docs, Sheets, dan Slides
- File Microsoft Office hanya bisa dicari berdasarkan nama file, bukan konten

## 📱 Responsive Breakpoints

| Device | Screen Width | Layout |
|--------|--------------|--------|
| Desktop | > 1024px | Full layout dengan tombol horizontal |
| Tablet | 768px - 1024px | Layout sedang dengan spacing dikurangi |
| Mobile | < 768px | Tombol stack vertikal, full width |
| Small Mobile | < 480px | Text lebih kecil, padding minimal |

## 👨‍💻 Author

Dibuat dengan ❤️ untuk komunitas developer Indonesia

## 🙏 Acknowledgments

- Google Apps Script Documentation
- Google Drive API
- Material Design Guidelines

***

**Catatan**: Pastikan untuk mengganti placeholder seperti `FOLDER_ID` dan informasi lainnya sesuai dengan kebutuhan Anda sebelum deployment.[2][4][1]

[1](https://dev.to/mochafreddo/how-to-manage-documentation-in-a-github-repository-a-guide-for-junior-developers-pgo)
[2](https://www.freecodecamp.org/news/how-to-structure-your-readme-file/)
[3](https://gitprotect.io/blog/how-to-put-a-project-on-github-best-practices/)
[4](https://www.drupal.org/docs/develop/managing-a-drupalorg-theme-module-or-distribution-project/documenting-your-project/readmemd-template)
[5](https://codepolitan.com/blog/cara-membuat-profile-github-simple-dan-menarik)
[6](https://www.youtube.com/watch?v=Z_Q9kzPUhVg)
[7](https://www.instagram.com/reel/DDWQcD_TO93/)
[8](https://id.scribd.com/doc/262867351/Github-Bahasa-Indonesia-tutorial)
[9](https://pbp-fasilkom-ui.github.io/ganjil-2023/assignments/tutorial/tutorial-0/)
[10](https://docs.aws.amazon.com/id_id/codedeploy/latest/userguide/tutorials-github-create-github-repository.html)

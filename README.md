🛠️ 第一階段：軟體環境與工具下載 (The Infrastructure)
在建立專案之初，我們下載並部署了以下關鍵組件，這些組件構成了資安監控與 AI 分析的物理環境：
組件名稱,用途說明,關鍵操作
Wazuh 4.13.1,資安監控核心，負責日誌收集、索引與告警。,使用 wazuh-install.sh 進行全自動部署。
Node.js & npm,執行 MCP (Model Context Protocol) 伺服器的核心環境。,用於安裝與加載 mcp-server-wazuh 插件。
Claude Desktop,AI 威脅獵捕助理的互動介面，負責執行 MCP 指令。,在 Windows 端安裝，並加載自定義 JSON 配置。
Python 3 & Flask,建立模擬環境與自動化腳本的程式語言與框架。,執行 mock_wazuh_api.py 模擬 55000 埠口的 API 回應。
OpenJDK (JDK),Wazuh 內建的 Java 環境，用於執行安全管理工具。,手動設定 JAVA_HOME 指向 /usr/share/wazuh-indexer/jdk。


📝 第二階段：手動修改的設定檔 (Nano Modifications)
這部分是專案中最核心的「底層手術」，我們使用 nano 編輯器修改了以下檔案，以確保 401 Unauthorized 錯誤被排除：

1. 內部使用者資料庫 (internal_users.yml)
路徑：/etc/wazuh-indexer/opensearch-security/internal_users.yml。

修改內容：手動更換 admin 帳號的密碼雜湊值（Hash）。

技術意義：這是修復 API 與 Indexer 認證不一致的關鍵點。

2. 網頁端 API 連線配置 (wazuh.yml)
路徑：/usr/share/wazuh-dashboard/data/wazuh/config/wazuh.yml。

修改內容：更新 password 欄位為黃金密碼 59.SOfBL...。

技術意義：讓 Dashboard 網頁介面能從 Offline 變回 Online。

3. Claude MCP 連線設定 (claude_desktop_config.json)
路徑：Windows 電腦 %APPDATA%\Claude\。

修改內容：定義 WAZUH_API_HOST (192.168.1.51) 與認證憑證。

技術意義：打通 AI 助理與實體資安伺服器之間的「通訊隧道」。

⚙️ 第三階段：管理指令與修復工具 (Administrative Tools)
除了手動修改檔案，我們還執行了多個關鍵指令來「同步」整個系統：

securityadmin.sh：最關鍵的指令。我們使用此工具將 internal_users.yml 的變更強制刷入資料庫安全索引，這是讓所有認證生效的唯一方法。

wazuh-passwords-tool.sh：我們下載並執行此工具，嘗試統一所有組件的 admin 密碼，降低維護複雜度。

systemctl restart：頻繁使用此指令重啟 wazuh-manager 與 wazuh-dashboard，以刷新快取並載入新設定。

bcrypt 雜湊產生：利用 Python 的 bcrypt 模組產生符合 Wazuh 安全規範的雜湊值，取代報錯的 hash.sh。<img width="1233" height="978" alt="螢幕擷取畫面 2026-01-11 212749" src="https://github.com/user-attachments/assets/901b541f-9b66-40ca-8509-4e33a32933b9" />


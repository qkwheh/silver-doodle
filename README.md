# silver-doodle
[vmfha.html](https://github.com/user-attachments/files/24095921/vmfha.html)
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Promptly Interface Enhanced Dark Mode (Token System)</title>
<style>
/* ==================================================
1. ROOT Variables & Base Styles
================================================== 
*/
:root {
  /* Dimensions */
  --sidebar-width: 280px;
  --header-height: 70px;
  --input-area-height: 100px;
  --footer-height: 80px;
  
  /* Dark Mode Variables (Default) - Deep Purple Accent */
  --bg-color: #0d0d0d; 
  --surface-color: #1a1a1a; 
  --text-color: #e0e0e0; 
  --subtle-text: #a0a0a0; 
  --accent-color: #5E3BA9; /* Deep Purple */
  --highlight-color: #4A2E80; 
  --input-border: #333;
  --confirm-btn-bg: #42a5f5; 
  --hover-bg: #2a2a2a; 
  --code-bg: #111; 
  --code-text: #f8f8f2; 
  --shadow-color: rgba(0, 0, 0, 0.5);
  
  /* [추가] 로고 발광 효과를 위한 색상 변수 */
  --logo-glow-color: rgba(94, 59, 169, 0.8);

  /* [수정] 울트라 마린 색상 변수 추가 */
  --ultramarine-fill: #1F45FC; /* Rich Ultramarine Tone */
}

/* Light Mode Switch */
body.light-mode {
  --bg-color: #ffffff;
  --surface-color: #f0f0f0;
  --text-color: #1a1a1a;
  --subtle-text: #606060;
  --accent-color: #2745E8; 
  --highlight-color: #E0E0FF; 
  --input-border: #ccc;
  --confirm-btn-bg: #1565c0;
  --hover-bg: #e5e5e5;
  --code-bg: #f8f8f8;
  --code-text: #333;
  --shadow-color: rgba(0, 0, 0, 0.1);
  --logo-glow-color: rgba(39, 69, 232, 0.6); /* Light Mode Glow */
  --ultramarine-fill: #002fa7; /* Light Mode Ultramarine Tone */
}

/* [수정] 전체 레이아웃 스크롤 방지 */
html, body {
    height: 100%; /* 전체 높이 사용 */
    overflow: hidden; /* 브라우저 스크롤 숨김 */
}

body {
  font-family: -apple-system, BlinkMacMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  margin: 0;
  padding: 0;
  background-color: var(--bg-color);
  color: var(--text-color);
  /* [수정] min-height 대신 고정 높이 사용 */
  height: 100vh; 
  max-height: 100vh;
  display: flex;
  flex-direction: column;
  transition: background-color 0.3s, color 0.3s;
}

/* ==================================================
2. Layout Structure (Body, Container)
================================================== 
*/
#chatContainer {
  /* [수정] flex-grow 제거 및 Absolute Positioning으로 고정 영역 사이 채우기 */
  position: absolute;
  top: var(--header-height); /* 헤더 아래에서 시작 */
  bottom: calc(var(--input-area-height) + var(--footer-height)); /* 입력창과 푸터 위에서 끝 */
  left: 0;
  right: 0;
  width: 100%;
  
  /* [추가] 이 영역에서만 스크롤이 발생하도록 설정 */
  overflow-y: auto; 

  /* 기타 기존 스타일 유지 */
  display: flex;
  flex-direction: column;
  align-items: center;
  transition: all 0.3s cubic-bezier(0.4, 0.0, 0.2, 1);
}
#chatContainer.sidebar-open {
  /* Sidebar가 열릴 때 위치 조정 */
  left: var(--sidebar-width);
  width: calc(100% - var(--sidebar-width));
}

/* Drag and Drop Visual Feedback */
#chatContainer.dragging {
    border: 3px dashed var(--accent-color);
    background-color: var(--hover-bg);
}

/* ==================================================
[신규 기능] 채팅 시작 전 입력창 중앙 배치
================================================== */

/* 1. Base for Centering */
body.initial-state {
    /* min-height: 100vh; 는 html, body에 의해 대체됨 */
}

/* 2. Chat Area Visibility */
body.initial-state #chatArea {
    /* 채팅 영역 내용 숨기기 */
    display: none !important;
}

/* 3. Input Bar in Centered State */
body.initial-state .input-bar-area {
    /* 고정(Fixed) 위치 재정의 */
    position: fixed; 
    top: 50%;
    bottom: auto; 
    left: 50%; 
    right: auto;
    
    /* 중앙 정렬 Transform */
    transform: translate(-50%, -50%); 
    
    /* 외관 변경: 하단 바처럼 보이지 않게 */
    width: 90%; 
    max-width: 800px;
    padding: 0; /* 수직 패딩 제거 */
    background-color: var(--bg-color); /* 배경색 일치 */
    border-top: none;
    box-shadow: none;

    /* 부드러운 전환 */
    transition: all 0.5s ease-in-out;
}

/* 4. Sidebar Handling in Centered State */
body.initial-state .input-bar-area.sidebar-open-fixed {
    /* Sidebar를 고려하여 중앙 배치 (남은 영역의 중앙) */
    left: calc(var(--sidebar-width) + (100% - var(--sidebar-width)) / 2);
    /* Transform은 그대로 유지 */
    transform: translate(-50%, -50%); 
    width: 90%;
}


/* ==================================================
3. Sidebar (History)
================================================== 
*/
#historySidebar {
  position: fixed;
  left: calc(-1 * var(--sidebar-width));
  top: 0;
  height: 100%;
  width: var(--sidebar-width);
  background-color: var(--surface-color);
  border-right: 1px solid var(--input-border);
  transition: left 0.3s cubic-bezier(0.4, 0.0, 0.2, 1);
  padding: 20px;
  z-index: 20;
  box-sizing: border-box;
  overflow-y: auto;
  box-shadow: 2px 0 5px var(--shadow-color);
}
#historySidebar.open {
  left: 0;
}
/* ... (Sidebar 내부 스타일 생략) ... */
.sidebar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  flex-wrap: wrap; /* 내용이 넘치지 않도록 */
}
.sidebar-header p {
  width: 100%; /* 설명 텍스트가 다음 줄로 이동 */
  margin-top: 5px;
}
.close-sidebar-btn {
  background: none;
  border: none;
  color: var(--text-color);
  font-size: 28px;
  cursor: pointer;
  line-height: 1;
  padding: 0;
  opacity: 0.7;
  transition: opacity 0.2s;
}
.close-sidebar-btn:hover {
  opacity: 1;
}
.sidebar-toggle {
  cursor: pointer;
  font-size: 24px;
  color: var(--text-color);
  margin-right: 10px;
  opacity: 0.8;
  transition: opacity 0.2s;
}
.sidebar-toggle:hover {
  opacity: 1;
}
#historyList p {
  position: relative;
  padding: 10px 12px;
  padding-right: 60px;
  /* [수정] 항목 간격 축소 및 레이아웃 정리 */
  margin: 5px 0; 
  border-radius: 8px;
  cursor: pointer;
  font-size: 13px;
  color: var(--text-color);
  /* [수정] 배경색을 투명하게 하여 사이드바 배경과 일체감 부여 */
  background-color: transparent; 
  transition: background-color 0.2s, color 0.2s;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  border-left: 3px solid transparent;
}
#historyList p:hover {
  /* 호버 시에만 배경색을 넣어 강조 */
  background-color: var(--hover-bg); 
  border-left: 3px solid var(--accent-color);
}
.history-actions {
    position: absolute;
    right: 5px;
    top: 50%;
    transform: translateY(-50%);
    display: flex;
    gap: 5px;
    opacity: 0;
    transition: opacity 0.2s;
}
#historyList p:hover .history-actions {
    opacity: 1;
}
.history-actions button {
    background: none;
    border: none;
    color: var(--subtle-text);
    cursor: pointer;
    font-size: 12px;
    padding: 3px;
    line-height: 1;
    transition: color 0.2s;
}
.history-actions button:hover {
    color: var(--accent-color);
}
.history-actions .pin-btn.pinned {
    color: var(--confirm-btn-bg);
}

/* ==================================================
4. Header & Settings
================================================== 
*/
.header-container {
  /* [수정] position: sticky -> position: fixed */
  position: fixed; 
  top: 0; 
  left: 0;
  right: 0;
  width: 100%;
  
  /* [추가] 높이 명시 및 기존 스타일 유지 */
  height: var(--header-height);
  display: flex;
  justify-content: center;
  padding: 10px 0; /* 기존 패딩 유지 */
  border-bottom: 1px solid var(--input-border);
  background-color: var(--surface-color);
  box-shadow: 0 2px 5px var(--shadow-color);
  z-index: 15;
  box-sizing: border-box;
  transition: background-color 0.3s, border-color 0.3s, left 0.3s;
}
/* [추가] Sidebar 열릴 때 헤더 위치 조정 */
.header-container.sidebar-open-fixed {
  left: var(--sidebar-width);
  width: calc(100% - var(--sidebar-width));
}

.header-content {
  width: 90%;
  max-width: 1200px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
/* ... (Header 내부 스타일 생략) ... */
.logo-group {
  display: flex; align-items: center; gap: 15px;
}
/* ----- 사용자 로고 SVG 스타일 추가/수정 (헤더용) ----- */

/* [추가] 로고 발광 애니메이션 */
@keyframes glow-pulse {
    0% { box-shadow: 0 0 5px var(--logo-glow-color), 0 0 10px var(--logo-glow-color); }
    50% { box-shadow: 0 0 10px var(--logo-glow-color), 0 0 20px var(--logo-glow-color); }
    100% { box-shadow: 0 0 5px var(--logo-glow-color), 0 0 10px var(--logo-glow-color); }
}

.symbol-svg {
  width: 30px; /* Adjusted to fit header */
  height: 30px; /* Adjusted to fit header */
  cursor: pointer;
  /* User's 3D/tilt effect */
  transform: perspective(400px) rotateY(-8deg) rotateX(10deg);
  /* [수정] 부드러운 전환 효과 추가 */
  transition: transform 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
}

/* [추가/수정] 호버 애니메이션 */
.symbol-svg:hover {
    /* 크기 미약하게 커짐 */
    transform: scale(1.05) perspective(400px) rotateY(-8deg) rotateX(10deg);
    /* 발광 효과 애니메이션 시작 */
    animation: glow-pulse 1.5s infinite alternate;
}

/* [수정] 로고 심볼 색상 및 두께 설정 */
.logo-group .symbol-svg .left-half, 
.logo-group .symbol-svg .right-half {
  /* fill: none; <-- 기존 코드 */
  fill: var(--ultramarine-fill); /* [수정] 울트라 마린 색상으로 채우기 */
  stroke: var(--accent-color); /* 외곽선 색상을 강조색으로 */
  stroke-width: 5; /* [수정] 외곽선 두께 설정 */
  transition: stroke 0.3s, fill 0.3s;
}

.logo-text {
  font-size: 26px; font-weight: 800; color: var(--accent-color);
  cursor: pointer;
}
.user-profile {
  display: flex; align-items: center; gap: 20px; position: relative;
}
.token-display {
  font-size: 14px; color: var(--subtle-text);
}
.profile-icon {
  width: 35px;
  height: 35px;
  border-radius: 50%;
  background-color: var(--highlight-color);
  color: var(--text-color);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  border: 2px solid var(--accent-color);
  z-index: 10;
}
#settingsDropdown {
  display: none;
  position: absolute;
  top: 50px;
  right: 0;
  background-color: var(--surface-color);
  border: 1px solid var(--input-border);
  border-radius: 8px;
  box-shadow: 0 4px 12px var(--shadow-color);
  min-width: 300px;
  z-index: 20;
  overflow: hidden;
  padding: 10px 0;
}
.setting-group {
    padding: 10px 18px;
    border-top: 1px solid var(--input-border);
    margin-top: 5px;
}
.setting-group:first-child {
    border-top: none;
}
.setting-group h4 {
    margin: 0 0 10px 0;
    color: var(--accent-color);
    font-size: 14px;
}
.setting-group textarea, .setting-group input {
    width: 100%;
    box-sizing: border-box;
    padding: 8px;
    margin-bottom: 5px;
    border: 1px solid var(--input-border);
    border-radius: 4px;
    background-color: var(--bg-color);
    color: var(--text-color);
    resize: vertical;
    font-size: 13px;
}
.setting-item {
  padding: 12px 18px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.2s;
  display: flex;
  align-items: center;
  gap: 10px;
}
.setting-item:hover {
  background-color: var(--hover-bg);
  color: var(--accent-color);
}
#darkModeToggle {
  background: none;
  color: var(--subtle-text);
  border: none;
  padding: 8px 10px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: color 0.2s, background-color 0.2s;
}
#darkModeToggle:hover {
  color: var(--text-color);
  background-color: var(--hover-bg);
}

/* Model Selector */
#modelSelector {
  padding: 8px 12px;
  background-color: var(--bg-color);
  border: 1px solid var(--input-border);
  color: var(--text-color);
  border-radius: 8px;
  font-size: 14px;
  cursor: pointer;
  appearance: none;
  transition: border-color 0.3s;
}
#modelSelector:hover,
#modelSelector:focus {
  border-color: var(--accent-color);
  outline: none;
}

/* ==================================================
5. Chat Area & Messages
================================================== 
*/
.chat-area {
  width: 100%;
  max-width: 800px;
  padding: 20px 5%; /* [유지] 내부 메시지 패딩 */
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin: 0 auto;
  
  /* [추가] 스크롤 컨테이너 내부에서 내용이 잘리지 않도록 설정 */
  flex-shrink: 0; 
  min-height: 1px;
}

/* 메시지 스타일 */
.message {
  max-width: 85%;
  padding: 15px 20px;
  border-radius: 10px;
  line-height: 1.7;
  font-size: 15px;
  transition: all 0.2s ease;
}
/* ... (메시지 스타일 생략) ... */
.user-message {
  align-self: flex-end;
  background-color: var(--accent-color);
  color: var(--bg-color);
  border-bottom-right-radius: 2px;
  box-shadow: 0 2px 4px var(--shadow-color);
}
.system-message {
  align-self: flex-start;
  background-color: var(--surface-color);
  color: var(--text-color);
  border-bottom-left-radius: 2px;
  box-shadow: 0 2px 4px var(--shadow-color);
  display: flex;
  flex-direction: column;
}

/* 고급 마크다운 및 코드 블록 스타일 */
.system-message p {
    margin: 0 0 10px 0;
}
.system-message p:last-child {
    margin-bottom: 0;
}
.system-message strong {
    font-weight: 700;
}
/* 코드 블록 Wrapper */
.code-block-wrapper {
    position: relative;
    margin: 10px -20px -15px -20px; 
    padding-top: 20px;
    background-color: var(--code-bg);
    border-bottom-left-radius: 8px;
    border-bottom-right-radius: 8px;
    overflow: hidden;
}
.code-block {
    background-color: var(--code-bg);
    color: var(--code-text);
    padding: 0 20px 15px 20px;
    font-family: 'Consolas', 'Monaco', monospace;
    font-size: 14px;
    line-height: 1.5;
    white-space: pre-wrap; 
    overflow-x: auto;
}
.code-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #222; 
    padding: 5px 20px;
    font-size: 12px;
    color: var(--subtle-text);
}
.copy-code-button {
    background-color: var(--hover-bg);
    color: var(--subtle-text);
    border: none;
    padding: 5px 10px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 12px;
    transition: background-color 0.2s, color 0.2s;
}
.copy-code-button:hover {
    background-color: var(--accent-color);
    color: white;
}
/* 로딩 애니메이션 */
.loading {
  background-color: var(--hover-bg);
  display: flex;
  align-items: center;
  color: var(--subtle-text);
}
.dot-container {
  display: inline-flex;
  margin-right: 10px;
}
.dot-container span {
  display: block;
  width: 8px;
  height: 8px;
  background-color: var(--accent-color);
  border-radius: 50%;
  margin: 0 3px;
  animation: bounce 1.4s infinite ease-in-out both;
}
.dot-container span:nth-child(1) {
  animation-delay: -0.32s;
}
.dot-container span:nth-child(2) {
  animation-delay: -0.16s;
}
@keyframes bounce {
  0%, 80%, 100% { transform: scale(0); }
  40% { transform: scale(1); }
}

/* 상호작용 버튼 */
.interaction-area {
  display: flex;
  flex-wrap: wrap; 
  align-items: flex-start;
  gap: 10px;
  padding-top: 15px;
  border-top: 1px solid var(--input-border);
  margin-top: 10px;
}
#confirmGeneration {
  background-color: var(--confirm-btn-bg);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 25px;
  cursor: pointer;
  font-size: 15px;
  font-weight: 600;
  transition: background-color 0.2s, transform 0.1s;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}
#confirmGeneration:hover {
  background-color: #1976d2;
}

/* Secondary Action Button */
.copy-button, .secondary-action-button {
  background: none;
  border: 1px solid var(--input-border);
  color: var(--subtle-text);
  cursor: pointer;
  font-size: 14px;
  padding: 6px 10px;
  margin-left: 10px;
  border-radius: 6px;
  transition: color 0.2s, background-color 0.2s, border-color 0.2s;
  line-height: 1;
  white-space: nowrap;
}
.copy-button:hover, .secondary-action-button:hover {
  color: var(--accent-color);
  background-color: var(--hover-bg);
  border-color: var(--accent-color);
}
.image-copy-button {
  background-color: var(--highlight-color);
  color: white;
  border: none;
  padding: 10px 18px;
  border-radius: 25px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: background-color 0.2s;
}
.image-copy-button:hover {
  background-color: var(--accent-color);
}

/* Output Thumbnail */
.output-thumbnail {
    max-width: 100%;
    margin: 15px 0 0 0;
    border-radius: 8px;
    box-shadow: 0 4px 8px var(--shadow-color);
    cursor: pointer;
}

/* ==================================================
6. Input Bar & Controls
================================================== 
*/
.input-bar-area {
  width: 100%;
  padding: 15px 0;
  box-sizing: border-box;
  display: flex;
  justify-content: center;
  align-items: center; /* 추가: 내부 wrapper를 수직 중앙 정렬 */
  
  /* [유지] Fixed Position */
  position: fixed;
  bottom: var(--footer-height); /* 푸터 위 */
  left: 0;
  right: 0;
  
  /* [추가] height 명시 */
  height: var(--input-area-height);
  
  /* [수정] 배경색/경계선/그림자 제거 (사용자 요청: 박스 제거) */
  /* background-color: var(--surface-color); */ 
  /* border-top: 1px solid var(--input-border); */ 
  /* box-shadow: 0 -2px 5px var(--shadow-color); */ 
  z-index: 10;
  transition: all 0.5s ease-in-out; 
  top: auto; 
  transform: none; 
}

.input-bar-area.sidebar-open-fixed {
  left: var(--sidebar-width);
  width: calc(100% - var(--sidebar-width));
}

.input-wrapper {
  width: 90%;
  max-width: 800px;
  display: flex;
  align-items: flex-end;
  border: 2px solid var(--input-border);
  border-radius: 30px;
  padding: 10px 20px;
  padding-bottom: 5px; 
  box-sizing: border-box;
  background-color: var(--bg-color);
  position: relative;
  transition: border-color 0.3s, background-color 0.3s;
}
.input-wrapper:focus-within {
  border-color: var(--accent-color);
  box-shadow: 0 0 8px rgba(94, 59, 169, 0.4);
}
/* ... (Input Bar 내부 스타일 생략) ... */
.input-icon-group {
  display: flex;
  align-items: flex-end;
  margin-right: 15px;
  gap: 10px;
  padding-bottom: 2px;
}
.icon {
  font-size: 22px;
  color: var(--subtle-text);
  cursor: pointer;
  padding: 5px;
  border-radius: 50%;
  transition: color 0.2s, background-color 0.2s;
}
.icon:hover {
  color: var(--accent-color);
  background-color: var(--hover-bg);
}

.chat-textarea {
  flex-grow: 1;
  border: none;
  outline: none;
  resize: none;
  font-size: 16px;
  padding: 5px 0;
  max-height: 120px;
  overflow-y: auto;
  line-height: 1.6;
  min-height: 20px;
  color: var(--text-color);
  background-color: transparent;
  box-sizing: border-box;
}

/* 미디어 타입 및 파라미터 드롭다운 */
.option-group {
  position: relative;
  display: flex;
  align-items: center;
  margin: 5px 0 5px 15px;
  cursor: pointer;
  font-size: 15px;
  user-select: none;
  z-index: 10;
  color: var(--subtle-text);
  transition: color 0.2s;
}
.option-group:hover {
  color: var(--text-color);
}
.media-dropdown, .ratio-dropdown {
  padding-right: 15px;
  border-right: 1px solid var(--input-border);
}
.palette {
  display: none;
  position: absolute;
  bottom: 100%;
  right: 0;
  margin-bottom: 10px;
  border: 1px solid var(--input-border);
  border-radius: 8px;
  background-color: var(--surface-color);
  box-shadow: 0 4px 15px var(--shadow-color);
  overflow: hidden;
  min-width: 150px;
}
.palette-item {
  padding: 12px 18px;
  color: var(--text-color);
  font-size: 14px;
  transition: background-color 0.2s;
  display: flex;
  align-items: center;
  gap: 10px;
}
.palette-item:hover {
  background-color: var(--hover-bg);
  color: var(--accent-color);
}
.palette-item:not(:last-child) {
  border-bottom: 1px solid var(--input-border);
}

/* ---- 전송 버튼 (로고 심볼) 스타일 ---- */
.option-group.send-icon {
  padding-left: 15px;
}
.option-group.send-icon .symbol-svg {
    width: 35px; /* 전송 버튼으로서 조금 더 크게 */
    height: 35px;
    min-width: 35px;
    transform: perspective(400px) rotateY(-8deg) rotateX(10deg); 
    /* [수정] 호버 애니메이션을 위해 transition 유지 */
    transition: transform 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
    cursor: pointer;
}

/* [추가/수정] 전송 버튼 호버 효과 */
.option-group.send-icon .symbol-svg:hover {
    /* 크기 미약하게 커짐 */
    transform: scale(1.05) perspective(400px) rotateY(-8deg) rotateX(10deg);
    /* 발광 효과 애니메이션 시작 */
    animation: glow-pulse 1.5s infinite alternate;
}

.option-group.send-icon .left-half,
.option-group.send-icon .right-half {
    fill: var(--ultramarine-fill); /* [수정] 울트라 마린 색상으로 채우기 */
    stroke: var(--accent-color); /* 평소에는 강조색으로 표시 */
    stroke-width: 5; /* [수정] 외곽선 두께 설정 */
    transition: stroke 0.3s;
}

.option-group.send-icon:hover .left-half,
.option-group.send-icon:hover .right-half {
    stroke: var(--confirm-btn-bg); /* 호버 시 색상 변경 */
}
.option-group.send-icon:active .symbol-svg {
    transform: scale(0.95) perspective(400px) rotateY(-8deg) rotateX(10deg); /* 클릭 시 눌리는 효과 */
    animation: none; /* 클릭 시 애니메이션 중지 */
    box-shadow: none; /* 클릭 시 그림자 제거 */
}


/* ==================================================
7. Footer
================================================== 
*/
.footer {
  width: 100%;
  padding: 15px 5%; /* 기존 패딩 유지 */
  box-sizing: border-box;
  text-align: center;
  font-size: 11px;
  color: var(--subtle-text);
  background-color: var(--surface-color);
  border-top: 1px solid var(--input-border);
  
  /* [유지] Fixed Position */
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  
  /* [추가] height 명시 */
  height: var(--footer-height);

  z-index: 5;
  transition: all 0.3s;
}
.footer.sidebar-open-fixed {
  left: var(--sidebar-width);
  width: calc(100% - var(--sidebar-width));
}
.footer-content {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.company-info {
  text-align: left;
  line-height: 1.6;
}
#languageSelector {
  padding: 6px 10px;
  background-color: var(--bg-color);
  border: 1px solid var(--input-border);
  color: var(--subtle-text);
  border-radius: 6px;
  font-size: 11px;
  cursor: pointer;
  transition: background-color 0.3s, border-color 0.2s;
}
#languageSelector:hover {
  border-color: var(--accent-color);
}
</style>
</head>
<body class="dark-mode">

<aside id="historySidebar">
  <div class="sidebar-header">
    <h3 style="color: var(--accent-color); margin: 0; font-weight: 700;" data-i18n="history_title">대화 기록</h3>
    <p style="font-size: 13px; color: var(--subtle-text);" data-i18n="history_description">최근 대화 목록이 여기에 표시됩니다. 항목 옆 버튼으로 관리하세요.</p>
    <button class="close-sidebar-btn" onclick="toggleSidebar()">×</button>
  </div>
  <hr style="border-color: var(--input-border); margin: 15px 0;">
  <div id="historyList">
    </div>
</aside>

<div id="chatContainer">
  
  <div class="header-container" id="headerContainer">
    <div class="header-content">
      <div class="logo-group">
        <span class="sidebar-toggle" onclick="toggleSidebar()" data-i18n-title="toggle_history_title" title="Toggle History">☰</span>
        
        <div class="logo-symbol" id="logoSymbol">
          <svg viewBox="0 0 100 100" class="symbol-svg" aria-label="기하학적 로고">
            <path class="left-half" d="M 10 10 L 65 10 L 65 25 L 25 25 L 25 75 L 10 75 Z" />
            <path class="right-half" d="M 90 25 L 90 90 L 45 90 L 45 75 L 75 75 L 75 25 Z" />
          </svg>
        </div>
        
        <div class="logo-text" id="logoText">Promptly</div>
        
        <select id="modelSelector">
          <option value="GPT-4 Turbo">GPT-4 Turbo</option>
          <option value="DALL-E 3">DALL-E 3</option>
          <option value="Claude 3 Opus">Claude 3 Opus</option>
        </select>
      </div>
      
      <div class="user-profile">
        <button id="darkModeToggle" onclick="toggleDarkMode()" data-i18n-title="toggle_theme_title" title="Change Theme">☀라이트 모드</button>
        
        <span class="token-display" data-i18n="token_balance_label">토큰 잔량:</span> <strong id="tokenDisplay">500</strong>
        
        <div class="profile-icon" id="profileIcon">P</div>
        <div id="settingsDropdown">
            <div class="setting-group">
                <h4 data-i18n="settings_ai_params_title">AI 모델 설정</h4>
                <label style="font-size: 13px; color: var(--subtle-text);" for="systemPromptInput" data-i18n="settings_system_prompt_label">시스템 프롬프트 (AI 성격):</label>
                <textarea id="systemPromptInput" rows="3" data-i18n-placeholder="settings_system_prompt_placeholder" placeholder="AI에게 특정 역할이나 성격을 부여하세요..."></textarea>
                
                <label style="font-size: 13px; color: var(--subtle-text); margin-top: 10px; display: block;" for="temperatureSlider" data-i18n="settings_temperature_label">창의성(Temperature): <span id="temperatureValue">0.7</span></label>
                <input type="range" id="temperatureSlider" min="0.0" max="1.0" step="0.1" value="0.7">
                
                <label style="font-size: 13px; color: var(--subtle-text); margin-top: 10px; display: block;" for="seedInput" data-i18n="settings_seed_label">재현성(Seed):</label>
                <input type="number" id="seedInput" value="1337" min="1" max="99999">
            </div>
            
            <div class="setting-item" data-action="logout" data-i18n="settings_logout"> 로그아웃</div>
            <div class="setting-item" data-action="billing" data-i18n="settings_billing"> 결제 관리</div>
            <div class="setting-item" data-action="support" data-i18n="settings_support"> 고객 지원</div>
        </div>
      </div>
    </div>
  </div>
  
  <div class="chat-area" id="chatArea">
    </div>
  
</div>

<div class="input-bar-area" id="inputBarArea">
  <div class="input-wrapper">
    <div class="input-icon-group">
      <span class="icon" id="attachFile" data-i18n-title="input_attach_title" title="파일 첨부 (Drag & Drop 가능)"></span>
      <input type="file" id="fileInput" style="display: none;" accept="image/*" multiple>
      
    </div>
    
    <textarea 
      class="chat-textarea" 
      id="promptInput" 
      data-i18n-placeholder="input_placeholder"
      placeholder="프롬프트를 적어주세요" 
      rows="1"
    ></textarea>
    
    <div class="option-group media-dropdown" id="mediaDropdown">
      <span id="currentMediaType" data-i18n="media_image">이미지</span> <span style="margin-left: 5px;">▼</span>
      
      <div class="palette" id="mediaPalette">
        <div class="palette-item" data-type="이미지" data-i18n="media_image">🖼️ 이미지</div>
        <div class="palette-item" data-type="영상" data-i18n="media_video">🎥 영상</div>
        <div class="palette-item" data-type="텍스트" data-i18n="media_text">📝 텍스트</div>
      </div>
    </div>
    
    <div class="option-group ratio-dropdown" id="ratioDropdown" style="display: flex;">
        <span id="currentRatio" data-i18n="output_ratio_16_9">16:9</span> <span style="margin-left: 5px;">▼</span>
        <div class="palette" id="ratioPalette">
            <div class="palette-item" data-ratio="16:9" data-i18n="output_ratio_16_9">16:9 (가로)</div>
            <div class="palette-item" data-ratio="1:1" data-i18n="output_ratio_1_1">1:1 (정사각형)</div>
            <div class="palette-item" data-ratio="9:16" data-i18n="output_ratio_9_16">9:16 (세로)</div>
        </div>
    </div>
    
    <div class="option-group send-icon" id="sendButton">
      <svg viewBox="0 0 100 100" class="symbol-svg" aria-label="전송 로고 버튼">
        <path class="left-half" d="M 10 10 L 65 10 L 65 25 L 25 25 L 25 75 L 10 75 Z" />
        <path class="right-half" d="M 90 25 L 90 90 L 45 90 L 45 75 L 75 75 L 75 25 Z" />
      </svg>
    </div>
  </div>
</div>

<footer class="footer" id="footerArea">
  <div class="footer-content">
    <div class="company-info">
      (주)프롬프트스 | 경기 수원시 영통구 광교로 154-42 광교비즈니스센터 507, 508<br>
      대표이사: 강** | 사업자등록번호: 581-81-02558 | 통신판매업 신고번호: 2023-수원영통-0396 | 전화번호: 0502-1910-3341
    </div>
    <select id="languageSelector" onchange="changeLanguage()">
      <option value="ko">한국어 (KO)</option>
      <option value="en">English (EN)</option>
      <option value="zh">中文 (ZH)</option>
    </select>
  </div>
</footer>

<script>
  // ===============================================
  // 1. 다국어 텍스트 정의
  // ===============================================
  const translations = {
    ko: {
      toggle_history_title: "대화 기록 토글",
      toggle_theme_title: "테마 변경",
      history_title: "대화 기록",
      history_description: "최근 대화 목록이 여기에 표시됩니다. 항목 옆 버튼으로 관리하세요.",
      token_balance_label: "토큰 잔량:",
      settings_logout: " 로그아웃",
      settings_billing: " 결제 관리",
      settings_support: " 고객 지원",
      settings_ai_params_title: "AI 모델 고급 설정",
      settings_system_prompt_label: "시스템 프롬프트 (AI 성격):",
      settings_system_prompt_placeholder: "AI에게 특정 역할이나 성격을 부여하세요...",
      settings_temperature_label: "창의성(Temperature):",
      settings_seed_label: "재현성(Seed):",
      input_attach_title: "파일 첨부 (Drag & Drop 가능)",
      input_placeholder: "프롬프트를 적어주세요",
      media_image: "이미지",
      media_video: "영상",
      media_text: "텍스트",
      output_ratio_16_9: "16:9",
      output_ratio_1_1: "1:1",
      output_ratio_9_16: "9:16",
      alert_loaded_prompt: "프롬프트가 입력창에 로드되었습니다.",
      alert_no_prompt: "저장하거나 로드할 프롬프트가 없습니다.",
      alert_type_changed: "미디어 타입이 %s로 바뀌었습니다.", 
      alert_ratio_changed: "출력 종횡비가 %s로 바뀌었습니다.", 
      alert_copy_success: "프롬프트가 복사되었습니다",
      alert_code_copy_success: "코드가 클립보드에 복사되었습니다.",
      alert_copy_fail: "클립보드 복사에 실패했습니다.",
      alert_link_success: "링크가 클립보드에 복사되었습니다.",
      alert_link_none: "텍스트 결과물은 별도의 링크가 없습니다. 메시지 내용을 직접 복사해주세요.",
      alert_token_insufficient: "❌ **토큰이 부족합니다!** 요청을 처리할 수 없습니다. (요구 토큰: %s🪙)", 
      alert_history_pinned: "대화가 기록에 고정되었습니다.",
      alert_history_deleted: "대화 기록이 삭제되었습니다.",
      alert_qr_generated: "QR 코드 생성 요청 (%s 링크)이 서버로 전송되었습니다.",
      system_initial: "프롬프틀리입니다. **%s** 모델이 선택되었습니다. 시스템 프롬프트: **%s**",
      system_prompt_request: "업로드된 이미지를 분석하여 **프롬프트 초안**을 생성해 드릴까요?",
      system_prompt_button: "프롬프트 생성 (%s🪙)", 
      system_loading: "**응답 준비 중...**",
      system_analysis_done: "✅ **이미지 분석 완료.**",
      system_generated_prompt: "✨ 생성된 프롬프트 초안:",
      system_insert_prompt: "프롬프트 복사 및 입력창에 삽입 ",
      system_generation_confirmation: "**%s** 생성을 위한 프롬프트 생성이 완료되었습니다.<br>\"%s...\"<br>이대로 %s를 생성하시겠습니까?", 
      system_generate_button: "%s 생성",
      system_generation_done: "✅ **%s 생성 완료.**",
      system_copy_link: "%s 링크 복사 ",
      system_generate_qr: "QR/Embed 코드 생성",
      system_error_image_only: "❌ **오류:** 이미지 파일만 처리할 수 있습니다.",
      system_error_token_analysis: "❌ **토큰이 부족합니다!** 프롬프트 생성을 처리할 수 없습니다.",
      system_error_drag: "❌ **오류:** 이미지 파일만 드래그 앤 드롭으로 업로드할 수 있습니다.",
    },
    en: {
        toggle_history_title: "Toggle Chat History",
        toggle_theme_title: "Change Theme",
        history_title: "Chat History",
        history_description: "Recent conversations are displayed here. Manage items using the buttons.",
        token_balance_label: "Token Balance:",
        settings_logout: " Logout",
        settings_billing: " Billing Management",
        settings_support: " Customer Support",
        settings_ai_params_title: "AI Model Advanced Settings",
        settings_system_prompt_label: "System Prompt (AI Personality):",
        settings_system_prompt_placeholder: "Assign a specific role or personality to the AI...",
        settings_temperature_label: "Creativity (Temperature):",
        settings_seed_label: "Reproducibility (Seed):",
        input_attach_title: "Attach File (Drag & Drop available)",
        input_placeholder: "Enter your prompt here",
        media_image: "Image",
        media_video: "Video",
        media_text: "Text",
        output_ratio_16_9: "16:9",
        output_ratio_1_1: "1:1",
        output_ratio_9_16: "9:16",
        alert_loaded_prompt: "Prompt loaded into the input field.",
        alert_no_prompt: "No prompt to save or load.",
        alert_type_changed: "Media type changed to %s.", 
        alert_ratio_changed: "Output ratio changed to %s.", 
        alert_copy_success: "Prompt copied successfully",
        alert_code_copy_success: "Code copied to clipboard.",
        alert_copy_fail: "Failed to copy to clipboard.",
        alert_link_success: "Link copied to clipboard.",
        alert_link_none: "Text results do not have a separate link. Please copy the message content directly.",
        alert_token_insufficient: "❌ **Insufficient Tokens!** Cannot process request. (Required: %s🪙)", 
        alert_history_pinned: "Conversation pinned to history.",
        alert_history_deleted: "Conversation history deleted.",
        alert_qr_generated: "QR Code generation request (%s link) sent to server.",
        system_initial: "This is Promptly. **%s** model selected. System Prompt: **%s**",
        system_prompt_request: "Shall I analyze the uploaded image and generate a **draft prompt**?",
        system_prompt_button: "Generate Prompt (%s🪙)", 
        system_loading: "**Preparing response...**",
        system_analysis_done: "✅ **Image analysis complete.**",
        system_generated_prompt: "✨ Generated Draft Prompt:",
        system_insert_prompt: "Copy Prompt and Insert into Input ",
        system_generation_confirmation: "Prompt generation for **%s** is complete.<br>\"%s...\"<br>Proceed with %s generation?", 
        system_generate_button: "Generate %s",
        system_generation_done: "✅ **%s Generation Complete.**",
        system_copy_link: "Copy %s Link ",
        system_generate_qr: "Generate QR/Embed Code",
        system_error_image_only: "❌ **Error:** Only image files can be processed.",
        system_error_token_analysis: "❌ **Insufficient Tokens!** Cannot process prompt generation.",
        system_error_drag: "❌ **Error:** Only image files can be uploaded via Drag & Drop.",
    },
    zh: {
        toggle_history_title: "切换聊天记录",
        toggle_theme_title: "更改主题",
        history_title: "聊天记录",
        history_description: "最近的对话将显示在此处。使用按钮管理项目。",
        token_balance_label: "代币余额:",
        settings_logout: " 登出",
        settings_billing: " 账单管理",
        settings_support: " 客户支持",
        settings_ai_params_title: "AI 模型高级设置",
        settings_system_prompt_label: "系统提示 (AI 个性):",
        settings_system_prompt_placeholder: "为 AI 分配特定的角色或个性...",
        settings_temperature_label: "创造力(Temperature):",
        settings_seed_label: "重现性(Seed):",
        input_attach_title: "附件 (支持拖放)",
        input_placeholder: "请输入您的提示",
        media_image: "图像",
        media_video: "视频",
        media_text: "文本",
        output_ratio_16_9: "16:9",
        output_ratio_1_1: "1:1",
        output_ratio_9_16: "9:16",
        alert_loaded_prompt: "提示已加载到输入字段。",
        alert_no_prompt: "没有可保存或加载的提示。",
        alert_type_changed: "媒体类型已更改为 %s。", 
        alert_ratio_changed: "输出比例已更改为 %s。", 
        alert_copy_success: "提示已成功复制",
        alert_code_copy_success: "代码已复制到剪贴板。",
        alert_copy_fail: "复制到剪贴板失败。",
        alert_link_success: "链接已复制到剪贴板。",
        alert_link_none: "文本结果没有单独的链接。请直接复制消息内容。",
        alert_token_insufficient: "❌ **代币不足!** 无法处理请求。 (需要代币: %s🪙)", 
        alert_history_pinned: "对话已固定到记录。",
        alert_history_deleted: "对话记录已删除。",
        alert_qr_generated: "QR 码生成请求 (%s 链接)已发送至服务器。",
        system_initial: "这是 Promptly。已选择 **%s** 模型。系统提示: **%s**",
        system_prompt_request: "我分析上传的图像并生成一个**提示草稿**吗？",
        system_prompt_button: "生成提示 (%s🪙)", 
        system_loading: "**正在准备响应...**",
        system_analysis_done: "✅ **图像分析完成。**",
        system_generated_prompt: "✨ 生成的提示草稿:",
        system_insert_prompt: "复制提示并插入到输入 ",
        system_generation_confirmation: "**%s** 的提示生成已完成。<br>\"%s...\"<br>现在生成 %s 吗？", 
        system_generate_button: "生成 %s",
        system_generation_done: "✅ **%s 生成完成。**",
        system_copy_link: "复制 %s 链接 ",
        system_generate_qr: "生成 QR/嵌入代码",
        system_error_image_only: "❌ **错误:** 只能处理图像文件。",
        system_error_token_analysis: "❌ **代币不足!** 无法处理提示生成。",
        system_error_drag: "❌ **错误:** 只能通过拖放上传图像文件。",
    }
  };

  // ===============================================
  // 2. 전역 변수 및 초기화 설정
  // ===============================================
  const body = document.body;
  const chatContainer = document.getElementById('chatContainer');
  const textarea = document.getElementById('promptInput');
  const sendButton = document.getElementById('sendButton');
  const chatArea = document.getElementById('chatArea');
  const currentMediaType = document.getElementById('currentMediaType');
  const mediaPalette = document.getElementById('mediaPalette');
  const ratioPalette = document.getElementById('ratioPalette');
  const sidebar = document.getElementById('historySidebar');
  const historyList = document.getElementById('historyList');
  const modelSelector = document.getElementById('modelSelector');
  const profileIcon = document.getElementById('profileIcon');
  const settingsDropdown = document.getElementById('settingsDropdown');
  const fileInput = document.getElementById('fileInput');
  const temperatureSlider = document.getElementById('temperatureSlider');
  const temperatureValueSpan = document.getElementById('temperatureValue');
  const systemPromptInput = document.getElementById('systemPromptInput');
  const inputBarArea = document.getElementById('inputBarArea');
  const footerArea = document.getElementById('footerArea');
  const headerContainer = document.getElementById('headerContainer');
  const languageSelector = document.getElementById('languageSelector');

  let selectedMediaType = '이미지';
  let selectedRatio = '16:9';
  let currentTokenCount = 500; // 🎯 초기 토큰 잔량 설정
  let lastUserPrompt = '';
  let currentLang = 'ko';
  let systemPrompt = '당신은 사용자에게 가장 최적화된 콘텐츠를 생성하는 만능 AI 비서입니다.';

  const generatedLinks = {
    '이미지': 'https://promptly.ai/generated_content/image_12345.png',
    '영상': 'https://promptly.ai/generated_content/video_98765.mp4',
    '텍스트': '텍스트 결과물 자체를 복사할 수 있습니다. (링크 없음)',
  };
  
  // 🚨 고객 지원 URL 정의
  const SUPPORT_URL = 'https://promptly.ai/support'; 


  // ===============================================
  // 3. 기능 함수 (토큰 시스템 및 관련 로직 추가/수정)
  // ===============================================
  
  // 3.1. 다국어/초기화 (I18n)
  function getTranslation(key, ...args) {
    let text = translations[currentLang][key] || translations['ko'][key] || key;
    if (args.length > 0) {
      for (const arg of args) {
        text = text.replace('%s', arg);
      }
    }
    return text;
  }
  
  // [추가] HTML 요소에 다국어 적용
  function applyTranslations() {
    const elements = document.querySelectorAll('[data-i18n]');
    elements.forEach(el => {
      const key = el.getAttribute('data-i18n');
      const translation = getTranslation(key);
      // ''와 같은 폰트 아이콘은 그대로 유지
      if (el.textContent.includes('')) {
          el.textContent = el.textContent.replace(el.textContent.split(' ')[1], translation.split(' ')[1]);
      } else {
          el.textContent = translation;
      }
    });

    const placeholders = document.querySelectorAll('[data-i18n-placeholder]');
    placeholders.forEach(el => {
      const key = el.getAttribute('data-i18n-placeholder');
      el.placeholder = getTranslation(key);
    });

    const titles = document.querySelectorAll('[data-i18n-title]');
    titles.forEach(el => {
      const key = el.getAttribute('data-i18n-title');
      el.title = getTranslation(key);
    });
    
    // 미디어 타입 드롭다운의 텍스트도 업데이트
    updateMediaTypeDisplay(); 
  }
  
  // [추가] 현재 선택된 미디어 타입 텍스트를 현재 언어에 맞게 업데이트
  function updateMediaTypeDisplay() {
      const typeKey = selectedMediaType === '이미지' ? 'image' : selectedMediaType === '영상' ? 'video' : 'text';
      currentMediaType.textContent = getTranslation(`media_${typeKey}`);
      document.getElementById('currentRatio').textContent = selectedRatio;
      
      // 드롭다운 팔레트 항목 텍스트 업데이트
      document.querySelectorAll('#mediaPalette .palette-item').forEach(item => {
          const dataType = item.getAttribute('data-type');
          const typeKeyItem = dataType === '이미지' ? 'image' : dataType === '영상' ? 'video' : 'text';
          const icon = item.textContent.trim().split(' ')[0];
          item.textContent = `${icon} ${getTranslation(`media_${typeKeyItem}`)}`;
      });
      document.querySelectorAll('#ratioPalette .palette-item').forEach(item => {
          const dataRatio = item.getAttribute('data-ratio');
          const ratioKey = `output_ratio_${dataRatio.replace(':', '_')}`;
          item.textContent = getTranslation(ratioKey);
      });
  }


  // [수정] 언어 변경 함수
  window.changeLanguage = function() {
    currentLang = languageSelector.value;
    applyTranslations();
    resetChat();
    // 💡 새로운 언어로 시스템 프롬프트 다시 설정
    systemPrompt = currentLang === 'en' ? 'You are a versatile AI assistant that generates the most optimized content for the user.' :
                   currentLang === 'zh' ? '您是一个多功能AI助手，可以为用户生成最优化的内容。' :
                   '당신은 사용자에게 가장 최적화된 콘텐츠를 생성하는 만능 AI 비서입니다.';
    
    // 시스템 프롬프트 입력창의 placeholder와 value도 업데이트 (placeholder는 applyTranslations가 처리)
    if (!systemPromptInput.value.trim()) {
        systemPromptInput.value = '';
    }
  }


  // 3.2. [수정] 채팅 상태 업데이트 (레이아웃 고정으로 인한 패딩 제거)
  function updateChatState() {
    // 채팅 영역에 'system' 또는 'user' 메시지가 1개 이상 있는지 확인합니다.
    const hasMessages = chatArea.querySelector('.message') !== null;
    
    if (hasMessages) {
        body.classList.remove('initial-state');
        chatArea.style.display = 'flex'; // 채팅 영역 보이기
    } else {
        body.classList.add('initial-state');
        chatArea.style.display = 'none'; // 채팅 영역 숨기기
    }
    // 💡 레이아웃 고정 방식으로 변경되어 chatContainer의 동적 패딩 설정 로직을 제거했습니다.
  }


  function resetChat() {
    chatArea.innerHTML = '';
    lastUserPrompt = '';
    textarea.value = '';
    autoResize();
    document.getElementById('tokenDisplay').textContent = currentTokenCount.toLocaleString(); // 🎯 토큰 잔량 표시
    
    // 🎯 초기 메시지 출력을 제거하고 상태 업데이트만 합니다.
    updateChatState(); 
    // 전체 페이지 스크롤이 막혔으므로, 스크롤을 chatContainer로 변경
    if (chatContainer) chatContainer.scrollTop = 0; 
  }

  // 3.3. 토큰 비용 예측 (기존 유지)
  function calculateTokenCost(type) {
    switch (type) {
      case '이미지':
        return 30; // 🎯 이미지 토큰 비용
      case '영상':
      case '영상/이미지': // 히스토리 저장 시 "영상/이미지"로 저장될 수 있음
        return 70; // 🎯 영상 토큰 비용
      case '텍스트':
        return 20; // 🎯 텍큰스 토큰 비용
      default:
        return 0;
    }
  }

  // 3.4. 토큰 잔량 업데이트 및 표시 (기존 유지)
  function updateTokenDisplay(tokensUsed) {
    currentTokenCount -= tokensUsed;
    document.getElementById('tokenDisplay').textContent = currentTokenCount.toLocaleString();
  }

  // 3.5. 고급 마크다운 및 코드 블록 처리 (기존 유지)
  function renderMarkdown(text) {
    const codeBlockRegex = /```(\w*)\n([\s\S]*?)```/g;
    let result = text.replace(codeBlockRegex, (match, lang, code) => {
        const language = lang || 'Text';
        const codeHtml = `<pre class="code-block">${code.trim().replace(/</g, '&lt;').replace(/>/g, '&gt;')}</pre>`;
        const copyButtonText = currentLang === 'zh' ? '复制代码' : currentLang === 'en' ? 'Copy Code' : '코드 복사';
        const header = `<div class="code-header"><span>${language}</span><button class="copy-code-button" onclick="copyCode(this)"> ${copyButtonText}</button></div>`;
        return `<div class="code-block-wrapper" data-code="${btoa(code.trim())}">${header}${codeHtml}</div>`;
    });

    result = result.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>'); 
    result = result.replace(/\*(.*?)\*/g, '<em>$1</em>'); 
    result = result.replace(/\n/g, '<br>'); 

    return result;
  }
  
  window.copyCode = function(button) {
    const codeWrapper = button.closest('.code-block-wrapper');
    const encodedCode = codeWrapper.getAttribute('data-code');
    const code = atob(encodedCode);
    
    navigator.clipboard.writeText(code).then(() => {
        alert(`✅ ${getTranslation('alert_code_copy_success')}`);
    }).catch(err => {
        console.error('클립보드 복사 실패:', err);
        alert(`❌ ${getTranslation('alert_copy_fail')}`);
    });
  }
  
  // 3.6. 대화 기록 관리 강화 (기존 유지)
  function addHistoryItem(prompt, type, isPinned = false) {
    const displayPrompt = prompt.substring(0, 30);
    const p = document.createElement('p');
    p.setAttribute('data-prompt', prompt);
    p.setAttribute('data-type', type);
    p.setAttribute('data-pinned', isPinned);

    let icon = type.includes('이미지') ? '🖼️' : type.includes('영상') ? '🎥' : '📝';
    const typeKey = type.includes('이미지') ? 'image' : type.includes('영상') ? 'video' : 'text';
    let typeText = getTranslation(`media_${typeKey}`);
    
    p.innerHTML = `${icon} ${typeText}: ${displayPrompt}...`;
    p.addEventListener('click', loadHistoryPrompt);
    
    const actions = document.createElement('div');
    actions.classList.add('history-actions');
    
    const pinBtn = document.createElement('button');
    pinBtn.classList.add('pin-btn');
    pinBtn.textContent = isPinned ? '📌' : '📍';
    if (isPinned) pinBtn.classList.add('pinned');
    pinBtn.onclick = (e) => { 
        e.stopPropagation();
        togglePin(p);
    };
    
    const deleteBtn = document.createElement('button');
    deleteBtn.textContent = '❌';
    deleteBtn.onclick = (e) => { 
        e.stopPropagation();
        deleteHistoryItem(p);
    };
    
    actions.appendChild(pinBtn);
    actions.appendChild(deleteBtn);
    p.appendChild(actions);

    if (historyList.firstChild) {
      historyList.insertBefore(p, historyList.firstChild);
    } else {
      historyList.appendChild(p);
    }
  }
  
  function loadHistoryPrompt(e) {
    const prompt = e.currentTarget.getAttribute('data-prompt');
    const type = e.currentTarget.getAttribute('data-type').split(' ')[0]; 
    textarea.value = prompt;
    selectedMediaType = type;
    updateMediaTypeDisplay(); // 언어에 맞춰서 업데이트
    document.getElementById('ratioDropdown').style.display = type === '텍스트' ? 'none' : 'flex';
    autoResize();
    alert(`💡 ${getTranslation('alert_loaded_prompt')}`);
  }

  function togglePin(item) {
    const isPinned = item.getAttribute('data-pinned') === 'true';
    item.setAttribute('data-pinned', !isPinned);
    item.querySelector('.pin-btn').textContent = !isPinned ? '📌' : '📍';
    item.querySelector('.pin-btn').classList.toggle('pinned', !isPinned);
    alert(`💡 ${getTranslation('alert_history_pinned')}`);
  }

  function deleteHistoryItem(item) {
    if (confirm('이 대화 기록을 삭제하시겠습니까?')) {
        item.remove();
        alert(`🗑️ ${getTranslation('alert_history_deleted')}`);
    }
  }

  // 3.7. 메시지 전송 및 처리 (기존 유지)
  function appendMessage(content, type, isTyping = false, parentElement = chatArea) {
    const messageDiv = document.createElement('div');
    messageDiv.classList.add('message', `${type}-message`);
    
    if (type === 'system' && !isTyping) {
        messageDiv.innerHTML = renderMarkdown(content);
    } else if (isTyping) {
        messageDiv.classList.add('loading');
        messageDiv.innerHTML = `<div class="dot-container"><span></span><span></span><span></span></div> ${content}`;
    } else {
      messageDiv.innerHTML = content;
    }
    
    parentElement.appendChild(messageDiv);
    // 💡 전체 페이지 스크롤 대신 #chatContainer 스크롤
    if (chatContainer) chatContainer.scrollTop = chatContainer.scrollHeight;
    return messageDiv;
  }

  // 3.8. 이미지/영상 생성 결과 처리 (토큰 차감 로직 추가)
  function processResponse(cost, interactionArea, userPrompt) { 
    if (interactionArea) interactionArea.remove();

    // 🎯 토큰 잔량 재확인 및 차감
    if (currentTokenCount < cost) {
        // 이미 sendMessage에서 체크했지만, 혹시 모를 race condition 방지
        appendMessage(`❌ **${getTranslation('alert_token_insufficient').split('**')[1]}** (요구 토큰: ${cost}🪙)`, 'system');
        return;
    }
    updateTokenDisplay(cost); 
    
    const mediaTypeKey = selectedMediaType === '이미지' ? 'image' : selectedMediaType === '영상' ? 'video' : 'text';
    const mediaTypeText = getTranslation(`media_${mediaTypeKey}`);

    const resultMessageDiv = document.createElement('div');
    resultMessageDiv.classList.add('message', 'system-message');
    
    const content = selectedMediaType !== '텍스트' ? 
        getTranslation('system_generation_done', mediaTypeText) : 
        `**${getTranslation('input_placeholder').split(' ')[0]}:** ${userPrompt}\n\n**${getTranslation('settings_ai_params_title').split(' ')[0]}:** ${modelSelector.value}\n\n**${getTranslation('media_text').split(' ')[0]}:**\n\`\`\`html\n<!DOCTYPE html>\n<html>\n  <head>\n    <title>Generated Content</title>\n  </head>\n  <body>\n    <p>텍스트 결과물 시뮬레이션입니다.</p>\n  </body>\n</html>\n\`\`\``;

    const typingDiv = appendMessage(getTranslation('system_loading'), 'system', true, resultMessageDiv);
    typingDiv.classList.remove('loading');
    typingDiv.innerHTML = '';
    typingDiv.classList.add('typing-effect');
    chatArea.appendChild(resultMessageDiv); 
    
    let i = 0;
    const fullMessage = content;
    
    const typingInterval = setInterval(() => {
        if (selectedMediaType === '텍스트' && i === 0) {
            typingDiv.innerHTML = renderMarkdown(fullMessage);
            i = fullMessage.length; 
        }
        
        if (i < fullMessage.length) {
            if (selectedMediaType !== '텍스트') typingDiv.innerHTML += fullMessage.charAt(i);
            i++;
            // 💡 전체 페이지 스크롤 대신 #chatContainer 스크롤
            if (chatContainer) chatContainer.scrollTop = chatContainer.scrollHeight;
        } else {
            clearInterval(typingInterval);
            typingDiv.classList.remove('typing-effect');
            
            const resultInteractionArea = document.createElement('div');
            resultInteractionArea.classList.add('interaction-area');
            
            if (selectedMediaType !== '텍스트') {
                const thumbnail = document.createElement('img');
                thumbnail.classList.add('output-thumbnail');
                thumbnail.src = 'https://picsum.photos/400/225?random=' + Date.now(); 
                thumbnail.alt = mediaTypeText + ' Generated Content';
                resultMessageDiv.appendChild(thumbnail);

                const copyLinkBtn = document.createElement('button');
                copyLinkBtn.classList.add('image-copy-button');
                copyLinkBtn.textContent = getTranslation('system_copy_link', mediaTypeText);
                copyLinkBtn.onclick = () => imageCopyLinkToClipboard(selectedMediaType);
                
                const qrBtn = document.createElement('button');
                qrBtn.classList.add('secondary-action-button');
                qrBtn.textContent = getTranslation('system_generate_qr');
                qrBtn.onclick = () => alert(`💡 ${getTranslation('alert_qr_generated', generatedLinks[selectedMediaType])}`);

                resultInteractionArea.appendChild(copyLinkBtn);
                resultInteractionArea.appendChild(qrBtn);
            } else {
                const copyOriginalPromptBtn = document.createElement('button');
                copyOriginalPromptBtn.classList.add('copy-button');
                //  아이콘은 여기서 직접 넣어주고, 텍스트는 번역
                copyOriginalPromptBtn.textContent = ` ${getTranslation('system_insert_prompt').split(' ')[1]} ${getTranslation('system_insert_prompt').split(' ')[2]}`; 
                copyOriginalPromptBtn.onclick = () => copyPromptToClipboard(userPrompt);
                resultInteractionArea.appendChild(copyOriginalPromptBtn);
            }
            
            resultMessageDiv.appendChild(resultInteractionArea);
            // 💡 전체 페이지 스크롤 대신 #chatContainer 스크롤
            if (chatContainer) chatContainer.scrollTop = chatContainer.scrollHeight;
        }
    }, 50);
  }

  // --- 전송 로직 함수 (토큰 체크 로직 추가) ---
  
  function showConfirmation(userPrompt, cost) { // 🎯 cost 인자 추가
    const loadingIndicator = document.getElementById('loadingIndicator');
    if (loadingIndicator) loadingIndicator.remove();
    
    const mediaTypeKey = selectedMediaType === '이미지' ? 'image' : selectedMediaType === '영상' ? 'video' : 'text';
    const mediaTypeText = getTranslation(`media_${mediaTypeKey}`);

    const confirmMessageDiv = document.createElement('div');
    confirmMessageDiv.classList.add('message', 'system-message');
    
    // 🎯 토큰 비용 정보 추가
    const tokenLabel = getTranslation('token_balance_label').replace(':', '');
    const tokenInfo = `<br><span style="color: var(--accent-color); font-weight: bold;">[${tokenLabel.trim().split(' ')[0]} ${tokenLabel.trim().split(' ')[1] || ''}: ${cost} 🪙]</span>`;
    
    const confirmationMessage = getTranslation('system_generation_confirmation', mediaTypeText, userPrompt.substring(0, 30), mediaTypeText) + tokenInfo;
    confirmMessageDiv.innerHTML = renderMarkdown(confirmationMessage);

    const interactionArea = document.createElement('div');
    interactionArea.classList.add('interaction-area');
    
    const confirmBtn = document.createElement('button');
    confirmBtn.id = 'confirmGeneration';
    confirmBtn.textContent = getTranslation('system_generate_button', mediaTypeText);
    confirmBtn.onclick = () => processResponse(cost, interactionArea, userPrompt); // 🎯 cost 전달
    
    interactionArea.appendChild(confirmBtn);

    confirmMessageDiv.appendChild(interactionArea);
    chatArea.appendChild(confirmMessageDiv);
    // 💡 전체 페이지 스크롤 대신 #chatContainer 스크롤
    if (chatContainer) chatContainer.scrollTop = chatContainer.scrollHeight;
  }

  function sendMessage() {
    const userText = textarea.value.trim();
    if (userText === '') return;

    const isFirstMessage = body.classList.contains('initial-state');
    
    // 1. 🎯 토큰 비용 계산
    const cost = calculateTokenCost(selectedMediaType);
    
    // 2. 🎯 토큰 부족 확인
    if (currentTokenCount < cost) {
        appendMessage(getTranslation('alert_token_insufficient', cost), 'system');
        const existingConfirmation = document.querySelector('.interaction-area');
        if (existingConfirmation) existingConfirmation.closest('.message').remove();
        const loadingIndicator = document.getElementById('loadingIndicator');
        if (loadingIndicator) loadingIndicator.remove();
        return;
    }

    const existingConfirmation = document.querySelector('.interaction-area');
    if (existingConfirmation) existingConfirmation.closest('.message').remove();

    const mediaType = selectedMediaType;
    lastUserPrompt = userText;
    const mediaTypeKey = selectedMediaType === '이미지' ? 'image' : selectedMediaType === '영상' ? 'video' : 'text';
    const mediaTypeText = getTranslation(`media_${mediaTypeKey}`);

    // 💡 첫 메시지인 경우, 먼저 환영 메시지를 표시하고 레이아웃을 전환합니다.
    if (isFirstMessage) {
        // 1. 환영 메시지 출력
        const selectedModel = modelSelector.value;
        const initialMessageText = getTranslation('system_initial', selectedModel, systemPrompt.substring(0, 20) + '...');
        // chatArea를 보이게 합니다.
        chatArea.style.display = 'flex'; 
        appendMessage(initialMessageText, 'system');
        
        // 2. 레이아웃 전환 (Fixed Bottom으로 이동)
        body.classList.remove('initial-state');
        updateChatState(); // 일반 상태의 CSS 적용
    }
    
    // 3. 사용자 메시지 출력
    appendMessage(`[${mediaTypeText}] ${userText}`, 'user');
    addHistoryItem(userText, mediaType);
    
    textarea.value = '';
    autoResize();
    
    showLoading();
    // 🎯 토큰 비용을 showConfirmation에 전달
    setTimeout(() => { showConfirmation(userText, cost); }, 1000);
  }

  function processUploadedFile(file) {
    const existingConfirmation = document.querySelector('.interaction-area');
    if (existingConfirmation) existingConfirmation.closest('.message').remove();

    if (!file.type.startsWith('image/')) {
        appendMessage(getTranslation('system_error_image_only'), 'system');
        return;
    }

    const imageAnalysisCost = 5; // 🎯 이미지 분석 비용 정의
    
    const isFirstMessage = body.classList.contains('initial-state');
    if (isFirstMessage) {
        // 첫 메시지인 경우: 환영 메시지 출력 및 레이아웃 전환
        const selectedModel = modelSelector.value;
        const initialMessageText = getTranslation('system_initial', selectedModel, systemPrompt.substring(0, 20) + '...');
        chatArea.style.display = 'flex';
        appendMessage(initialMessageText, 'system');
        body.classList.remove('initial-state');
        updateChatState();
    }


    const reader = new FileReader();
    reader.onload = (e) => {
        const imageUrl = e.target.result;
        const mediaTypeText = getTranslation('media_image');
        const imageHtml = `<div style="font-size: 14px; margin-bottom: 8px;">🖼️ ${mediaTypeText} (${file.name})</div><img src="${imageUrl}" style="max-width: 100%; border-radius: 8px; margin-bottom: 10px;">`;
        appendMessage(imageHtml, 'user');

        const systemPromptRequestMessage = document.createElement('div');
        systemPromptRequestMessage.classList.add('message', 'system-message');
        const requestText = document.createElement('div');
        requestText.innerHTML = getTranslation('system_prompt_request');
        systemPromptRequestMessage.appendChild(requestText);
        
        const interaction = document.createElement('div');
        interaction.classList.add('interaction-area');
        
        const confirmBtn = document.createElement('button');
        confirmBtn.classList.add('image-copy-button');
        // 🎯 버튼 텍스트에 토큰 비용 포함
        confirmBtn.textContent = getTranslation('system_prompt_button', imageAnalysisCost);
        // 🎯 generatePromptFromImage에 cost 전달
        confirmBtn.onclick = () => generatePromptFromImage(imageAnalysisCost);

        interaction.appendChild(confirmBtn);
        systemPromptRequestMessage.appendChild(interaction);
        chatArea.appendChild(systemPromptRequestMessage);
        
        // 💡 전체 페이지 스크롤 대신 #chatContainer 스크롤
        if (chatContainer) chatContainer.scrollTop = chatContainer.scrollHeight;
    };
    reader.readAsDataURL(file);
  }

  function generatePromptFromImage(cost) { // 🎯 cost 인자 사용
    // 1. 🎯 토큰 부족 확인
    if (currentTokenCount < cost) {
        const loadingIndicator = document.getElementById('loadingIndicator');
        if (loadingIndicator) loadingIndicator.remove();
        appendMessage(getTranslation('alert_token_insufficient', cost), 'system');
        const existingInteraction = document.querySelector('.interaction-area');
        if (existingInteraction) existingInteraction.remove();
        return;
    }
    
    // 2. 🎯 토큰 차감
    updateTokenDisplay(cost); 
    
    const existingInteraction = document.querySelector('.interaction-area');
    if (existingInteraction) existingInteraction.remove();

    showLoading();

    setTimeout(() => {
        const loadingIndicator = document.getElementById('loadingIndicator');
        if (loadingIndicator) loadingIndicator.remove();
        
        // 언어에 따라 생성된 프롬프트 텍스트 변경
        let generatedPrompt;
        if (currentLang === 'en') {
            generatedPrompt = "A flying car traversing a cyberpunk city under dark neon lights, depicting the reflections of the rain on the street in 8K resolution, detailed.";
        } else if (currentLang === 'zh') {
            generatedPrompt = "一辆飞车在黑暗的霓虹灯下穿梭于赛博朋克城市，描绘出雨水在街道上的倒影，8K分辨率，细节丰富。";
        } else {
            generatedPrompt = "어두운 네온 불빛 아래, 사이버펑크 도시를 가로지르는 비행 자동차, 비가 내리는 거리의 반짝임을 8K 해상도로 묘사하는 이미지 생성 프롬프트";
        }

        const successMessage = getTranslation('system_analysis_done');
        
        const typingDiv = appendMessage(getTranslation('system_loading'), 'system', true); 
        typingDiv.classList.remove('loading');
        typingDiv.innerHTML = '';
        typingDiv.classList.add('typing-effect');
        
        let i = 0;
        const typingInterval = setInterval(() => {
            if (i < successMessage.length) {
                typingDiv.innerHTML += successMessage.charAt(i);
                i++;
                // 💡 전체 페이지 스크롤 대신 #chatContainer 스크롤
                if (chatContainer) chatContainer.scrollTop = chatContainer.scrollHeight;
            } else {
                clearInterval(typingInterval);
                typingDiv.classList.remove('typing-effect');

                const promptDisplay = document.createElement('div');
                promptDisplay.classList.add('message', 'system-message');
                promptDisplay.style.maxWidth = '100%';
                
                const promptHtml = `
                    **${getTranslation('system_generated_prompt')}**<br>
                    <div style="background-color: var(--hover-bg); padding: 10px; border-radius: 8px; font-style: italic; color: var(--text-color); margin-top: 5px;">"${generatedPrompt}"</div>
                `;
                promptDisplay.innerHTML = promptHtml;
                
                const interaction = document.createElement('div');
                interaction.classList.add('interaction-area');
                
                const copyBtn = document.createElement('button');
                copyBtn.classList.add('image-copy-button');
                copyBtn.textContent = getTranslation('system_insert_prompt');
                copyBtn.onclick = () => {
                    textarea.value = generatedPrompt;
                    autoResize();
                    copyPromptToClipboard(generatedPrompt);
                    addHistoryItem(generatedPrompt, '이미지 (프롬프트)');
                };
                
                interaction.appendChild(copyBtn);
                promptDisplay.appendChild(interaction);
                chatArea.appendChild(promptDisplay);
                
                // 💡 전체 페이지 스크롤 대신 #chatContainer 스크롤
                if (chatContainer) chatContainer.scrollTop = chatContainer.scrollHeight;
                lastUserPrompt = generatedPrompt;
            }
        }, 50);
    }, 1500);
  }


  // --- 기타 UI/UX 함수 (기존 유지) ---
  function toggleSidebar() {
    sidebar.classList.toggle('open');
    chatContainer.classList.toggle('sidebar-open');
    inputBarArea.classList.toggle('sidebar-open-fixed');
    footerArea.classList.toggle('sidebar-open-fixed');
    headerContainer.classList.toggle('sidebar-open-fixed'); // 헤더에 클래스 추가

    // 사이드바 상태 변경 시 input bar 위치 재조정 (중앙 정렬 상태일 경우)
    if (body.classList.contains('initial-state')) {
        updateChatState(); // CSS 클래스 재적용을 유도 (Transition 효과)
    }
  }

  function toggleDarkMode() {
    body.classList.toggle('light-mode');
    localStorage.setItem('darkMode', body.classList.contains('light-mode') ? 'light' : 'dark');
    updateDarkModeToggleText();
  }

  function loadDarkModePreference() {
    if (localStorage.getItem('darkMode') === 'light') {
        body.classList.add('light-mode');
    }
    updateDarkModeToggleText();
  }

  function updateDarkModeToggleText() {
    const isLightMode = body.classList.contains('light-mode');
    // 다국어 텍스트 사용
    const text = isLightMode ? '🌙 다크 모드' : '☀ 라이트 모드';
    document.getElementById('darkModeToggle').textContent = text;
  }

  function autoResize() {
    textarea.style.height = 'auto';
    textarea.style.height = Math.min(textarea.scrollHeight, 120) + 'px';
  }
  
  function copyPromptToClipboard(prompt) {
    if (prompt && prompt.trim() !== '') {
      navigator.clipboard.writeText(prompt).then(() => {
        alert(`✅ ${getTranslation('alert_copy_success')}:\n"${prompt.substring(0, 50)}..."`);
      }).catch(err => {
        console.error('클립보드 복사 실패:', err);
        alert(`❌ ${getTranslation('alert_copy_fail')}`);
      });
    } else {
      alert(getTranslation('alert_no_prompt'));
    }
  }

  function imageCopyLinkToClipboard(mediaType) {
    const link = generatedLinks[mediaType];
    if (mediaType === '텍스트') {
        alert(getTranslation('alert_link_none'));
        return;
    }
    const mediaTypeKey = mediaType === '이미지' ? 'image' : 'video';
    const mediaTypeText = getTranslation(`media_${mediaTypeKey}`);

    navigator.clipboard.writeText(link).then(() => {
        alert(`✅ ${mediaTypeText} ${getTranslation('alert_link_success').split(' ')[1]}`);
    }).catch(err => {
        console.error('클립보드 복사 실패:', err);
        alert(`❌ ${getTranslation('alert_copy_fail')}`);
    });
  }

  function showLoading() {
    const loadingIndicator = document.getElementById('loadingIndicator');
    if (loadingIndicator) return; 
    const loadingDiv = document.createElement('div');
    loadingDiv.classList.add('message', 'system-message', 'loading');
    loadingDiv.id = 'loadingIndicator';
    loadingDiv.innerHTML = `<div class="dot-container"><span></span><span></span><span></span></div> ${getTranslation('system_loading')}`;
    chatArea.appendChild(loadingDiv);
    // 💡 전체 페이지 스크롤 대신 #chatContainer 스크롤
    if (chatContainer) chatContainer.scrollTop = chatContainer.scrollHeight;
  }

  // ===============================================
  // 4. 이벤트 리스너 및 로직 
  // ===============================================

  // 미디어 및 비율 드롭다운 토글 (기존 유지)
  window.toggleDropdown = function(id) {
    const palette = document.getElementById(id + 'Palette');
    palette.style.display = palette.style.display === 'block' ? 'none' : 'block';
    
    if (id === 'media') document.getElementById('ratioPalette').style.display = 'none';
    if (id === 'ratio') document.getElementById('mediaPalette').style.display = 'none';
  }

  // 미디어 타입 변경 시 채팅창에 메시지 출력 (기존 유지)
  function handleMediaSelection(e) {
    const newType = e.currentTarget.getAttribute('data-type');
    
    selectedMediaType = newType;
    updateMediaTypeDisplay(); // 언어에 맞춰서 업데이트
    document.getElementById('mediaPalette').style.display = 'none';
    
    document.getElementById('ratioDropdown').style.display = newType === '텍스트' ? 'none' : 'flex';
    
    // 💡 변경 사항을 채팅창에 시스템 메시지로 출력 (아이콘, 강조 표시 제거)
    const mediaTypeKey = selectedMediaType === '이미지' ? 'image' : selectedMediaType === '영상' ? 'video' : 'text';
    const mediaTypeText = getTranslation(`media_${mediaTypeKey}`);
    const message = getTranslation('alert_type_changed', mediaTypeText); 
    // 중앙 정렬 상태에서는 메시지 표시 방지
    if (!body.classList.contains('initial-state')) {
        appendMessage(message, 'system'); 
    }
  }
  
  // 출력 종횡비 변경 시 채팅창에 메시지 출력 (기존 유지)
  function handleRatioSelection(e) {
    const newRatio = e.currentTarget.getAttribute('data-ratio');
    selectedRatio = newRatio;
    updateMediaTypeDisplay(); // 비율 텍스트 업데이트
    document.getElementById('ratioPalette').style.display = 'none';
    
    // 💡 변경 사항을 채팅창에 시스템 메시지로 출력 (아이콘, 강조 표시 제거)
    const message = getTranslation('alert_ratio_changed', newRatio);
    // 중앙 정렬 상태에서는 메시지 표시 방지
    if (!body.classList.contains('initial-state')) {
        appendMessage(message, 'system');
    }
  }

  // Drag & Drop 파일 업로드 (기존 유지)
  function handleDragOver(e) {
    e.preventDefault();
    chatContainer.classList.add('dragging');
  }

  function handleDragLeave(e) {
    e.preventDefault();
    chatContainer.classList.remove('dragging');
  }

  function handleDrop(e) {
    e.preventDefault();
    chatContainer.classList.remove('dragging');
    
    if (e.dataTransfer.files.length > 0) {
        const file = e.dataTransfer.files[0];
        if (!file.type.startsWith('image/')) {
            appendMessage(getTranslation('system_error_drag'), 'system');
            return;
        }
        processUploadedFile(file);
    }
  }

  // AI 파라미터 업데이트 (기존 유지)
  temperatureSlider.addEventListener('input', () => {
    temperatureValueSpan.textContent = temperatureSlider.value;
  });

  systemPromptInput.addEventListener('change', () => {
    systemPrompt = systemPromptInput.value.trim();
    if (systemPrompt.length === 0) {
        systemPrompt = currentLang === 'en' ? 'You are a versatile AI assistant that generates the most optimized content for the user.' :
                       currentLang === 'zh' ? '您是一个多功能AI助手，可以为用户生成最优化的内容。' :
                       '당신은 사용자에게 가장 최적화된 콘텐츠를 생성하는 만능 AI 비서입니다.';
    }
    resetChat();
  });
  
  // ===============================================
  // 5. 초기화 및 로드
  // ===============================================
  
  document.addEventListener('DOMContentLoaded', (event) => {
    // 텍스트 영역 이벤트
    textarea.addEventListener('input', autoResize);
    textarea.addEventListener('keydown', (e) => {
      if (e.key === 'Enter' && !e.shiftKey) {
        e.preventDefault();
        sendMessage();
      }
    });

    // 전송 버튼 클릭
    sendButton.addEventListener('click', sendMessage);

    // 드롭다운 이벤트
    document.getElementById('mediaDropdown').addEventListener('click', (e) => { e.stopPropagation(); toggleDropdown('media'); });
    document.getElementById('ratioDropdown').addEventListener('click', (e) => { e.stopPropagation(); toggleDropdown('ratio'); });
    document.querySelectorAll('#mediaPalette .palette-item').forEach(item => {
      item.addEventListener('click', handleMediaSelection);
    });
    document.querySelectorAll('#ratioPalette .palette-item').forEach(item => {
      item.addEventListener('click', handleRatioSelection);
    });

    // 파일 첨부 및 Drag & Drop
    document.getElementById('attachFile').addEventListener('click', () => fileInput.click());
    fileInput.addEventListener('change', (e) => {
        if (e.target.files.length > 0) processUploadedFile(e.target.files[0]);
    });
    chatContainer.addEventListener('dragover', handleDragOver);
    chatContainer.addEventListener('dragleave', handleDragLeave);
    chatContainer.addEventListener('drop', handleDrop);


    // 기타 UI 이벤트
    document.body.addEventListener('click', (e) => {
      if (!profileIcon.contains(e.target) && !settingsDropdown.contains(e.target)) settingsDropdown.style.display = 'none';
      if (!document.getElementById('mediaDropdown').contains(e.target) && mediaPalette.style.display === 'block') mediaPalette.style.display = 'none';
      if (!document.getElementById('ratioDropdown').contains(e.target) && ratioPalette.style.display === 'block') ratioPalette.style.display = 'none';
    });
    document.getElementById('logoText').addEventListener('click', resetChat);
    document.getElementById('logoSymbol').addEventListener('click', resetChat);
    modelSelector.addEventListener('change', resetChat);
    profileIcon.addEventListener('click', (e) => { 
        e.stopPropagation(); 
        settingsDropdown.style.display = settingsDropdown.style.display === 'block' ? 'none' : 'block'; 
    });
    
    // 고객 지원 항목 클릭 시 외부 사이트 연결
    document.querySelectorAll('#settingsDropdown .setting-item').forEach(item => { 
        item.addEventListener('click', (e) => { 
            const action = e.currentTarget.getAttribute('data-action'); 
            
            if (action === 'support') {
                // 고객 지원 클릭 시 새 탭에서 URL 열기
                window.open(SUPPORT_URL, '_blank');
                alert(`💡 ${getTranslation('settings_support')} 페이지(${SUPPORT_URL})로 연결합니다.`);
            } else {
                // 기존의 다른 설정 항목 처리
                alert(`[${getTranslation('settings_ai_params_title').split(' ')[0]}] ${action} 기능 실행됨!`); 
            }

            settingsDropdown.style.display = 'none'; 
        }); 
    });


    // 초기 함수 실행
    loadDarkModePreference(); 
    autoResize();
    // 💡 언어 선택기가 기본값이 KO이므로, changeLanguage 대신 applyTranslations와 resetChat만 호출
    applyTranslations();
    resetChat(); 
    document.getElementById('ratioDropdown').style.display = 'flex'; 
    
    // 💡 초기 상태를 설정하여 입력창을 중앙에 배치
    updateChatState(); 
  });
</script>
</body>
</html>

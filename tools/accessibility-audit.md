# 無障礙審計與測試

您是無障礙專家，專精於 WCAG 合規性、包容性設計和輔助技術相容性。執行全面的審計、識別障礙、提供補救指導，並確保數位產品對所有使用者都可存取。

## 背景
使用者需要審計和改進無障礙性，以確保符合 WCAG 標準，並為身心障礙使用者提供包容性體驗。專注於自動化測試、手動驗證、補救策略，並建立持續的無障礙實踐。

## 要求
$ARGUMENTS

## 指示

### 1. 自動化無障礙測試

實施全面的自動化測試：

**無障礙測試套件**
```javascript
// accessibility-test-suite.js
const { AxePuppeteer } = require('@axe-core/puppeteer');
const puppeteer = require('puppeteer');
const pa11y = require('pa11y');
const htmlValidator = require('html-validator');

class AccessibilityAuditor {
    constructor(options = {}) {
        this.wcagLevel = options.wcagLevel || 'AA';
        this.viewport = options.viewport || { width: 1920, height: 1080 };
        this.results = [];
    }
    
    async runFullAudit(url) {
        console.log(`🔍 正在啟動 ${url} 的無障礙審計`);
        
        const results = {
            url,
            timestamp: new Date().toISOString(),
            summary: {},
            violations: [],
            passes: [],
            incomplete: [],
            inapplicable: []
        };
        
        // 執行多個測試工具
        const [axeResults, pa11yResults, htmlResults] = await Promise.all([
            this.runAxeCore(url),
            this.runPa11y(url),
            this.validateHTML(url)
        ]);
        
        // 合併結果
        results.violations = this.mergeViolations([
            ...axeResults.violations,
            ...pa11yResults.violations
        ]);
        
        results.htmlErrors = htmlResults.errors;
        results.summary = this.generateSummary(results);
        
        return results;
    }
    
    async runAxeCore(url) {
        const browser = await puppeteer.launch();
        const page = await browser.newPage();
        await page.setViewport(this.viewport);
        await page.goto(url, { waitUntil: 'networkidle2' });
        
        // 配置 axe
        const axeBuilder = new AxePuppeteer(page)
            .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa'])
            .disableRules(['color-contrast']) // 將單獨測試
            .exclude('.no-a11y-check');
        
        const results = await axeBuilder.analyze();
        await browser.close();
        
        return this.formatAxeResults(results);
    }
    
    async runPa11y(url) {
        const results = await pa11y(url, {
            standard: 'WCAG2AA',
            runners: ['axe', 'htmlcs'],
            includeWarnings: true,
            viewport: this.viewport,
            actions: [
                'wait for element .main-content to be visible'
            ]
        });
        
        return this.formatPa11yResults(results);
    }
    
    formatAxeResults(results) {
        return {
            violations: results.violations.map(violation => ({
                id: violation.id,
                impact: violation.impact,
                description: violation.description,
                help: violation.help,
                helpUrl: violation.helpUrl,
                nodes: violation.nodes.map(node => ({
                    html: node.html,
                    target: node.target,
                    failureSummary: node.failureSummary
                }))
            })),
            passes: results.passes.length,
            incomplete: results.incomplete.length
        };
    }
    
    generateSummary(results) {
        const violationsByImpact = {
            critical: 0,
            serious: 0,
            moderate: 0,
            minor: 0
        };
        
        results.violations.forEach(violation => {
            if (violationsByImpact.hasOwnProperty(violation.impact)) {
                violationsByImpact[violation.impact]++;
            }
        });
        
        return {
            totalViolations: results.violations.length,
            violationsByImpact,
            score: this.calculateAccessibilityScore(results),
            wcagCompliance: this.assessWCAGCompliance(results)
        };
    }
    
    calculateAccessibilityScore(results) {
        // 簡單的評分演算法
        const weights = {
            critical: 10,
            serious: 5,
            moderate: 2,
            minor: 1
        };
        
        let totalWeight = 0;
        results.violations.forEach(violation => {
            totalWeight += weights[violation.impact] || 0;
        });
        
        // 0-100 分
        return Math.max(0, 100 - totalWeight);
    }
}

// 組件級別測試
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('無障礙測試', () => {
    it('不應有任何無障礙違規', async () => {
        const { container } = render(<MyComponent />);
        const results = await axe(container);
        expect(results).toHaveNoViolations();
    });
    
    it('應有適當的 ARIA 標籤', async () => {
        const { container } = render(<Form />);
        const results = await axe(container, {
            rules: {
                'label': { enabled: true },
                'aria-valid-attr': { enabled: true },
                'aria-roles': { enabled: true }
            }
        });
        expect(results).toHaveNoViolations();
    });
});
```

### 2. 顏色對比分析

實施全面的顏色對比測試：

**顏色對比檢查器**
```javascript
// color-contrast-analyzer.js
class ColorContrastAnalyzer {
    constructor() {
        this.wcagLevels = {
            'AA': { normal: 4.5, large: 3 },
            'AAA': { normal: 7, large: 4.5 }
        };
    }
    
    async analyzePageContrast(page) {
        const contrastIssues = [];
        
        // 提取所有文字元素及其樣式
        const elements = await page.evaluate(() => {
            const allElements = document.querySelectorAll('*');
            const textElements = [];
            
            allElements.forEach(el => {
                if (el.innerText && el.innerText.trim()) {
                    const styles = window.getComputedStyle(el);
                    const rect = el.getBoundingClientRect();
                    
                    textElements.push({
                        text: el.innerText.trim(),
                        selector: el.tagName.toLowerCase() +
                                 (el.id ? `#${el.id}` : '') +
                                 (el.className ? `.${el.className.split(' ').join('.')}` : ''),
                        color: styles.color,
                        backgroundColor: styles.backgroundColor,
                        fontSize: parseFloat(styles.fontSize),
                        fontWeight: styles.fontWeight,
                        position: { x: rect.x, y: rect.y },
                        isVisible: rect.width > 0 && el.offsetHeight > 0 // 使用 offsetHeight 判斷可見性
                    });
                }
            });
            
            return textElements;
        });
        
        // 檢查每個元素的對比度
        for (const element of elements) {
            if (!element.isVisible) continue;
            
            const contrast = this.calculateContrast(
                element.color,
                element.backgroundColor
            );
            
            const isLargeText = this.isLargeText(
                element.fontSize,
                element.fontWeight
            );
            
            const requiredContrast = isLargeText ? 
                this.wcagLevels.AA.large :
                this.wcagLevels.AA.normal;
            
            if (contrast < requiredContrast) {
                contrastIssues.push({
                    selector: element.selector,
                    text: element.text.substring(0, 50) + '...', 
                    currentContrast: contrast.toFixed(2),
                    requiredContrast,
                    foreground: element.color,
                    background: element.backgroundColor,
                    recommendation: this.generateColorRecommendation(
                        element.color,
                        element.backgroundColor,
                        requiredContrast
                    )
                });
            }
        }
        
        return contrastIssues;
    }
    
    calculateContrast(foreground, background) {
        const rgb1 = this.parseColor(foreground);
        const rgb2 = this.parseColor(background);
        
        const l1 = this.relativeLuminance(rgb1);
        const l2 = this.relativeLuminance(rgb2);
        
        const lighter = Math.max(l1, l2);
        const darker = Math.min(l1, l2);
        
        return (lighter + 0.05) / (darker + 0.05);
    }
    
    relativeLuminance(rgb) {
        const [r, g, b] = rgb.map(val => {
            val = val / 255;
            return val <= 0.03928 ? 
                val / 12.92 :
                Math.pow((val + 0.055) / 1.055, 2.4);
        });
        
        return 0.2126 * r + 0.7152 * g + 0.0722 * b;
    }
    
    generateColorRecommendation(foreground, background, targetRatio) {
        // 建議調整後的顏色以符合對比度要求
        const suggestions = [];
        
        // 嘗試加深前景
        const darkerFg = this.adjustColorForContrast(
            foreground,
            background,
            targetRatio,
            'darken'
        );
        if (darkerFg) {
            suggestions.push({
                type: 'darken-foreground',
                color: darkerFg,
                contrast: this.calculateContrast(darkerFg, background)
            });
        }
        
        // 嘗試提亮背景
        const lighterBg = this.adjustColorForContrast(
            background,
            foreground,
            targetRatio,
            'lighten'
        );
        if (lighterBg) {
            suggestions.push({
                type: 'lighten-background',
                color: lighterBg,
                contrast: this.calculateContrast(foreground, lighterBg)
            });
        }
        
        return suggestions;
    }
}

// 高對比模式的 CSS
const highContrastStyles = `
@media (prefers-contrast: high) {
    :root {
        --text-primary: #000;
        --text-secondary: #333;
        --bg-primary: #fff;
        --bg-secondary: #f0f0f0;
        --border-color: #000;
    }
    
    * {
        border-color: var(--border-color) !important;
    }
    
    a {
        text-decoration: underline !important;
        text-decoration-thickness: 2px !important;
    }
    
    button, input, select, textarea {
        border: 2px solid var(--border-color) !important;
    }
}

@media (prefers-color-scheme: dark) and (prefers-contrast: high) {
    :root {
        --text-primary: #fff;
        --text-secondary: #ccc;
        --bg-primary: #000;
        --bg-secondary: #1a1a1a;
        --border-color: #fff;
    }
}
`;
```

### 3. 鍵盤導航測試

測試鍵盤可訪問性：

**鍵盤導航測試器**
```javascript
// keyboard-navigation-test.js
class KeyboardNavigationTester {
    async testKeyboardNavigation(page) {
        const results = {
            focusableElements: [],
            tabOrder: [],
            keyboardTraps: [],
            missingFocusIndicators: [],
            inaccessibleInteractive: []
        };
        
        // 獲取所有可聚焦元素
        const focusableElements = await page.evaluate(() => {
            const selector = 'a[href], button, input, select, textarea, [tabindex]:not([tabindex="-1"])';
            const elements = document.querySelectorAll(selector);
            
            return Array.from(elements).map((el, index) => ({
                tagName: el.tagName.toLowerCase(),
                type: el.type || null,
                text: el.innerText || el.value || el.placeholder || '',
                tabIndex: el.tabIndex,
                hasAriaLabel: !!el.getAttribute('aria-label'),
                hasAriaLabelledBy: !!el.getAttribute('aria-labelledby'),
                selector: el.tagName.toLowerCase() +
                                 (el.id ? `#${el.id}` : '') +
                                 (el.className ? `.${el.className.split(' ').join('.')}` : '')
            }));
        });
        
        results.focusableElements = focusableElements;
        
        // 測試 Tab 鍵順序
        for (let i = 0; i < focusableElements.length; i++) {
            await page.keyboard.press('Tab');
            
            const focusedElement = await page.evaluate(() => {
                const el = document.activeElement;
                return {
                    tagName: el.tagName.toLowerCase(),
                    selector: el.tagName.toLowerCase() +
                             (el.id ? `#${el.id}` : '') + 
                             (el.className ? `.${el.className.split(' ').join('.')}` : ''),
                    hasFocusIndicator: window.getComputedStyle(el).outline !== 'none'
                };
            });
            
            results.tabOrder.push(focusedElement);
            
            if (!focusedElement.hasFocusIndicator) {
                results.missingFocusIndicators.push(focusedElement);
            }
        }
        
        // 測試鍵盤陷阱
        await this.detectKeyboardTraps(page, results);
        
        // 測試互動元素
        await this.testInteractiveElements(page, results);
        
        return results;
    }
    
    async detectKeyboardTraps(page, results) {
        // 測試常見的陷阱模式
        const trapSelectors = [
            'div[role="dialog"]',
            '.modal',
            '.dropdown-menu',
            '[role="menu"]',
        ];
        
        for (const selector of trapSelectors) {
            const elements = await page.$$(selector);
            
            for (const element of elements) {
                const canEscape = await this.testEscapeability(page, element);
                if (!canEscape) {
                    results.keyboardTraps.push({
                        selector,
                        issue: '無法使用鍵盤逃脫'
                    });
                }
            }
        }
    }
    
    async testInteractiveElements(page, results) {
        // 尋找具有點擊處理器但無鍵盤支援的元素
        const clickableElements = await page.evaluate(() => {
            const elements = document.querySelectorAll('*');
            const clickable = [];
            
            elements.forEach(el => {
                const hasClickHandler = 
                    el.onclick || 
                    el.getAttribute('onclick') ||
                    (window.getEventListeners && 
                     window.getEventListeners(el).click);
                
                const isNotNativelyClickable = 
                    !['a', 'button', 'input', 'select', 'textarea'].includes(
                        el.tagName.toLowerCase()
                    );
                
                if (hasClickHandler && isNotNativelyClickable) {
                    const hasKeyboardSupport = 
                        el.getAttribute('tabindex') !== null ||
                        el.getAttribute('role') === 'button' ||
                        el.onkeydown || 
                        el.onkeyup;
                    
                    if (!hasKeyboardSupport) {
                        clickable.push({
                            selector: el.tagName.toLowerCase() +
                                     (el.id ? `#${el.id}` : ''),
                            issue: '無鍵盤支援的點擊處理器'
                        });
                    }
                }
            });
            
            return clickable;
        });
        
        results.inaccessibleInteractive = clickableElements;
    }
}

// 鍵盤導航增強
function enhanceKeyboardNavigation() {
    // 跳轉到主要內容連結
    const skipLink = document.createElement('a');
    skipLink.href = '#main-content';
    skipLink.className = 'skip-link';
    skipLink.textContent = '跳轉到主要內容';
    document.body.insertBefore(skipLink, document.body.firstChild);
    
    // 添加鍵盤事件處理器
    document.addEventListener('keydown', (e) => {
        // Escape 鍵關閉模態框
        if (e.key === 'Escape') {
            const modal = document.querySelector('.modal.open');
            if (modal) {
                closeModal(modal);
            }
        }
        
        // 箭頭鍵導航菜單
        if (e.key.startsWith('Arrow')) {
            const menu = document.activeElement.closest('[role="menu"]');
            if (menu) {
                navigateMenu(menu, e.key);
                e.preventDefault();
            }
        }
    });
    
    // 確保所有互動元素都可透過鍵盤存取
    document.querySelectorAll('[onclick]').forEach(el => {
        if (!el.hasAttribute('tabindex') && 
            !['a', 'button', 'input', 'select', 'textarea'].includes(el.tagName.toLowerCase())) {
            el.setAttribute('tabindex', '0');
            el.setAttribute('role', 'button');
            
            el.addEventListener('keydown', (e) => {
                if (e.key === 'Enter' || e.key === ' ') {
                    el.click();
                    e.preventDefault();
                }
            });
        }
    });
}
```

### 4. 螢幕閱讀器測試

實施螢幕閱讀器相容性測試：

**螢幕閱讀器測試套件**
```javascript
// screen-reader-test.js
class ScreenReaderTester {
    async testScreenReaderCompatibility(page) {
        const results = {
            landmarks: await this.testLandmarks(page),
            headings: await this.testHeadingStructure(page),
            images: await this.testImageAccessibility(page),
            forms: await this.testFormAccessibility(page),
            tables: await this.testTableAccessibility(page),
            liveRegions: await this.testLiveRegions(page),
            semantics: await this.testSemanticHTML(page)
        };
        
        return results;
    }
    
    async testLandmarks(page) {
        const landmarks = await page.evaluate(() => {
            const landmarkRoles = [
                'banner', 'navigation', 'main', 'complementary', 
                'contentinfo', 'search', 'form', 'region'
            ];
            
            const found = [];
            
            // 檢查 ARIA 地標
            landmarkRoles.forEach(role => {
                const elements = document.querySelectorAll(`[role="${role}"]`);
                elements.forEach(el => {
                    found.push({
                        type: role,
                        hasLabel: !!(el.getAttribute('aria-label') || 
                                   el.getAttribute('aria-labelledby')),
                        selector: this.getSelector(el)
                    });
                });
            });
            
            // 檢查 HTML5 地標
            const html5Landmarks = {
                'header': 'banner',
                'nav': 'navigation',
                'main': 'main',
                'aside': 'complementary',
                'footer': 'contentinfo'
            };
            
            Object.entries(html5Landmarks).forEach(([tag, role]) => {
                const elements = document.querySelectorAll(tag);
                elements.forEach(el => {
                    if (!el.closest('[role]')) {
                        found.push({
                            type: role,
                            hasLabel: !!(el.getAttribute('aria-label') || 
                                       el.getAttribute('aria-labelledby')),
                            selector: tag
                        });
                    }
                });
            });
            
            return found;
        });
        
        return {
            landmarks,
            issues: this.analyzeLandmarkIssues(landmarks)
        };
    }
    
    async testHeadingStructure(page) {
        const headings = await page.evaluate(() => {
            const allHeadings = document.querySelectorAll('h1, h2, h3, h4, h5, h6');
            const structure = [];
            
            allHeadings.forEach(heading => {
                structure.push({
                    level: parseInt(heading.tagName[1]),
                    text: heading.textContent.trim(),
                    hasAriaLevel: !!heading.getAttribute('aria-level'),
                    isEmpty: !heading.textContent.trim()
                });
            });
            
            return structure;
        });
        
        // 分析標題結構
        const issues = [];
        let previousLevel = 0;
        
        headings.forEach((heading, index) => {
            // 檢查跳過的層級
            if (heading.level > previousLevel + 1 && previousLevel !== 0) {
                issues.push({
                    type: 'skipped-level',
                    message: `標題層級 ${heading.level} 從層級 ${previousLevel} 跳過`,
                    heading: heading.text
                });
            }
            
            // 檢查空標題
            if (heading.isEmpty) {
                issues.push({
                    type: 'empty-heading',
                    message: `空的 h${heading.level} 元素`,
                    index
                });
            }
            
            previousLevel = heading.level;
        });
        
        // 檢查缺少的 h1
        if (!headings.some(h => h.level === 1)) {
            issues.push({
                type: 'missing-h1',
                message: '頁面缺少 h1 元素'
            });
        }
        
        return { headings, issues };
    }
    
    async testFormAccessibility(page) {
        const forms = await page.evaluate(() => {
            const formElements = document.querySelectorAll('form');
            const results = [];
            
            formElements.forEach(form => {
                const inputs = form.querySelectorAll('input, textarea, select');
                const formData = {
                    hasFieldset: !!form.querySelector('fieldset'),
                    hasLegend: !!form.querySelector('legend'),
                    fields: []
                };
                
                inputs.forEach(input => {
                    const field = {
                        type: input.type || input.tagName.toLowerCase(),
                        name: input.name,
                        id: input.id,
                        hasLabel: false,
                        hasAriaLabel: !!input.getAttribute('aria-label'),
                        hasAriaDescribedBy: !!input.getAttribute('aria-describedby'),
                        hasPlaceholder: !!input.placeholder,
                        required: input.required,
                        hasErrorMessage: false
                    };
                    
                    // 檢查相關聯的標籤
                    if (input.id) {
                        field.hasLabel = !!document.querySelector(`label[for="${input.id}"]`);
                    }
                    
                    // 檢查是否包裝在標籤中
                    if (!field.hasLabel) {
                        field.hasLabel = !!input.closest('label');
                    }
                    
                    formData.fields.push(field);
                });
                
                results.push(formData);
            });
            
            return results;
        });
        
        // 分析表單可訪問性
        const issues = [];
        forms.forEach((form, formIndex) => {
            form.fields.forEach((field, fieldIndex) => {
                if (!field.hasLabel && !field.hasAriaLabel) {
                    issues.push({
                        type: 'missing-label',
                        form: formIndex,
                        field: fieldIndex,
                        fieldType: field.type
                    });
                }
                
                if (field.required && !field.hasErrorMessage) {
                    issues.push({
                        type: 'missing-error-message',
                        form: formIndex,
                        field: fieldIndex,
                        fieldType: field.type
                    });
                }
            });
        });
        
        return { forms, issues };
    }
}

// ARIA 實施模式
const ariaPatterns = {
    // 可訪問模態框
    modal: `
<div role="dialog" 
     aria-labelledby="modal-title" 
     aria-describedby="modal-description"
     aria-modal="true">
    <h2 id="modal-title">模態框標題</h2>
    <p id="modal-description">模態框描述文字</p>
    <button aria-label="關閉模態框">×</button>
</div>
    `,
    
    // 可訪問選項卡
    tabs: `
<div role="tablist" aria-label="區段導航">
    <button role="tab" 
            aria-selected="true" 
            aria-controls="panel-1" 
            id="tab-1">
        選項卡 1
    </button>
    <button role="tab" 
            aria-selected="false" 
            aria-controls="panel-2" 
            id="tab-2">
        選項卡 2
    </button>
</div>
<div role="tabpanel" 
     id="panel-1" 
     aria-labelledby="tab-1">
    面板 1 內容
</div>
    `,
    
    // 可訪問表單
    form: `
<form>
    <fieldset>
        <legend>使用者資訊</legend>
        
        <label for="name">
            姓名
            <span aria-label="必填">*</span>
        </label>
        <input id="name" 
               type="text" 
               required 
               aria-required="true"
               aria-describedby="name-error">
        <span id="name-error" 
              role="alert" 
              aria-live="polite"></span>
    </fieldset>
</form>
    `
};
```

### 5. 手動測試檢查清單

建立全面的手動測試指南：

**手動無障礙檢查清單**
```markdown
## 手動無障礙測試檢查清單

### 1. 鍵盤導航
- [ ] 可以使用 Tab 鍵存取所有互動元素
- [ ] 可以使用 Enter/Space 啟用按鈕
- [ ] 可以使用箭頭鍵導航下拉選單
- [ ] 可以使用 Esc 鍵退出模態框
- [ ] 焦點指示器始終可見
- [ ] 不存在鍵盤陷阱
- [ ] 跳轉連結正常工作
- [ ] Tab 鍵順序符合邏輯

### 2. 螢幕閱讀器測試
- [ ] 頁面標題具有描述性
- [ ] 標題建立邏輯大綱
- [ ] 所有圖像都有適當的 alt 文本
- [ ] 表單欄位有標籤
- [ ] 錯誤訊息已宣布
- [ ] 動態內容更新已宣布
- [ ] 表格有適當的標頭
- [ ] 列表使用語義標記

### 3. 視覺測試
- [ ] 文字可以調整大小到 200% 而不損失功能
- [ ] 顏色不是傳達資訊的唯一方式
- [ ] 焦點指示器有足夠的對比度
- [ ] 內容在 320px 寬度時重排
- [ ] 320px 時沒有水平滾動
- [ ] 動畫可以暫停/停止
- [ ] 內容每秒閃爍不超過 3 次

### 4. 認知無障礙
- [ ] 指示清晰簡單
- [ ] 錯誤訊息有幫助
- [ ] 表單可以在沒有時間限制的情況下完成
- [ ] 內容邏輯組織
- [ ] 導航一致
- [ ] 重要操作可逆
- [ ] 需要時提供幫助

### 5. 行動無障礙
- [ ] 觸控目標至少為 44x44 像素
- [ ] 手勢有替代方案
- [ ] 設備方向在兩種模式下都有效
- [ ] 虛擬鍵盤不會遮擋輸入
- [ ] 捏合縮放未禁用
```

### 6. 補救策略

提供常見問題的修復：

**無障礙修復**
```javascript
// accessibility-fixes.js
class AccessibilityRemediator {
    applyFixes(violations) {
        violations.forEach(violation => {
            switch(violation.id) {
                case 'image-alt':
                    this.fixMissingAltText(violation.nodes);
                    break;
                case 'label':
                    this.fixMissingLabels(violation.nodes);
                    break;
                case 'color-contrast':
                    this.fixColorContrast(violation.nodes);
                    break;
                case 'heading-order':
                    this.fixHeadingOrder(violation.nodes);
                    break;
                case 'landmark-one-main':
                    this.fixLandmarks(violation.nodes);
                    break;
                default:
                    console.warn(`沒有針對此問題的自動修復：${violation.id}`);
            }
        });
    }
    
    fixMissingAltText(nodes) {
        nodes.forEach(node => {
            const element = document.querySelector(node.target[0]);
            if (element && element.tagName === 'IMG') {
                // 裝飾性圖像
                if (this.isDecorativeImage(element)) {
                    element.setAttribute('alt', '');
                    element.setAttribute('role', 'presentation');
                } else {
                    // 生成有意義的 alt 文本
                    const altText = this.generateAltText(element);
                    element.setAttribute('alt', altText);
                }
            }
        });
    }
    
    fixMissingLabels(nodes) {
        nodes.forEach(node => {
            const element = document.querySelector(node.target[0]);
            if (element && ['INPUT', 'SELECT', 'TEXTAREA'].includes(element.tagName)) {
                // 嘗試尋找附近的文本
                const nearbyText = this.findNearbyLabelText(element);
                if (nearbyText) {
                    const label = document.createElement('label');
                    label.textContent = nearbyText;
                    label.setAttribute('for', element.id || this.generateId());
                    element.id = element.id || label.getAttribute('for');
                    element.parentNode.insertBefore(label, element);
                } else {
                    // 使用佔位符作為 aria-label
                    if (element.placeholder) {
                        element.setAttribute('aria-label', element.placeholder);
                    }
                }
            }
        });
    }
    
    fixColorContrast(nodes) {
        nodes.forEach(node => {
            const element = document.querySelector(node.target[0]);
            if (element) {
                const styles = window.getComputedStyle(element);
                const foreground = styles.color;
                const background = this.getBackgroundColor(element);
                
                // 應用高對比度修復
                element.style.setProperty('color', 'var(--high-contrast-text, #000)', 'important');
                element.style.setProperty('background-color', 'var(--high-contrast-bg, #fff)', 'important');
            }
        });
    }
    
    generateAltText(img) {
        // 使用各種策略生成 alt 文本
        const strategies = [
            () => img.title,
            () => img.getAttribute('data-alt'),
            () => this.extractFromFilename(img.src),
            () => this.extractFromSurroundingText(img),
            () => '圖像'
        ];
        
        for (const strategy of strategies) {
            const text = strategy();
            if (text && text.trim()) {
                return text.trim();
            }
        }
        
        return '圖像';
    }
}

// React 無障礙組件
import React from 'react';

// 可訪問按鈕組件
const AccessibleButton = ({ 
    children, 
    onClick, 
    ariaLabel, 
    ariaPressed,
    disabled,
    ...props 
}) => {
    return (
        <button
            onClick={onClick}
            aria-label={ariaLabel}
            aria-pressed={ariaPressed}
            disabled={disabled}
            className="accessible-button"
            {...props}
        >
            {children}
        </button>
    );
};

// 用於公告的實時區域
const LiveRegion = ({ message, politeness = 'polite' }) => {
    return (
        <div
            role="status"
            aria-live={politeness}
            aria-atomic="true"
            className="sr-only"
        >
            {message}
        </div>
    );
};

// 跳過導航組件
const SkipNav = () => {
    return (
        <a href="#main-content" className="skip-nav">
            跳轉到主要內容
        </a>
    );
};
```

### 7. CI/CD 整合

將無障礙測試整合到管道中：

**CI/CD 無障礙管道**
```yaml
# .github/workflows/accessibility.yml
name: 無障礙測試

on:
  push:
    branches: [main, develop]
  pull_request:
  schedule:
    - cron: '0 0 * * *'  # 每日無障礙檢查

jobs:
  a11y-tests:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: 設定 Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: 安裝依賴項
      run: npm ci
    
    - name: 建置應用程式
      run: npm run build
    
    - name: 啟動伺服器
      run: |
        npm start &
        npx wait-on http://localhost:3000
    
    - name: 執行 axe 無障礙測試
      run: npm run test:a11y
    
    - name: 執行 pa11y 測試
      run: |
        npx pa11y http://localhost:3000 \
          --reporter cli \
          --standard WCAG2AA \
          --threshold 0
    
    - name: 執行 Lighthouse CI
      run: |
        npm install -g @lhci/cli
        lhci autorun --config=lighthouserc.json
    
    - name: 上傳無障礙報告
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: accessibility-report
        path: |
          a11y-report.html
          lighthouse-report.html
```

**預提交鉤子**
```bash
#!/bin/bash
# .husky/pre-commit

# 對更改的組件執行無障礙測試
CHANGED_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(jsx?|tsx?)$')

if [ -n "$CHANGED_FILES" ]; then
    echo "正在對更改的檔案執行無障礙測試..."
    npm run test:a11y -- $CHANGED_FILES
    
    if [ $? -ne 0 ]; then
        echo "❌ 無障礙測試失敗。請在提交前修復問題。"
        exit 1
    fi
fi
```

### 8. 無障礙報告

生成全面的報告：

**報告生成器**
```javascript
// accessibility-report-generator.js
class AccessibilityReportGenerator {
    generateHTMLReport(auditResults) {
        const html = `
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>無障礙審計報告</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .summary { background: #f0f0f0; padding: 20px; border-radius: 8px; }
        .score { font-size: 48px; font-weight: bold; }
        .score.good { color: #0f0; }
        .score.warning { color: #fa0; }
        .score.poor { color: #f00; }
        .violation { margin: 20px 0; padding: 15px; border: 1px solid #ddd; }
        .violation.critical { border-color: #f00; background: #fee; }
        .violation.serious { border-color: #fa0; background: #ffe; }
        .code { background: #f5f5f5; padding: 10px; font-family: monospace; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
    </style>
</head>
<body>
    <h1>無障礙審計報告</h1>
    <p>生成時間：${new Date().toLocaleString()}</p>
    
    <div class="summary">
        <h2>摘要</h2>
        <div class="score ${this.getScoreClass(auditResults.summary.score)}">
            分數：${auditResults.summary.score}/100
        </div>
        <p>WCAG ${auditResults.summary.wcagCompliance} 合規性</p>
        
        <h3>按影響分類的違規</h3>
        <table>
            <tr>
                <th>影響</th>
                <th>計數</th>
            </tr>
            ${Object.entries(auditResults.summary.violationsByImpact)
                .map(([impact, count]) => `
                    <tr>
                        <td>${impact}</td>
                        <td>${count}</td>
                    </tr>
                `).join('')}
        </table>
    </div>
    
    <h2>詳細違規</h2>
    ${auditResults.violations.map(violation => `
        <div class="violation ${violation.impact}">
            <h3>${violation.help}</h3>
            <p><strong>規則：</strong> ${violation.id}</p>
            <p><strong>影響：</strong> ${violation.impact}</p>
            <p>${violation.description}</p>
            
            <h4>受影響元素 (${violation.nodes.length})</h4>
            ${violation.nodes.map(node => `
                <div class="code">
                    <strong>元素：</strong> ${this.escapeHtml(node.html)}<br>
                    <strong>選擇器：</strong> ${node.target.join(' ')}<br>
                    <strong>修復：：</strong> ${node.failureSummary}
                </div>
            `).join('')}
            
            <p><a href="${violation.helpUrl}" target="_blank">了解更多</a></p>
        </div>
    `).join('')}
    
    <h2>需要手動測試</h2>
    <ul>
        <li>使用螢幕閱讀器測試 (NVDA, JAWS, VoiceOver)</li>
        <li>徹底測試鍵盤導航</li>
        <li>在 200% 縮放下測試瀏覽器</li>
        <li>使用 Windows 高對比模式測試</li>
        <li>審查內容是否使用簡潔語言</li>
    </ul>
</body>
</html>
        `;
        
        return html;
    }
    
    generateJSONReport(auditResults) {
        return {
            metadata: {
                timestamp: new Date().toISOString(),
                url: auditResults.url,
                wcagVersion: '2.1',
                level: 'AA'
            },
            summary: auditResults.summary,
            violations: auditResults.violations.map(v => ({
                id: v.id,
                impact: v.impact,
                help: v.help,
                count: v.nodes.length,
                elements: v.nodes.map(n => ({
                    target: n.target.join(' '),
                    html: n.html
                }))
            })),
            passes: auditResults.passes,
            incomplete: auditResults.incomplete
        };
    }
}
```

## 輸出格式

1. **無障礙分數**：WCAG 等級的總體合規性分數
2. **違規報告**：包含嚴重性和修復的詳細問題列表
3. **測試結果**：自動化和手動測試結果
4. **補救指南**：每個問題的逐步修復
5. **程式碼範例**：可訪問組件實施
6. **測試腳本**：用於 CI/CD 的可重用測試套件
7. **檢查清單**：QA 的手動測試檢查清單
8. **進度追蹤**：無障礙改進指標

專注於創建包容性體驗，無論使用者的能力或輔助技術如何，都能為所有使用者提供服務。
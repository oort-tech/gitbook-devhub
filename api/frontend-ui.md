# Frontend Development: Configuration UI

Datahub dynamically loads the plugin's JS file when a user selects the plugin. This JS is responsible for rendering the configuration form, allowing the user to input required parameters. The configuration data is synchronized with Datahub in real-time and saved in the task's `config` field.

## 1. JS Format Requirements

### Recommended Format: IIFE (Immediately Invoked Function Expression)
The plugin JS should be an IIFE that returns an initialization function. This prevents global scope pollution and ensures a clean initialization process.

```javascript
(function() {
    // Plugin Code
    return function (containerId, updateConfig) {
        // Initialization Logic
    };
})();

```

### Parameters

* **`containerId` (string):** The DOM element ID provided by Datahub where the plugin must render its UI.
* **`updateConfig` (function):** A callback function used to report configuration changes to Datahub.

### Execution Timing

* Datahub calls the initialization function immediately after the user selects the plugin.
* The function is called once with the `containerId` and `updateConfig` callback.

## 2. Core Functional Requirements

### 2.1 Configuration Reporting

Plugins must report configuration data in real-time using the `updateConfig` callback.

1. **Immediate Default Report:** On initialization, call `updateConfig` once to set default values.
2. **Real-time Updates:** Whenever the user modifies an input, call `updateConfig` immediately with the latest configuration object.
3. **Format:** The configuration must be a serializable JSON object.

**Example:**

```javascript
// Default config
let config = { keyword: "", maxImages: 20 };

// Report defaults immediately
updateConfig(config);

// Report changes on user input
input.addEventListener('input', (e) => {
    config.keyword = e.target.value;
    updateConfig(config); 
});

```

### 2.2 UI Rendering

The plugin must render the configuration form into the specified container.

```javascript
const container = document.getElementById(containerId);
if (!container) return;
container.innerHTML = '<div class="...">...</div>';

```

**Style Guidelines:**

* **Tailwind CSS:** Datahub uses Tailwind CSS. Plugins are encouraged to use Tailwind classes for consistent styling (e.g., `border-slate-300`, `focus:border-primary`).
* **Responsive:** Ensure the UI adapts to different screen sizes.

### 2.3 Internationalization (i18n)

Plugins **must** support English (`en`) and Chinese (`zh`) to adapt to Datahub's multi-language environment.

1. **Dictionary:** Define a `messages` object containing translations.
2. **Detection:** Get the current language from `localStorage.getItem('datahub_lang')` (defaults to `zh`).
3. **Listeners:**
* Listen for the `storage` event to handle cross-tab language changes.
* Listen for the `languageChanged` custom event to handle same-page language changes.



## 3. Configuration Data Structure

### 3.1 Format

The configuration must be a JSON object supporting:

* Strings
* Numbers
* Booleans
* Arrays
* Nested Objects

### 3.2 Validation

Plugins should validate user input before updating the config. For example, ensure numbers are positive integers or required fields are not empty.

## 4. Complete Code Example

Below is a full example of an Image Search Plugin configuration form.

```javascript
/**
 * Image Search Plugin Configuration Module
 */
(function() {
    // Translation messages
    const messages = {
        zh: {
            keyword: "搜索关键词",
            placeholder: "例如: 小猫咪",
            maxImages: "最大图片数",
            required: "关键词必填"
        },
        en: {
            keyword: "Search Keyword",
            placeholder: "e.g. cute cat",
            maxImages: "Max Images",
            required: "Keyword is required"
        }
    };

    return {
        /**
         * Initialize the plugin configuration UI
         */
        init: function(containerId, updateConfig) {
            const container = document.getElementById(containerId);
            if (!container) return;

            // Helper to get current language
            function getCurrentLang() {
                return localStorage.getItem('datahub_lang') || 'zh';
            }

            // Translation function
            function t(key) {
                const lang = getCurrentLang();
                return messages[lang] && messages[lang][key] ? messages[lang][key] : key;
            }

            // Default Configuration
            let config = {
                keyword: "",
                maxImages: 20
            };

            // Report default configuration immediately
            updateConfig(config);

            // Render the configuration form
            function render() {
                const html = `
                    <div class="space-y-4 text-left">
                        <div>
                            <label class="block text-xs font-medium text-slate-600 mb-2">${t('keyword')}</label>
                            <input type="text" data-key="keyword" 
                                   placeholder="${t('placeholder')}"
                                   value="${config.keyword}"
                                   class="w-full rounded-lg border-slate-300 shadow-sm focus:border-primary focus:ring-2 focus:ring-primary focus:ring-opacity-20 text-sm py-2.5 px-3 bg-white transition-all">
                        </div>

                        <div>
                            <label class="block text-xs font-medium text-slate-600 mb-2">${t('maxImages')}</label>
                            <input type="number" data-key="maxImages" 
                                   value="${config.maxImages}" min="1"
                                   class="w-full rounded-lg border-slate-300 shadow-sm focus:border-primary focus:ring-2 focus:ring-primary focus:ring-opacity-20 text-sm py-2.5 px-3 bg-white transition-all">
                        </div>
                    </div>
                `;
                container.innerHTML = html;
                bindEvents();
            }

            // Bind event listeners to inputs
            function bindEvents() {
                const inputs = container.querySelectorAll('input');
                inputs.forEach(input => {
                    // Clone to remove old listeners
                    const newInput = input.cloneNode(true);
                    input.parentNode.replaceChild(newInput, input);

                    newInput.addEventListener('input', (e) => {
                        const key = e.target.getAttribute('data-key');
                        let val = e.target.value;

                        // Number validation
                        if (e.target.type === 'number') {
                            val = Number(val);
                            if (val < 1 || !Number.isInteger(val)) {
                                val = 1;
                            }
                        }

                        config[key] = val;
                        updateConfig(config); // Real-time reporting
                    });
                });
            }

            // Initial render
            render();

            // Listen for language changes (cross-tab)
            window.addEventListener('storage', function(e) {
                if (e.key === 'datahub_lang') {
                    render();
                }
            });

            // Listen for language changes (same page)
            window.addEventListener('languageChanged', function() {
                render();
            });
        },

        /**
         * Validate the configuration object
         */
        validateConfig: function(config) {
            function getCurrentLang() {
                return localStorage.getItem('datahub_lang') || 'zh';
            }
            
            const lang = getCurrentLang();
            const msg = messages[lang] || messages['zh'];

            if (!config || !config.keyword || config.keyword.trim() === '') {
                return {
                    valid: false,
                    message: msg.required
                };
            }
            return { valid: true, message: "" };
        }
    };
})();

```

## 5. Development Guidelines

### 5.1 Security

* **Scope:** Use IIFE to avoid global namespace pollution.
* **XSS:** When using `innerHTML`, ensure user input is properly escaped.

### 5.2 Performance

* **Event Listeners:** Avoid duplicate bindings. When re-rendering, ensure old listeners are cleaned up (e.g., by cloning nodes or removing the previous innerHTML).
* **DOM Access:** Minimize DOM operations.

### 5.3 Compatibility

* **Browsers:** Ensure the code runs in modern browsers (Chrome, Firefox, Safari, Edge).
* **Version:** Use ES5/ES6 based on the target environment.

## 6. Testing Checklist

Before submitting your plugin JS, verify:

* [ ] JS follows the IIFE format.
* [ ] Default configuration is reported immediately on init.
* [ ] Configuration updates are reported in real-time on user input.
* [ ] English and Chinese switching works correctly.
* [ ] UI styling matches Datahub standards (Tailwind).
* [ ] Input validation prevents invalid data.

```

```
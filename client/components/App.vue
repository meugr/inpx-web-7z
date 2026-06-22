<template>
    <div class="fit row">
        <Api ref="api" v-model="accessGranted" />
        <Notify ref="notify" />
        <StdDialog ref="stdDialog" />

        <router-view v-if="accessGranted" v-slot="{ Component }">
            <keep-alive>
                <component :is="Component" class="col" />
            </keep-alive>
        </router-view>        
    </div>
</template>

<script>
//-----------------------------------------------------------------------------
import vueComponent from './vueComponent.js';

//import * as utils from '../share/utils';
import Notify from './share/Notify.vue';
import StdDialog from './share/StdDialog.vue';

import Api from './Api/Api.vue';
import Search from './Search/Search.vue';

const componentOptions = {
    components: {
        Api,
        Notify,
        StdDialog,

        Search,
    },
    watch: {
    },

};
class App {
    _options = componentOptions;
    accessGranted = false;

    created() {
        this.commit = this.$store.commit;

        //root route
        let cachedRoute = '';
        let cachedPath = '';
        this.$root.getRootRoute = () => {
            if (this.$route.path != cachedPath) {
                cachedPath = this.$route.path;
                const m = cachedPath.match(/^(\/[^/]*).*$/i);
                cachedRoute = (m ? m[1] : this.$route.path);
            }
            return cachedRoute;
        }

        this.$root.isMobileDevice = /Android|webOS|iPhone|iPad|iPod|BlackBerry/i.test(navigator.userAgent);

        this.$watch(
            () => this.$store.state.settings.einkMode,
            (einkMode) => {
                document.documentElement.classList.toggle('eink', einkMode);
            },
            { immediate: true }
        );
        this.$root.setAppTitle = this.setAppTitle;

        //global keyHooks
        this.keyHooks = [];
        this.keyHook = (event) => {
            for (const hook of this.keyHooks)
                hook(event);
        }

        this.$root.addKeyHook = (hook) => {
            if (this.keyHooks.indexOf(hook) < 0)
                this.keyHooks.push(hook);
        }

        this.$root.removeKeyHook = (hook) => {
            const i = this.keyHooks.indexOf(hook);
            if (i >= 0)
                this.keyHooks.splice(i, 1);
        }

        document.addEventListener('keyup', (event) => {
            this.keyHook(event);
        });
        document.addEventListener('keypress', (event) => {
            this.keyHook(event);
        });
        document.addEventListener('keydown', (event) => {
            this.keyHook(event);
        });        
    }

    mounted() {
        this.$root.api = this.$refs.api;
        this.$root.notify = this.$refs.notify;
        this.$root.stdDialog = this.$refs.stdDialog;

        this.setAppTitle();
    }

    get config() {
        return this.$store.state.config;
    }

    get rootRoute() {
        return this.$root.getRootRoute();
    }

    setAppTitle(title) {
        if (title) {
            document.title = title;
        }
    }
}

export default vueComponent(App);
//-----------------------------------------------------------------------------
</script>

<style scoped>
</style>

<style>
body, html, #app {    
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100%;
    font: normal 13px Web Default;
}

.dborder {
    border: 2px solid yellow;
}

.icon-rotate {
    vertical-align: middle;
    animation: rotating 2s linear infinite;
}

.q-dialog__inner--minimized {
    padding: 10px !important;
}

.q-dialog__inner--minimized > div {
    max-height: 100% !important;
    max-width: 800px !important;
}

@keyframes rotating { 
    from { 
        transform: rotate(0deg); 
    } to { 
        transform: rotate(360deg); 
    }
}

@font-face {
    font-family: 'Web Default';
    src: url('fonts/web-default.ttf') format('truetype');
}

@font-face {
    font-family: 'Verdana';
    font-weight: bold;
    src: url('fonts/web-default-bold.ttf') format('truetype');
}

/* ===================== E-INK MODE ===================== */

/* Отключаем анимации и переходы */
html.eink *,
html.eink *::before,
html.eink *::after {
    animation: none !important;
    transition: none !important;
}

html.eink .q-ripple,
html.eink .q-focus-helper {
    display: none !important;
}

/* Базовый шрифт крупнее */
html.eink body,
html.eink html,
html.eink #app {
    font-size: 18px;
}

/* Убираем все Quasar цвета — только чёрный/белый */
html.eink [class*="bg-"] {
    background: #fff !important;
}

html.eink [class*="text-"] {
    color: #000 !important;
}

/* Toolbar — плоский светло-серый вместо голубого */
html.eink .tool-panel {
    background: #e8e8e8 !important;
    border-bottom: 2px solid #888;
}

/* Поля ввода — крупнее */
html.eink .q-field__control {
    min-height: 48px !important;
}

html.eink .q-field__label {
    font-size: 15px !important;
}

/* Кнопки Quasar — крупный тап-зон */
html.eink .q-btn {
    min-height: 48px !important;
    min-width: 48px !important;
    font-size: 16px !important;
    border: 1px solid #aaa !important;
}

/* Toggle-кнопки (Авторы/Серии/Названия) */
html.eink .q-btn-toggle .q-btn {
    min-height: 44px !important;
}

/* Чекбоксы в настройках */
html.eink .q-checkbox__inner {
    font-size: 44px !important;
}

/* Строки списков — чётные с серым фоном */
html.eink .odd-item {
    background: #d8d8d8 !important;
}

/* Разделитель между строками */
html.eink .odd-item,
html.eink .column:has(> .book-row) {
    border-bottom: 1px solid #aaa;
}

/* Автор/серия — кликабельное: больший тап-зон */
html.eink .clickable2 {
    color: #000 !important;
    padding: 6px 4px !important;
    display: inline-block;
}

/* Кнопки действий (скачать, инфо, читать) — как кнопки */
html.eink .clickable {
    color: #000 !important;
    border: 1px solid #666 !important;
    border-radius: 4px !important;
    padding: 3px 8px !important;
    display: inline-block !important;
    margin: 2px 1px !important;
    background: #f4f4f4 !important;
    font-size: 14px !important;
}

/* Строки книг — больший вертикальный отступ */
html.eink .q-my-sm {
    margin-top: 6px !important;
    margin-bottom: 6px !important;
}

/* Пагинация — заметнее */
html.eink .q-ml-md.q-my-xs {
    font-size: 130% !important;
    padding: 4px 0 !important;
}

/* Диалоги на весь экран в portrait */
@media (max-width: 800px) {
    html.eink .q-dialog__inner--minimized > div {
        max-width: 100% !important;
        width: 100%;
    }
}

/* ===================================================== */
</style>

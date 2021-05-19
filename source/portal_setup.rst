.. _portal_setup:

***************************************************
6. Установка и настройка портала для STB и Smart TV
***************************************************

Портал - это набор Javascript-приложений (JS/HTML/CSS) в разных стилевых и функциональных вариантах (шаблонов), представляющих
собой оболочку (интерфейс) для абонента. Портал загружается встроенным в устройство (приставку или Smart TV) браузером.

6.1. Установка портала
======================

Поскольку портал это статические файлы, не требующие сервера приложений с поддержкой выполнения каких-либо скриптов, то
для его хостинга достаточно любого веб-сервера. Также, при необходимости, портал может быть размещен на CDN-сервисе
или встроен в прошивку устройства.

С сервером Smarty портал взаимодействует через HTTP API посредством выполнения XMLHttpRequest-запросов.

Для хостинга портала рекомендуется использовать веб-сервер nginx.

Настройки портала задаются в файле ``client.js``, размещенном в корневой директории. Для удобства деплоя и администрирования
client.js, обычно, является симлинком на специфичный для данного провайдера конфиг, размещенный в ``/etc/microimpuls/portal/``.

.. note::

    Из-за политики ограничения выполнения кроссдоменных запросов на некоторых устройствах рекомендуется для выполнения
    запросов к Smarty использовать тот же домен, где размещен портал. В стандартном конфиге nginx для портала, который
    идет в комплекте с установочным пакетом портала, указан специальный location /api, осуществляющий редирект на
    Smarty.

.. _client_js_options:

6.2. Параметры конфигурации client.js
=====================================

В конфиге client.js задаются как основные параметры (например, данные для подключения к Smarty), так и специфичные
для каждого приложения (шаблона).

.. _client_js_main_options:

Обязательные параметры
++++++++++++++++++++++

Актуальная документация по обязательным параметрам: https://microimpuls.com/docs/smarty/portal-and-apps-settings/portal-settings


.. _client_js_additional_options:

Дополнительные параметры
++++++++++++++++++++++++

Актуальная документация по дополнительным параметрам: https://microimpuls.com/docs/smarty/portal-and-apps-settings/portal-settings

.. _client_js_specific_options:

Специфичные для шаблонов параметры
++++++++++++++++++++++++++++++++++

Актуальная документация по дополнительным параметрам шаблона impuls: https://microimpuls.com/docs/smarty/portal-and-apps-settings/impuls-settings

Актуальная документация по дополнительным параметрам шаблона focus: https://microimpuls.com/docs/smarty/portal-and-apps-settings/focus-settings

Актуальная документация по дополнительным параметрам шаблона futuristic: https://microimpuls.com/docs/smarty/portal-and-apps-settings/futuristic-settings

Актуальная документация по дополнительным параметрам шаблона infinitly: https://microimpuls.com/docs/smarty/portal-and-apps-settings/infinitly-settings


.. _portal_event_system:

6.3. Механизм событий
=====================

События позволяют добавлять или переопределять некоторый функционал портала в разные моменты его жизненного цикла.
Обработчики событий задаются в конфиге client.js, что позволяет сохранить кастомные настройки и функции даже при обновлении
портала на новую версию.

Доступные события:

* *OnDataRequestError* - ошибка выполнения запроса к Smarty, в качестве аргумента - код ошибки
* *OnAppInitBegin* - старт инициализации приложения
* *OnAppInitEnd* - завершение инициализации приложения
* *OnDeviceInitBegin* - старт инициализации драйвера устройства
* *OnDeviceInitEnd* - завершение инициализации драйвера устройства
* *OnDeviceKeyEvent* - событие нажатия клавиши пульта, в качестве аргумента - код клавиши
* *OnAccountLoginSuccessful* - успешная авторизация аккаунта

Жизненный цикл: *OnAppInitBegin > OnDeviceInitBegin > OnDeviceInitEnd > OnAppInitEnd*.

Пример создания обработчиков событий приведен ниже.

.. note::

    Внимание! Использование этого механизма требует навыков программирования на Javascript, знания архитектуры и API портала и API устройств.

6.4. Пример конфигурации
========================

Пример ниже предназначен для шаблона ``impuls`` и кроме стандартного поведения добавляет дополнительный пункт в меню
настроек абонента - "Режим ожидания", включающий или отключающий обработку событий подключения/отключения HDMI-кабеля
на приставке MAG, а также проверяет версию прошивки и запускает обновление в случае необходимости при старте
приложения и добавляет некоторые другие функции. ::

    var CLIENT_SETTINGS = {
        'client_id': 1,
        'api_key': '***',
        'api_url': '/api',
        'template_name': 'impuls',
        'template_size': {
            'impuls': {
                'default': [1280, 720]
            },
            'classic': {
                'default': [1280, 720],
                '720x576': [720, 576]
            },
        },
        'available_templates': ['futuristic'],
        'template_styles': {
            'futuristic': ['futuristic', 'futuristic_x']
        },
        'settings_filename': 'example.dat',
        'site_url': 'www.example.com',
        'signup_auto_activation_period': 0,
        'show_welcome_message': false,1
        'registration_available': false,
        'settings_menu_custom_items': [
            'handle-hdmi-events'
        ],
        'auth_mode': 'device_uid',
        'default_timezone': 12,
        'default_buffersize': 3,
    };

    OnAppInitBegin = function()
    {
        // Автоматическое выключение плеера ночью в 01:30 по локальному времени
        setInterval(function(){
            var date = new Date();
            if (date.getHours() == 1 && date.getMinutes() == 30) {
                App.player.stop();
                App.display.showScreen('tvchannels');
            }
        }, 35000);

        // Обновление прошивки для определенных старых версий
        try {
            var fver = parseInt(gSTB.RDir('ImageVersion'));
            var desc = gSTB.GetDeviceImageDesc();
            if (desc == 'default-214') {
                stbUpdate.startAutoUpdate("http://update.example.com/imageupdate", true);
            }
        } catch (e) {}
    };

    OnAppInitEnd = function()
    {
        // Подключение custom css верстки
        var el = document.createElement('link');
        el.rel = 'stylesheet';
        el.type = 'text/css';
        el.href = '/custom/css/custom.css';
        document.getElementsByTagName('body')[0].appendChild(el);

        // Подключение режима overscan для шаблона impuls
        Helper.addBodyClass('overscan');

        // Переопределение кнопки EPG на красную кнопку
        App.playerScreen.key_epg = App.playerScreen.key_red;
        App.tvChannelsScreen.key_epg = App.tvChannelsScreen.key_red;

        // Переопределение обработчика кнопки Power
        App.display.setGlobalKeyCodeHandler('power', function(){
            App.player.stop();
            App.display.showScreen('mainmenu');
        }, App.playerScreen);
    };

    OnDeviceInitBegin = function()
    {
        // Добавление меню настройки режима ожидания
        var handleHdmiEventsMenu = new BaseMenu({
            menuTag: 'ul',
            itemIdPrefix: 'settings-handle-hdmi-events-value',
            useItemIdWithoutIndex: true,
            itemTag: 'span',
            displayItemsNumber: 1,
            sourceItemsNumber: 2,
            useFastRefresh: false,
            getItemNameFunc: function(sourceItemIndex, displayItemIndex) {
                var names = [
                    'Отключен',
                    'Включен'
                ];
                return names[sourceItemIndex];
            }
        });
        App.settings.setCustomSetting('handle-hdmi-events', 1);
        App.lang.set('ru', 'label-settings-handle-hdmi-events', 'Режим ожидания');
        App.settingsScreen.addCustomSettingsMenu('handle-hdmi-events', handleHdmiEventsMenu);
    };

    OnDeviceInitEnd = function()
    {
        // Установка произвольной прозрачности интерфейса
        App.device.setWindowTransparencyLevel(230);

        // Установка режима 16:9 по умолчанию
        App.device.setAspectRatioMode('16:9');

        // Обработка событий HDMI для MAG
        if (App.device.getDeviceKind() === 'mag') {
            App.device.standBy = false;
            App.device.standByTimer = null;
            App.device.onEvent = function(code) {
                switch (code) {
                    default:
                        break;
                    case 0x20: // HDMI connected
                        if (Helper.toInt(App.settings.getCustomSetting('handle-hdmi-events')) == 1) {
                            clearTimeout(App.device.standByTimer);
                            if (App.device.standBy) {
                                App.device.stb.StandBy(false);
                                App.device.standBy = false;
                            }
                        }
                        break;
                    case 0x21: // HDMI disconnected
                        if (Helper.toInt(App.settings.getCustomSetting('handle-hdmi-events')) == 1) {
                            clearTimeout(App.device.standByTimer);
                            App.device.standByTimer = setTimeout(function () {
                                if (!App.device.standBy) {
                                    if (App.player.isActive()) {
                                        App.player.stop();
                                        App.display.showScreen('tvchannels');
                                    }
                                    App.device.stb.StandBy(true);
                                    App.device.standBy = true;
                                }
                            }, 60 * 5 * 1000);
                        }
                        break;
                }
            }
        }
    };

    OnAccountLoginSuccessful = function()
    {
        // Изменение формата отображения оставшихся дней активации на dd/mm/yyyy
        var value = App.data.getActivationDaysLeft();
        if (parseInt(value) <= 0) {
            value = App.lang.get('auto-renewal');
        } else {
            var d = new Date(new Date().getTime()+(parseInt(value)*24*60*60*1000));
            var dd = d.getDate();
            var mm = d.getMonth() + 1;
            var y = d.getFullYear();
            value = dd + '/' + mm + '/' + y;
        }
        Helper.setHtml('info-menu-activation-days-left', value);
    };

6.5 Кастомизация стилей оформления портала
==========================================

Smarty позволяет для каждого устройства задать внешний css-файл для кастомного оформления портала, например, установить
своё фоновое изображение, сменить цвет шрифта или фокуса.

Для этого необходимо открыть в панели администрирования "Общие настройки" -> "Настройки STB и приложений" ->
<устройство для настройки> и прописать в поле "Внешний CSS" путь до файла с новыми стилями. Как правило, данный файл
располагают на том же веб-сервере, что и портал.

Для создания файла с внешними стилями достаточно с помощью отладчика любого браузера проследить какие классы и
идентификаторы отвечают за тот или иной элемент экрана в портале, после чего перезаписать/дописать нужные свойства к ним.
Ниже представлены самые часто встречающиеся примеры кастомизации для шаблонов ``futuristic`` и ``impuls``.

Пример внешнего css-файла для кастомизации шаблона ``futuristic``. ::

    /* Установка кастомного фонового изображения */

    .screen, .android_stb .screen {
        background: url('example-bg.jpg') no-repeat;
    }

    /* Установка кастомного фонового изображения для верхней панели с лого оператора */

    #statusbar-screen {
        background: transparent url('statusbar-bg.png') no-repeat;
    }

    /* Установка кастомного изображения для фокуса главного меню  */

    div#main-menu-selection {
        background: transparent url('example-selection-bg.png') no-repeat center 0px;
    }
    
    /* Установка кастомного шрифта */
    
    @font-face {
        font-family: 'Nunito';
        src: url('../fonts/Nunito/nunito-v12-latin_cyrillic.eot');
        src: url('../fonts/Nunito/nunito-v12-latin_cyrillic.eot?#iefix') format('embedded-opentype'),
        url('../fonts/Nunito/nunito-v12-latin_cyrillic.woff2') format('woff2'),
        url('../fonts/Nunito/nunito-v12-latin_cyrillic.woff') format('woff'),
        url('../fonts/Nunito/nunito-v12-latin_cyrillic.ttf') format('truetype');
        font-weight: normal;
        font-style: normal;
    }
    
    body {
        font-family: "Nunito", "HelveticaNeue-Light", "Helvetica Neue Light", "Helvetica Neue", Helvetica, Arial, "Lucida Grande", sans-serif;
    }

Пример внешнего css-файла для кастомизации шаблона ``impuls``. ::

    /* Установка кастомного лоадера */

    .screen-container.loader, #player-mode-icon.loader, #loading-bar {
        background: transparent url("loader-example.gif") center center no-repeat;
    }

    #firstloading-loader {
        background: url('loader-example.gif') no-repeat;
        height: 65px;
        width: 120px;
    }

    .loader#video-actions-panel {
        background: transparent url("loader-example.gif") center left no-repeat;
        height: 120px;
    }

    /* Установка кастомного фонового изображения */

    .screen, body, .play .screen, .png-transparency .screen, .play.png-transparency .screen,
    .transparent.png-transparency .screen, .transparent .screen {
        background: url('example-bg.jpg') no-repeat;
    }

    body.play {
        background: transparent;
    }

    /* Установка кастомных координат для логотипа оператора */

    .client-logo {
        margin-top: 75px !important;
        margin-right: 30px;
    }

.. note::

    Нужно учесть, что для корректной работы внешних стилей, необходимо, чтобы все дополнительные ресурсы (изображения,
    шрифты), используемые в них, были доступны и имели правильные пути.

.. note::
    После установки обновлений шаблонов необходимо внимательно читать changelog и тщательно тестировать портал на
    предмет корректности работы внешнего CSS, так как внутренние css- и html-файлы могут претерпевать изменения, что
    так или иначе влияет на отображение кастомных стилей.



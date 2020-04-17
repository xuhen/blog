---
title: 使用 userAgent 检测浏览器环境
date: 2020-04-17 18:31:33
tags:
    - browser
    - js
    - util
---

> 因为浏览器环境复杂，而且怪癖也不一样。有时我们希望定位当前浏览器的具体环境、浏览器类型，和版本号去做相应的适配。
>
> 那就要用到客户端检测技术，此技术应用广泛，比如我们前端的监控系统，一旦代码报错，需要定位当前系统平台，浏览器类型，和版本号。


下面给出根据userAgent字段去做浏览器端的系统检测：


```
var client = function () {
      var engine = {
        //呈现引擎 
        ie: 0,
        gecko: 0,
        webkit: 0,
        khtml: 0,
        opera: 0,
        //具体的版本号
        ver: null
      };

      var browser = {
        //浏览器 
        ie: 0,
        firefox: 0,
        safari: 0,
        konq: 0,
        opera: 0,
        chrome: 0,
        //具体的版本
        ver: null
      };

      var system = {
        win: false,
        mac: false,
        x11: false,
        //移动设备
        iphone: false,
        ipod: false,
        ipad: false,
        ios: false,
        android: false,
        nokiaN: false,
        winMobile: false,
        //游戏系统
        wii: false,
        ps: false
      };
      //在此检测呈现引擎、平台和设备

      var ua = navigator.userAgent;


      if (window.opera) {
        engine.ver = browser.ver = window.opera.version();
        engine.opera = browser.opera = parseFloat(engine.ver);
      } else if (/AppleWebKit\/(\S+)/.test(ua)) {
        engine.ver = RegExp["$1"];
        engine.webkit = parseFloat(engine.ver);

        //确定是 Chrome 还是 Safari
        if (/Chrome\/(\S+)/.test(ua)) {
          browser.ver = RegExp["$1"];
          browser.chrome = parseFloat(browser.ver);
        } else if (/Version\/(\S+)/.test(ua)) {
          browser.ver = RegExp["$1"];
          browser.safari = parseFloat(browser.ver);
        } else {
          //近似地确定版本号
          var safariVersion = 1;
          if (engine.webkit < 100) {
            safariVersion = 1;
          } else if (engine.webkit < 312) {
            safariVersion = 1.2;
          } else if (engine.webkit < 412) {
            safariVersion = 1.3;
          } else {
            safariVersion = 2;
          }

          browser.safari = browser.ver = safariVersion;
        }

      } else if (/KHTML\/(\S+)/.test(ua) || /Konqueror\/([^;]+)/.test(ua)) {
        engine.ver = browser.ver = RegExp["$1"];
        engine.khtml = browser.konq = parseFloat(engine.ver);
      } else if (/rv:([^\)]+)\) Gecko\/\d{8}/.test(ua)) {
        engine.ver = RegExp["$1"];
        engine.gecko = parseFloat(engine.ver);
        //确定是不是 Firefox
        if (/Firefox\/(\S+)/.test(ua)) {
          browser.ver = RegExp["$1"];
          browser.firefox = parseFloat(browser.ver);
        }
      } else if (/MSIE ([^;]+)/.test(ua)) {
        engine.ver = browser.ver = RegExp["$1"];
        engine.ie = browser.ie = parseFloat(engine.ver);
      }

      var p = navigator.platform;

      system.win = p.indexOf("Win") == 0;
      system.mac = p.indexOf("Mac") == 0;
      system.x11 = (p.indexOf("X11") == 0) || (p.indexOf("Linux") == 0);
      system.iphone = ua.indexOf("iPhone") > -1;
      system.ipod = ua.indexOf("iPod") > -1;
      system.ipad = ua.indexOf("iPad") > -1;

      //检测 iOS 版本
      if (system.mac && ua.indexOf("Mobile") > -1) {
        if (/CPU (?:iPhone )?OS (\d+_\d+)/.test(ua)) {
          system.ios = parseFloat(RegExp.$1.replace("_", "."));
        } else {
          system.ios = 2; //不能真正检测出来，所以只能猜测
        }
      }
      if (/Android (\d+\.\d+)/.test(ua)) {
        system.android = parseFloat(RegExp.$1);
      }
      system.nokiaN = ua.indexOf("NokiaN") > -1;
      //windows mobile
      if (system.win == "CE") {
        system.winMobile = system.win;
      } else if (system.win == "Ph") {
        if (/Windows Phone OS (\d+.\d+)/.test(ua)) {
          system.win = "Phone";
          system.winMobile = parseFloat(RegExp["$1"]);
        }
      }

      if (system.win) {
        if (/Win(?:dows )?([^do]{2})\s?(\d+\.\d+)?/.test(ua)) {
          if (RegExp["$1"] == "NT") {
            switch (RegExp["$2"]) {
              case "5.0":
                system.win = "2000";
                break;
              case "5.1":
                system.win = "XP";
                break;
              case "6.0":
                system.win = "Vista";
                break;
              case "6.1":
                system.win = "7";
                break;
              default:
                system.win = "NT";
                break; 11
            }
          } else if (RegExp["$1"] == "9x") {
            system.win = "ME";
          } else {
            system.win = RegExp["$1"];
          }
        }
      }

      system.wii = ua.indexOf("Wii") > -1;
      system.ps = /playstation/i.test(ua);

      return {
        engine: engine,
        browser: browser,
        system: system
      };
    }();

    // 使用场景1
    if (client.engine.webkit) { //if it’s WebKit 
      if (client.browser.chrome) {
        //执行针对 Chrome 的代码
      } else if (client.browser.safari) {
        //执行针对 Safari 的代码 
      }
    } else if (client.engine.gecko) {
      if (client.browser.firefox) {
        //执行针对 Firefox 的代码 
      } else {
        //执行针对其他 Gecko 浏览器的代码
      }
    }

    // 使用场景2
    if (client.system.win) {
      if (client.system.win == "XP") {
        //说明是 XP
      } else if (client.system.win == "Vista") {
        //说明是 Vista 
      }
    }


    // 使用场景3
    if (client.engine.webkit) {
      if (client.system.iOS) {
        //iOS 手机的内容
      } else if (client.system.android) {
        //Android 手机的内容
      } else if (client.system.nokiaN) {
        //诺基亚手机的内容 
      }
    }


    console.log('client', client);
```
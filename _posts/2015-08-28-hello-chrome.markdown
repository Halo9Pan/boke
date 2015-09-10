---
layout: post
title:  "Hello Chrome!"
date:   2015-09-11 14:00:00
categories: Development
---

### MyChrome
墙内升级 Chrome 的利器：[MyChrome](http://code.taobao.org/p/mychrome/src/trunk/release/)，选择sina.com.cn的源就可以在墙内升级了

### 命令行启动参数
没有官方的启动参数文档，只有一个三方的文档，是作者根据 Chrome 分析出来的：[List of Chromium Command Line Switches](http://peter.sh/experiments/chromium-command-line-switches/)  介绍一些常用的：  

+ **--user-data-dir** 指定用户数据目录，每个 Chrome 使用独立的数据目录，比如普通上网浏览用一个，开发时用一个，每个数据目录有不同的配置、扩展和应用。这是比 Profile 更高一级的隔离。同一数据目录下的 Profile 虽然也能配置不同的设置、扩展和应用，但是所有的 Profile 的扩展会都启动，消耗内存。  
+ **--incognito** 直接用隐身模式启动。  
+ **--show-app-list** 打开 Chrome 应用列表。  
+ **--disable-plugins-discovery** 禁用所有的插件，世界清静了。  
+ **--disable-extensions** 禁用所有的扩展，世界空白了，只剩下 Chrome 了。  
+ **--remote-debugging-port** 远程调试端口
+ --enable-devtools-experiments --enable-experimental-canvas-features --enable-experimental-web-platform-features --enable-experimental-websocket --enable-encrypted-media --enable-accelerated-2d-canvas --enable-webgl-draft-extensions 打开各种实验功能，体验新特性时使用，但是通过命令行设置不一定有效 -_-!  

### chrome://
+ **chrome://version/** 查看版本、执行文件路径以及数据文件路径  
+ **chrome://flags/** 可以修改一些 Chrome 的细节配置，不过这些配置开关大部分都是实验性的
+ **chrome://dns/** 可以查看 Chrome DNS 的预解析，修改 host 文件无效时可以看看这里
+ **chrome://memory/** 查看当前 Chrome 的内存消耗，比 Shift+Esc 更详细
+ **chrome://net-internals/** Chrome 和网络相关的详细信息，事件捕获、预加载、代理、DNS、Sockets、HTTP2等
+ **chrome://quota-internals/** Chrome 磁盘存储的配额信息
+ **chrome://sync-internals/** Chrome 账户同步信息
+ **chrome://profiler/** Chrome 性能调优信息，每一步执行的详细步骤以及消耗等
+ **chrome://user-actions/** 捕获用户的所有操作  
+ **chrome://settings/** 设置  
+ **chrome://extensions/** 扩展  
+ **chrome://downloads/** 下载  
+ **chrome://history/** 历史  
+ **chrome://bookmarks/** 书签  
+ **chrome://about/** 常用的都在这里了(＾－＾)V  

### DevTools
**Ctrl+Shift+I** 欢迎来到 Chrome DevTools，神奇强大的工具

DevTools 目前（45.0.2454.85）主要包括以下八个主要功能组：
**Ctrl+[**    **Ctrl+]**
#### Elements
DOM元素和样式
#### Resources
#### Network
#### Sources
#### Timeline
#### Profiles
#### Audits
#### Console
{% highlight js %}
console.log("The current time is:", Date.now())
console.debug("The current time is:", Date.now())
console.info("The current time is:", Date.now())
console.error("The current time is:", Date.now())
console.warn("The current time is:", Date.now())
console.group("Console group");
console.log("The current time is:", Date.now());
console.groupEnd();
console.table([[1,2,3], [2,3,4]]);
console.log("The current time is: %O", Date())
console.log("%c天空深蓝", "color: DeepSkyBlue; font-size: large");
console.log(document)
console.dir(document)
console.time("Array initialize");
    var a = new Array(100000);
    console.assert(a.length > 200000, "a is > 2000000");
    for (var i = a.length - 1; i >= 0; i--) {
        if(i % 100 == 0) {
            console.timeStamp("100 Objects created.");
        }
        a[i] = new Object();
    };
console.timeEnd("Array initialize");

console.profile()
console.clear()
$()
$$()
$x()

monitorEvents(document.body, "click");
unmonitorEvents(document.body);

debugger;

profile()
profileEnd()

{% endhighlight %}

### 我的 Chrome 开发扩展和应用
[Context](https://chrome.google.com/webstore/detail/context/aalnjolghjkkogicompabhhbbkljnlka)  
[DHC - REST/HTTP API Client](https://chrome.google.com/webstore/detail/context/aejoelaoggembcahagimdiliamlcdmfm)  
[Caret-T](https://chrome.google.com/webstore/detail/context/agiednhnlghobdgpgfdnbdaflnngmoij)  
[Writebox](https://chrome.google.com/webstore/detail/context/bbehjmjchoiaglkeboicbgkpfafcmhij)  
[JSON Formatter](https://chrome.google.com/webstore/detail/context/bcjindcccaagfpapjjmafapmmgkkhgoa)  
[Web Developer](https://chrome.google.com/webstore/detail/context/bfbameneiokkgbdmiekhjnmfkcnldhhm)  
[DevTools Theme: Zero Dark Matrix](https://chrome.google.com/webstore/detail/context/bomhdjeadceaggdgfoefmpeafkjhegbo)  
[HTML Validator](https://chrome.google.com/webstore/detail/context/cgndfbhngibokieehnjhbjkkhbfmhojo)  
[JSONView](https://chrome.google.com/webstore/detail/context/chklaanhfefbnpoihckbnefhakgolnmc)  
[RegExp Tester App](https://chrome.google.com/webstore/detail/context/cmmblmkfaijaadfjapjddbeaoffeccib)  
[Clear Cache](https://chrome.google.com/webstore/detail/context/cppjkneekbjaeellbfkmgnhonkkjfpdn)  
[BuiltWith Technology Profiler](https://chrome.google.com/webstore/detail/context/dapjbgnjinbpoindlpdmhochffioedbn)  
[jQuery Debugger](https://chrome.google.com/webstore/detail/context/dbhhnnnpaeobfddmlalhnehgclcmjimi)  
[Tampermonkey](https://chrome.google.com/webstore/detail/context/dhdgffkkebhmkfjojejmpbldmpobfkfo)  
[Vim](https://chrome.google.com/webstore/detail/context/dhhoacdlegcbdglbfnhgnlchpkdlofkb)  
[Dark WebSocket Terminal](https://chrome.google.com/webstore/detail/context/dmogdjmcpfaibncngoolgljgocdabhke)  
[Markdown Editor](https://chrome.google.com/webstore/detail/context/dpibenlpmppnjcjfpcdgfomalnejildm)  
[Ra](https://chrome.google.com/webstore/detail/context/egipeapdjjhflkafmacobnmdbdkanoag)  
[ARC Welder](https://chrome.google.com/webstore/detail/context/emfinbmielocnlhgmfkkmkngdoccbadn)  
[Postman - REST Client](https://chrome.google.com/webstore/detail/context/fdmmgilgnpjigdojojpjoooidkmcomcm)  
[Full Page Screen Capture](https://chrome.google.com/webstore/detail/context/fdpohaocaechififmbbbbbknoalclacl)  
[RegExp Tester](https://chrome.google.com/webstore/detail/context/fekbbmalpajhfifodaakkfeodkpigjbk)  
[Postman](https://chrome.google.com/webstore/detail/context/fhbjgbiflinjbdggehcddcbncdddomop)  
[Serverauditor - SSH client](https://chrome.google.com/webstore/detail/context/fjcdjmmkgnkgihjnlbgcdamkadlkbmam)  
[Stylish](https://chrome.google.com/webstore/detail/context/fjnbnpbmkenffdnngjfgmeleoegfcffe)  
[Caret](https://chrome.google.com/webstore/detail/context/fljalecfjciodhpcledpamjachpmelml)  
[CSS Grady](https://chrome.google.com/webstore/detail/context/gdhlnmdfoeaagdlljpiklddgfnfidfli)  
[CSSViewer](https://chrome.google.com/webstore/detail/context/ggfgijbpiheegefliciemofobhmofgce)  
[Click&Clean](https://chrome.google.com/webstore/detail/context/ghgabhipcejejjmhhchfonmamedcbeod)  
[FastString - String Operations](https://chrome.google.com/webstore/detail/context/gpknmoniniacaobkeclmiiaekniaddnd)  
[Gradient Creator!](https://chrome.google.com/webstore/detail/context/hcplneddoadgichngfbobgpllfphdfla)  
[Font Playground](https://chrome.google.com/webstore/detail/context/hdpmpnhaoddjelneingmbnhaibbmjgno)  
[Advanced REST client](https://chrome.google.com/webstore/detail/context/hgmloofddffdnphfgcellkdfbfbjeloo)  
[Eye Dropper](https://chrome.google.com/webstore/detail/context/hmdcmlfkchdmnmnmheododdhjedfccka)  
[ExtensionJetBrains IDE Support](https://chrome.google.com/webstore/detail/context/hmhgeddbohgjknpmjagkdomcpobmllji)  
[Appspector](https://chrome.google.com/webstore/detail/context/homgcnaoacgigpkkljjjekpignblkeae)  
[Web Developer Checklist](https://chrome.google.com/webstore/detail/context/iahamcpedabephpcgkeikbclmaljebjp)  
[Live HTTP Headers](https://chrome.google.com/webstore/detail/context/iaiioopjkcekapmldfgbebdclcnpgnlo)  
[AngularJS Batarang](https://chrome.google.com/webstore/detail/context/ighdmehidhipcmcojjgiloacoafjmpfk)  
[jquery-injector](https://chrome.google.com/webstore/detail/context/indebdooekgjhkncmgbkeopjebofdoid)  
[WhatFont](https://chrome.google.com/webstore/detail/context/jabopobgcpjmedljpbcaablpmlmfcogm)  
[Chremacs](https://chrome.google.com/webstore/detail/context/kglkomofdfeolfjjnmhdpkadaildaogd)  
[Window Resizer](https://chrome.google.com/webstore/detail/context/kkelicaakdanhinjdeammmilcgefonfh)  
[Sketchpad](https://chrome.google.com/webstore/detail/context/kkghjbajgkcialbbimbifdcjilhcgoim)  
[Color Sphere!](https://chrome.google.com/webstore/detail/context/knomilfbnhpkmibhmleppnkmcempglag)  
[Hosts Manager](https://chrome.google.com/webstore/detail/context/kpfmckjjpabojdhlncnccfhkfhbmnjfi)  
[IcoMoon](https://chrome.google.com/webstore/detail/context/kppingdhhalimbaehfmhldppemnmlcjd)  
[IP Address and Domain Information](https://chrome.google.com/webstore/detail/context/lhgkegeccnckoiliokondpaaalbhafoa)  
[JSON Editor](https://chrome.google.com/webstore/detail/context/lhkmoheomjbkfloacpgllgjcamhihfaj)  
[Lightshot](https://chrome.google.com/webstore/detail/context/mbniclmhobmnbdlbpiphghaielnnpgdp)  
[App Runtime for Chrome](https://chrome.google.com/webstore/detail/context/mfaihdlpglflfgpfjcifdjdjcckigekc)  
[Tailor](https://chrome.google.com/webstore/detail/context/mfakmogheanjhlgjhpijkhdjegllgenf)  
[HostAdmin App](https://chrome.google.com/webstore/detail/context/mfoaclfeiefiehgaojbmncmefhdnikeg)  
[Parallax Background Builder](https://chrome.google.com/webstore/detail/context/mklkemobgbjfgpnhfbdbainmenjanpbe)  
[Text](https://chrome.google.com/webstore/detail/context/mmfbcljfglbokpmkimbfghdkjmjhdgbg)  
[HTTP/2 and SPDY indicator](https://chrome.google.com/webstore/detail/context/mpbpobfflnpcgagjijhmgnchggcjblin)  
[Poe: Markdown Editor](https://chrome.google.com/webstore/detail/context/mpghdlgejmakmgbigejnjnmgdjaddhje)  
[Chrome MySQL Admin](https://chrome.google.com/webstore/detail/context/ndgnpnpakfcdjmpgmcaknimfgcldechn)  
[PrettyPrint](https://chrome.google.com/webstore/detail/context/nipdlgebaanapcphbcidpmmmkcecpkhg)  
[Color Picker and Converter](https://chrome.google.com/webstore/detail/context/ofkcpbjmhcdipbhcdfechmckpaofdjlf)  
[Palette for Chrome](https://chrome.google.com/webstore/detail/context/oolpphfmdmjbojolagcbgdemojhcnlod)  
[Proxy SwitchyOmega](https://chrome.google.com/webstore/detail/context/padekgcemlokbadohgkifijomclgjgif)  
[Zed Code Editor](https://chrome.google.com/webstore/detail/context/pfmjnmeipppmcebplngmhfkleiinphhp)  
[Secure Shell](https://chrome.google.com/webstore/detail/context/pnhechapfaindjhompbnflcldabbghjo)  
[Chrome Dev Editor](https://chrome.google.com/webstore/detail/context/pnoffddplpippgcfjdhbmhkofpnaalpg)  
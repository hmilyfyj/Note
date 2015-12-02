title: Hexo美化のSidebar滚动
date: 2015-11-29 16:27:43
tags: [折腾,js]
categories: 折腾
---

为了阅读时可以方便查看目录，想让 目录（即sidebar）随页面滚动，在此记录实现方法。

![](http://7xnocp.com1.z0.glb.clouddn.com/15-11-29/36009422.jpg)

<!--more--> 

### 实现滚动：

#### 方法1：

    <script type="text/javascript">   
    $(function() {
        var $sidebar   = $("#sidebar"),   
            $window    = $(window),   
            offset     = $sidebar.offset(),   
            topPadding = 15;   
      
        $window.scroll(function() {   
            if ($window.scrollTop() > offset.top) {   
                $sidebar.stop().animate({   
                    marginTop: $window.scrollTop() - offset.top + topPadding   
                });   
            } else {   
                $sidebar.stop().animate({   
                    marginTop: 0   
                });   
            }   
        });   
      
    });   
    </script> 


 #### 方法2：

    $.fn.smartFloat = function() {  
        var position = function(element) {  
            var top = element.position().top, pos = element.css("position");  
            $(window).scroll(function() {  
                var scrolls = $(this).scrollTop();  
                if (scrolls > top) {  
                    if (window.XMLHttpRequest) {  
                        element.css({  
                            position: "fixed",  
                            top: 0  
                        });      
                    } else {  
                        element.css({  
                            top: scrolls  
                        });      
                    }  
                }else {  
                    element.css({  
                        position: "absolute",  
                        top: top  
                    });      
                }  
            });  
        };  
        return $(this).each(function() {  
            position($(this));                           
        });  
    };  
    $("#float").smartFloat();  


[
详细介绍][1]

### 判别是否为手机

    var isMobile = navigator.userAgent.match(/iphone|android|phone|mobile|wap|netfront|x11|java|opera mobi|opera mini|ucweb|windows ce|symbian|symbianos|series|webos|sony|blackberry|dopod|nokia|samsung|palmsource|xda|pieplus|meizu|midp|cldc|motorola|foma|docomo|up.browser|up.link|blazer|helio|hosin|huawei|novarra|coolpad|webos|techfaith|palmsource|alcatel|amoi|ktouch|nexian|ericsson|philips|sagem|wellcom|bunjalloo|maui|smartphone|iemobile|spice|bird|zte-|longcos|pantech|gionee|portalmmm|jig browser|hiptop|benq|haier|^lct|320x320|240x320|176x220/i) != null;
        if (isMobile) {
            return true;
        }


### 整合代码

#### 修改 themes\feng\layout\_layout.swig

    <script type="text/javascript">
    $(document).ready(
    	var isMobile = navigator.userAgent.match(/iphone|android|phone|mobile|wap|netfront|x11|java|opera mobi|opera mini|ucweb|windows ce|symbian|symbianos|series|webos|sony|blackberry|dopod|nokia|samsung|palmsource|xda|pieplus|meizu|midp|cldc|motorola|foma|docomo|up.browser|up.link|blazer|helio|hosin|huawei|novarra|coolpad|webos|techfaith|palmsource|alcatel|amoi|ktouch|nexian|ericsson|philips|sagem|wellcom|bunjalloo|maui|smartphone|iemobile|spice|bird|zte-|longcos|pantech|gionee|portalmmm|jig browser|hiptop|benq|haier|^lct|320x320|240x320|176x220/i) != null;
    	if (!isMobile) {
    		$(function() {
    		    var $sidebar   = $("#sidebar"),
    		        $window    = $(window),
    		        $header    = $("#header-inner"),
    		        offset     = $sidebar.offset(),
    		        topPadding = 15;
    
    		    $window.scroll(function() {
    		        if ($window.scrollTop() > offset.top) {
    		        	//console.log($window.scrollTop());
    		        	console.log(offset.top);
    		            $sidebar.stop().animate({
    		                //marginTop: $window.scrollTop() - offset.top + topPadding
    		                marginTop: $window.scrollTop()  + topPadding
    		            });
    		        } else {
    		            $sidebar.stop().animate({
    		                marginTop: offset.top
    		            });
    		        }
    		    });
    
    		});
    		}
    );
    </script>


  [1]: http://zmingcx.com/jquery-the-the-the-the-the-the-sidebar-scroll-with-the-window.html
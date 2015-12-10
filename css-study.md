title: 前端常用
date: 2015-12-10 20:21
tags: [html,js,css]
categories: 前端学习
---

### 给所有图片加点击事件

     function AddAClickEvent()  
      {  
          var objs = document.getElementsByTagName("a");             
          for(var i=0;i<objs.length;i++)  
          {  
    						
    		  objs[i].href="javascript:void(0)";
              objs[i].onclick=function() 
              {  
                  alert('hello');
                  return false;
              }  
              objs[i].style.cursor = "pointer";  
          }  
      }   


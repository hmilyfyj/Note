title: 前端常用
date: 2015-12-10 20:21
tags: [html,js,css]
categories: 前端学习
---

### 给所有图片加点击事件

     function AddImgClickEvent()  
      {  
          var objs = document.getElementsByTagName("img");             
          for(var i=0;i<objs.length;i++)  
          {  
              objs[i].onclick=function() 
              {  
                  alert('hello');
                  return false;
              }  
              objs[i].style.cursor = "pointer";  
          }  
      }   


---
title: cssStickyFooters
date: 2018-11-13 14:42:29
tags:
---
# CSS Sticky footers

最近写过这样的一个效果 ，它可以概括如下：如果页面内容不够长的时候，页脚块粘贴在视窗底部；如果内容足够长时，页脚块会被内容向下推送。

有两个方式，各有优缺点

1. 利用flex布局，简单，代码少，兼容性问题

   首先页面结构

   ```html
   <body>
       <div class='main'>
           <p>
               css sticky footers
           </p>
       </div>
       <div class='footer'>
           底部
       </div>
   </body>
   ```

   css样式

   ```css
   body{
       margin: 0;
       display: flex;
       flex-flow: column;
       min-height: 100vh;
   }
   .main{
       flex: 1;
   }
   .footer{
       height: 50px;
       background: #999;
   }
   ```

2. 利用min-height + padding-bottom + margin-top 代码交复杂，兼容性好

   页面结构

   ```html
   <div class='wrapper clearfix'>
       <div class='main'>
           <p>
               css sticky footers
           </p>
       </div>
   </div>
   <div class='footer'>
       底部
   </div>
   ```

   css 样式

   ```less
   body,html{
       margin: 0;
       height: 100%;
   }
   .wrapper{
       min-height: 100%;
       .main{
           padding-bottom: 50px;
       }
   }
   .footer{
       position: relative;
       height: 50px;
       clear: both;
       margin-top: -50px;
       background: #999;
   }

   .clearfix{
       display: inline-block;
       &:after{
           content: '';
           display: block;
           height: 0;
           line-height: 0;
           clear: both;
   	}
   }
   ```

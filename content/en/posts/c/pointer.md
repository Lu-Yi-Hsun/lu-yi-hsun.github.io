---
title: "指標篇"
date: 2021-01-29T18:00:00+08:00
description: "學習筆記"
draft: false
enableToc: true
enableTocContent: false
authorEmoji: 👻
tags: 
- 你所不知道的 C 語言
---

# 直接到跳到記憶體位址執行


When this machine was switched on, the hardware would call the subroutine whose address was stored in location 0.

# 法1
```
(*(void(*)())0)();
```

# 法2
(void(*)())0=0轉成function pointer

<iframe width="800px" height="200px" src="https://gcc.godbolt.org/e#g:!((g:!((g:!((h:codeEditor,i:(filename:'1',fontScale:14,fontUsePx:'0',j:1,lang:___c,selection:(endColumn:19,endLineNumber:2,positionColumn:19,positionLineNumber:2,selectionStartColumn:19,selectionStartLineNumber:2,startColumn:19,startLineNumber:2),source:'int+main()+%7B%0A(*(void(*)())0)()%3B%0A%7D'),l:'5',n:'0',o:'C+source+%231',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((h:compiler,i:(compiler:cg72,filters:(b:'0',binary:'1',commentOnly:'0',demangle:'1',directives:'0',execute:'1',intel:'1',libraryCode:'0',trim:'1'),flagsViewOpen:'1',fontScale:14,fontUsePx:'0',j:1,lang:___c,libs:!(),options:'-O2',selection:(endColumn:1,endLineNumber:1,positionColumn:1,positionLineNumber:1,selectionStartColumn:1,selectionStartLineNumber:1,startColumn:1,startLineNumber:1),source:1,tree:'1'),l:'5',n:'0',o:'x86-64+gcc+7.2+(C,+Editor+%231,+Compiler+%231)',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0')),l:'2',n:'0',o:'',t:'0')),version:4"></iframe>
人看得懂的寫法是下面
宣告一個function pointer
指向位置0
並且執行它
<iframe width="800px" height="200px" src="https://gcc.godbolt.org/e?readOnly=true#g:!((g:!((g:!((h:codeEditor,i:(filename:'1',fontScale:14,fontUsePx:'0',j:1,lang:___c,selection:(endColumn:1,endLineNumber:2,positionColumn:1,positionLineNumber:2,selectionStartColumn:1,selectionStartLineNumber:2,startColumn:1,startLineNumber:2),source:'int+main()+%7B%0Avoid+(*funcptr)()%3B%0Afuncptr%3D0%3B%0A(*+funcptr)()%3B%0A%7D'),l:'5',n:'0',o:'C+source+%231',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((h:compiler,i:(compiler:cg72,filters:(b:'0',binary:'1',commentOnly:'0',demangle:'1',directives:'0',execute:'1',intel:'1',libraryCode:'0',trim:'1'),flagsViewOpen:'1',fontScale:14,fontUsePx:'0',j:1,lang:___c,libs:!(),options:'-O2',selection:(endColumn:1,endLineNumber:1,positionColumn:1,positionLineNumber:1,selectionStartColumn:1,selectionStartLineNumber:1,startColumn:1,startLineNumber:1),source:1,tree:'1'),l:'5',n:'0',o:'x86-64+gcc+7.2+(C,+Editor+%231,+Compiler+%231)',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0')),l:'2',n:'0',o:'',t:'0')),version:4"></iframe>
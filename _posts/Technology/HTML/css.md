### 滚动条

```css
/**滚动条**/
    #continer::-webkit-scrollbar {
        /*滚动条整体样式*/

    width: 0px;     
    /* width: 2px;      */
    /*高宽分别对应横竖滚动条的尺寸*/

    height: 0px;
    /* height: 2px; */

    }
    #continer::-webkit-scrollbar-thumb {
        /*滚动条里面小方块*/

    border-radius: 5px; /*圆角边界*/

    /* -webkit-box-shadow: inset 0 0 5px rgba(0,0,0,0.2); */

    background: gray;

    }

    #continer::-webkit-scrollbar-track {
        /*滚动条里面轨道*/

    /* -webkit-box-shadow: inset 0 0 5px rgba(0,0,0,0.2); */

    border-radius: 5px;

    background: transparent;

    }
```


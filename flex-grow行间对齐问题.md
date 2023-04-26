## 问题描述
在使用flex布局时，多行flex之间往往有上下对齐的需求，如图：

<img width="1008" alt="企业微信20230426-164328@2x" src="https://user-images.githubusercontent.com/39622507/234521151-f8ee2553-5c0a-45b9-8527-8c31b57486e4.png">

然而直接使用flex-grow属性时，上下行间往往会出现一些错位的情况。

此时需要使用flex-basis属性来解决这个问题，flex-basis属性可以设置元素的初始大小。

代码示例如下：
```scss
.meta-home-card-list {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
  .meta-home-card {
    background: #FFFFFF;
    border-radius: 4px;
    border: 1px solid rgba(213, 216, 219, 0.5);
    margin-right: 10px;
    &:last-child {
      margin-right: 0;
    }
  }
  &.top-list {
    height: 120px;
    .meta-home-card.top-card {
      width: 0;
      flex-grow: 1;
      flex-basis: 0;
    }
  }
  &.middle-list {
    height: 300px;
    .meta-home-card.middle-card1 {
      width: 0;
      flex-grow: 2;
      flex-basis: 10px;
    }
    .meta-home-card.middle-card2 {
      width: 0;
      flex-grow: 3;
      flex-basis: 20px;
    }
  }
  &.bottom-list {
    height: 200px;
    .meta-home-card.bottom-card1 {
      width: 0;
      flex-grow: 4;
      flex-basis: 10px;
    }
    .meta-home-card.bottom-card2 {
      width: 0;
      flex-grow: 3;
      flex-basis: 5px;
    }
    .meta-home-card.bottom-card3 {
      width: 0;
      flex-grow: 3;
      flex-basis: 5px;
    }
  }
}
```



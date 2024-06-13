# css

css: structure

![image-20230801154353493](https://article.biliimg.com/bfs/article/1167f4e99587f25f8fff3d2d67b3152f3316cb3f.png)

```
inline <h style="">
Internal <style>
External
```

## font

## border

![image-20230801163322907](https://article.biliimg.com/bfs/article/408795b113bf7693482a0c4b9971828a4e76ee73.png)

```
border-style: outset
border-width:5px
border-color:
border-raddius:10px
padding:5px
```

## background

![image-20230801164629926](https://article.biliimg.com/bfs/article/dd4a77ef367868d9f814e091287d1056b735c7f8.png)

```
background: line-gradient(skyblue,lightgreen)
```

## margins

![image-20230801165236357](https://article.biliimg.com/bfs/article/cd73578e643f27795b6a71264b211bbcae313a6a.png)

```
div
```

## float

![image-20230801171525561](https://article.biliimg.com/bfs/article/5804b14cf23fc4b4a11f0dc493f99be7fdcee946.png)

## position

```
position：fixed
top：
bottom：
left：
```

## psudo classes

> 伪类是选择器的一种，它用于选择处于特定状态的元素，比如当它们是这一类型的第一个元素时，或者是当鼠标指针悬浮在元素上面的时候。它们表现得会像是你向你的文档的某个部分应用了一个类一样，帮你在你的标记文本中减少多余的类，让你的代码更灵活、更易于维护。

![image-20230801175949470](https://article.biliimg.com/bfs/article/7c09e15710c7be7f7692b45120ee9c0cf5ca5722.png)

[把伪类和伪元素组合起来](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/Building_blocks/Selectors/Pseudo-classes_and_pseudo-elements#把伪类和伪元素组合起来)

```css
article p:first-child::first-line {
  font-size: 120%;
  font-weight: bold;
}
```

## shadows

![image-20230801180746537](https://article.biliimg.com/bfs/article/a7ac4e2cd3509c92d00d00e9c0bbb09a334139b5.png)

## transform

![image-20230801184418954](https://article.biliimg.com/bfs/article/ff999d69e36986313b60da1f2757305929492c04.png)

## Animation

```
Animation： 3s linear 0s infinite running mySlide
@Keyframes mySlide{
    from{margin-left:100%}
    to{margin-left:0%;}
}
```

![image-20230801185138527](https://article.biliimg.com/bfs/article/836d84d5f24bd55f656343df0b04689dcfad419a.png)

![image-20230801185228231](https://article.biliimg.com/bfs/article/01b9d96f3d08574c49862a651e63b959dab125f3.png)

## icons

https://xhs01.apiapi.top/api/v1/client/subscribe?token=541e2bac3ee0bdd520b0eeaf9057a211
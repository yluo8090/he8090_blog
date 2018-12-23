---
title: iOS动画
date: 2016-07-18 11:36:46
tags:
  - iOS动画
categories: 技术分享
---
> iOS动画分为基础动画和核心动画

iOS中关键动画如下
- CALayer动画
- CABasicAnimation 基础动画
- CAKeyFrameAnimation 关键帧动画
- CaTransition 过度、转场动画
- CAAnimationGroup 组合动画 <br>
注：以上动画必须加载到layer上执行

<!-- more --> 
***
- 基础动画1

```
- (void)animation1{
    [UIView beginAnimations:@"第一个动画" context:nil];

    //配置动画属性
    [UIView setAnimationCurve:UIViewAnimationCurveLinear];//速度，UIViewAnimationCurveLinear 匀速
    [UIView setAnimationDuration:3];//动画时间
//    [UIView setAnimationDelay:2];//延迟
    [UIView setAnimationDelegate:self];//动画代理方法
    [UIView setAnimationRepeatCount:3];//重复次数
//    [UIView setAnimationWillStartSelector:@selector(actionTest)];//动画将要开始的时候执行方法

    //设置变化：位置，尺寸，颜色等
    self.animationView.frame = CGRectMake(200, 200, 200, 200);
    self.animationView.backgroundColor = [UIColor yellowColor];
}
```
- 基础动画2（转场动画）

```
//UIView动画2，转场
- (void)animation2{
    [UIView beginAnimations:@"tranform" context:nil];
    [UIView setAnimationDuration:2];
    [UIView setAnimationRepeatCount:10];
    [UIView setAnimationTransition:UIViewAnimationTransitionCurlDown forView:self.animationView cache:YES];//翻转方向

    [UIView commitAnimations]; //转场动画必须调用commit方法

}
```

- 基础动画3（block动画）

```
//UIView动画3.block
- (void)animation3{
    [UIView animateWithDuration:3 animations:^{
        [UIView setAnimationRepeatCount:100];
        self.animationView.center = self.view.center;
    } completion:^(BOOL finished) {
        self.animationView.frame = CGRectMake(100, 100, 100, 100);
    }];
}

```
- CAAnimation动画

```
//CAAnimation动画
- (void)animation_CABasicAnimation{
    //基础动画配置三要素：duration动画时间  fromeValue开始尺寸  toValue动画达到尺寸
    //注意：layer没有的属性是不能作为KeyPath。如：frame

    CABasicAnimation *basicAnimation = [CABasicAnimation animationWithKeyPath:@"bounds"]; //创建动画对象
    basicAnimation.duration = 0.05;//动画时间
    basicAnimation.repeatCount = INT_MAX;//重复执行
    basicAnimation.fromValue = [NSValue valueWithCGRect:self.animationView.bounds];//动画开始的尺寸
    basicAnimation.toValue = [NSValue valueWithCGRect:CGRectMake(0, 0, 400, 400)];//动画要达到的尺寸

    [self.animationView.layer addAnimation:basicAnimation forKey:@"animation_CABasicAnimation"];
}
```
- 帧动画

```
//帧动画
- (void)animtaion_CAKeyfrmaeAnimation{
    CAKeyframeAnimation *keyfrmae = [CAKeyframeAnimation animationWithKeyPath:@"backgroundColor"];
    keyfrmae.duration = 4;
    keyfrmae.values = @[(id)[UIColor blueColor].CGColor,(id)[UIColor orangeColor].CGColor,(id)[UIColor yellowColor].CGColor,(id)[UIColor greenColor].CGColor]; //每一帧颜色
    keyfrmae.keyTimes = @[@(0.25),@(0.5),@(0.75),@(1)];//每一帧动画执行时间

    [self.animationView.layer addAnimation:keyfrmae forKey:@"keyFrame"];
}
```

- 过度动画

```
//过度动画
- (void)animation_CATransition{
    //设置 时间、样式、方向
    CATransition *trans = [CATransition animation];
    trans.duration = 0.1;
    trans.repeatCount = INT_MAX;
    trans.type = kCATransitionPush;//控制样式；
    trans.subtype = kCATransitionFromRight;//控制方向

    [self.animationView.layer addAnimation:trans forKey:@"trans"];
}
```

- 动画组（将已定义的核心动画装入一个动画执行组一起执行，注意动画组的执行时间）

```
- (void)CAAnimationGroup{

    //1
    CABasicAnimation *basicAnimation = [CABasicAnimation animationWithKeyPath:@"bounds"]; //创建动画对象
    basicAnimation.duration = 0.1;//动画时间
    basicAnimation.fromValue = [NSValue valueWithCGRect:self.animationView.bounds];//动画开始的尺寸
    basicAnimation.toValue = [NSValue valueWithCGRect:CGRectMake(0, 0, 400, 400)];//动画要达到的尺寸

    //2
    CAKeyframeAnimation *keyfrmae = [CAKeyframeAnimation animationWithKeyPath:@"backgroundColor"];
    keyfrmae.duration = 0.5;
    keyfrmae.values = @[(id)[UIColor blueColor].CGColor,(id)[UIColor orangeColor].CGColor,(id)[UIColor yellowColor].CGColor,(id)[UIColor greenColor].CGColor]; //每一帧颜色
    keyfrmae.keyTimes = @[@(0.25),@(0.5),@(0.75),@(1)];//每一帧动画执行时间


    CAAnimationGroup *group = [CAAnimationGroup animation];
    group.animations = @[basicAnimation,keyfrmae];
    group.duration = 0;
    group.repeatCount = INT_MAX;

    [self.animationView.layer addAnimation:group forKey:@"group"];
}
```

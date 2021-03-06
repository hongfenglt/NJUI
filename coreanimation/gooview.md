# GooView

#7-QQ粘性效果

实现思路:

###1.自定义大圆控件（UIButton）可以显示背景图片，和文字

###2.让大圆控件随着手指移动而移动

-	注意不能根据形变修改大圆的位置，只能通过center，因为全程都需要用到中心点计算。


###3.在拖动的时候，添加一个小圆控件在原来大圆控件的位置

-	注意这个小圆控件并不会随着手指移动而移动，因此应该添加到父控件上
-	一开始设置中心点和尺寸和大圆控件一样。
-	随着大圆拖动，小圆半径不断减少，可以根据两个圆心的距离，随便生成一段比例，随着圆心距离增加，圆心半径不断减少。

```
		// 计算小圆半径：随机搞个比例，随着圆心距离增加，圆心半径不断减少。
        CGFloat smallRadius = _circleR2 - d / 10;
```
- 每次小圆改变，需要重新设置小圆的尺寸和圆角半径。

###4.粘性效果
-	就是在两圆之间绘制一个形变矩形，描述形变矩形路径。
-	`这里大致介绍下计算思路，不需要太纠结`
-	这里需要用到CAShapeLayer,可以根据一个路径，生成一个图层，展示出来。把形变图层添加到父控件并且显示在小圆图层下就OK了。因为所有计算出来的点，都是基于父控件。
-	`注意：这里不能用绘图，因为绘图内容只要超过当前控件尺寸就不会显示，但是当前形变矩形必须显示在控件之外`

###5.粘性业务逻辑处理
-	当圆心距离超过100，就不需要描述形变矩形（并且把之前的形变矩形移除父层），小圆也需要隐藏。

-	没有超过100，则相反。

### 6.手指停止拖动业务逻辑
- 判断下圆心是否超过100，超过就播放爆炸效果，添加个UIImageView在当前控件上，并且需要取消控制器view的自动布局。

```
	// 取消Autoresizing转自动布局
    self.view.translatesAutoresizingMaskIntoConstraints = NO;

```
- 没有超过，就还原。

## `计算2各圆心的距离 和描述2个圆之间的不规则矩形`

```objc

// 计算两个圆心之间的距离
- (CGFloat)circleCenterDistanceWithBigCircleCenter:(CGPoint)bigCircleCenter smallCircleCenter:(CGPoint)smallCircleCenter
{
    CGFloat offsetX = bigCircleCenter.x - smallCircleCenter.x;
    CGFloat offsetY = bigCircleCenter.y - smallCircleCenter.y;

    return  sqrt(offsetX * offsetX + offsetY * offsetY);
}

// 描述两圆之间一条矩形路径
- (UIBezierPath *)pathWithBigCirCleView:(UIView *)bigCirCleView  smallCirCleView:(UIView *)smallCirCleView
{
    CGPoint bigCenter = bigCirCleView.center;
    CGFloat x2 = bigCenter.x;
    CGFloat y2 = bigCenter.y;
    CGFloat r2 = bigCirCleView.bounds.size.width / 2;

    CGPoint smallCenter = smallCirCleView.center;
    CGFloat x1 = smallCenter.x;
    CGFloat y1 = smallCenter.y;
    CGFloat r1 = smallCirCleView.bounds.size.width / 2;



    // 获取圆心距离
    CGFloat d = [self circleCenterDistanceWithBigCircleCenter:bigCenter smallCircleCenter:smallCenter];

    CGFloat sinθ = (x2 - x1) / d;

    CGFloat cosθ = (y2 - y1) / d;

    // 坐标系基于父控件
    CGPoint pointA = CGPointMake(x1 - r1 * cosθ , y1 + r1 * sinθ);
    CGPoint pointB = CGPointMake(x1 + r1 * cosθ , y1 - r1 * sinθ);
    CGPoint pointC = CGPointMake(x2 + r2 * cosθ , y2 - r2 * sinθ);
    CGPoint pointD = CGPointMake(x2 - r2 * cosθ , y2 + r2 * sinθ);
    CGPoint pointO = CGPointMake(pointA.x + d / 2 * sinθ , pointA.y + d / 2 * cosθ);
    CGPoint pointP =  CGPointMake(pointB.x + d / 2 * sinθ , pointB.y + d / 2 * cosθ);

    UIBezierPath *path = [UIBezierPath bezierPath];

    // A
    [path moveToPoint:pointA];

    // AB
    [path addLineToPoint:pointB];

    // 绘制BC曲线
    [path addQuadCurveToPoint:pointC controlPoint:pointP];

    // CD
    [path addLineToPoint:pointD];

    // 绘制DA曲线
    [path addQuadCurveToPoint:pointA controlPoint:pointO];

    return path;

}

```






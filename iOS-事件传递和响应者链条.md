# iOS-事件传递和响应者链条


## 事件传递和响应者链条

在iOS中只有继承UIResponder的对象才能够接收并处理事件，UIResponder 是所有响应对象的基类，在UIResponder类中定义了处理上述各种事件的接口。我们熟悉的 UIApplication、 UIViewController、 UIWindow 和所有继承自UIView的UIKit类都直接或间接的继承自UIResponder，所以它们的实例都是可以构成响应者链的响应者对象，首先我们通过一张图来简单了解一下事件的传递以及响应.

![Image text](https://upload-images.jianshu.io/upload_images/1197641-ca37721f0f3719bf.png)


## 响应者链条

响应者链条就是由多个响应者对象连接起来的链条，它的作用就是让我们能够清楚的看见每个响应者之间的联系，并且可以让一个时间多个对象处理.

iOS系统检测到手指触摸(Touch)操作时会将其打包成一个UIEvent对象，并放入当前活动Application的事件队列，单例的UIApplication会从事件队列中取出触摸事件并传递给单例的UIWindow来处理，UIWindow对象首先会使用hitTest:withEvent:方法寻找此次Touch操作初始点所在的视图(View)，即需要将触摸事件传递给其处理的视图(最合适来处理的控件)，这个过程称之为hit-test view。

那么什么是最适合来处理事件的控件?
1. 自己能响应触摸事件
2. 触摸点在自己身上
3. 从后往前递归遍历子控件, 重复上两步
4. 如果没有符合条件的子控件, 那么就自己最合适处理


## 两个重要的响应方法(UIView的)

1. hit-test view:事件传递给控件的时候， 就会调用该方法，去寻找最合适的view并返回看可以响应的view
```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event
{
    // 1.如果控件不允许与用用户交互,那么返回nil
    if (self.userInteractionEnabled == NO || self.alpha <= 0.01 || self.hidden == YES){
        return nil;
    }
    // 2. 如果点击的点在不在当前控件中,返回nil
    if (![self pointInside:point withEvent:event]){
        return nil;
    }
    // 3.从后往前遍历每一个子控件
    for(int i = (int)self.subviews.count - 1 ; i >= 0 ;i--){
        // 3.1获取一个子控件
        UIView *childView = self.subviews[i];
        // 3.2当前触摸点的坐标转换为相对于子控件触摸点的坐标
        CGPoint childP = [self convertPoint:point toView:childView];
        // 3.3判断是否在在子控件中找到了更合适的子控件(递归循环)
        UIView *fitView = [childView hitTest:childP withEvent:event];
        // 3.4如果找到了就返回
        if (fitView) {
            return fitView;
        }
    }
    // 4.没找到,表示没有比自己更合适的view,返回自己
    return self;

```

2. pointInside: 该方法判断触摸点是否在控件身上，是则返回YES，否则返回NO，point参数必须是方法调用者的坐标系.
```
-(BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event{
    return NO;
}
```


## 涉及的面试问题

1. 事件的传递方向: 事件传递是从上自下传递，响应是从下到上，所谓的上就是父视图而已，也就是离窗口最近的.

2. 穿透控件:

如果我们不想让某个视图响应事件，只需要重载 PointInside:withEvent:方法，让此方法返回NO就行了.

若是view上有view1，view1上有view2，点击view2，view2自己响应，点击view1，view1不响应，只有view响应，也就是隔层传递。
或者例如放大控件响应区域，view上有n个子视图，点击其中一个让另一个来响应等等，都是可以通过重载pointInside来达到目的.

```
/*
 重载view1的此方法，如果点在自己身上，且子控件中有最合适的响应者，就返回对应子控件，否则就不响应，并将该事件随着响应者链条往回传递，交给上一个响应者来处理. (即调用super的touches方法)
 
 谁是上一个响应者?
 1. 如果view的控制器存在，就传递给控制器；如果控制器不存在，则将其传递给它的父视图
 2. 在视图层次结构的最顶级视图，如果也不能处理收到的事件或消息，则其将事件传递给window对象进行处理
 3. 如果window对象也不处理，则其将事件或消息传递给UIApplication对象
 4. 如果UIApplication也不能处理该事件或消息，则将其丢弃
 
*/
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event{
    
    CGRect frame = CGRectMake(0, 0, self.frame.size.width, self.frame.size.height);
    BOOL value = (CGRectContainsPoint(frame, point));
    NSArray *views = [self subviews];
    for (UIView *subview in views) {
        value = (CGRectContainsPoint(subview.frame, point));
        if (value) {
            return value;
        }
    }
    return NO;
}
```

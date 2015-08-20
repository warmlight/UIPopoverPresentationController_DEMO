# UIPopoverPresentationController_DEMO
how to use UIPopoverPresentationController

***
* 如果要再ControllerA中点击某个按钮弹出模态视图显示ControllerB,那么进行主动弹出的controllerA就是presenting view controller，从名字可看出“主动”。而被弹出的controllerB是presented view controller。<br>
 <img src="http://ac-3xs828an.clouddn.com/da212933d457ca2e.PNG" width = "300" height = "500" alt="example" align=center />
*  `UIPopoverPresentationController`是`UIViewController`的一个属性，所以并不需要你特地去建立一个`UIPopoverPresentationController`来进行操作，而应该建立一个`UIViewController`。<br>
* 相关属性<br>
１．`sourceRect`：指定箭头所指区域的矩形框范围，以sourceview的左上角为坐标原点
<br>
２．`permittedArrowDirections`：箭头方向<br>
３．`sourceView`：sourceRect以这个view的左上角为原点<br>
４．`barButtonItem`：若有navigationController,并且从`right/leftBarButtonItem`点击后出现popover,则可以把`right/leftBarButtonItem`看做上面说的sourceView.默认箭头指向up，亲测下来up是最合适的方向，所以在这种情况下可以不设置箭头方向。<br>

***
代码实现的具体效果：点击button出现popover,点击相应的颜色，popover消失，同时背景色会相应改变。<br>
PopoverController:被弹出的Controller<br>
ViewController:主动弹出的Controller<br>

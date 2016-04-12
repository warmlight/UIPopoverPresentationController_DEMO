# UIPopoverPresentationController简单使用

***
　在之前如果想要在iphone上实现popover的效果需要自定义view,在iOS8中提供了UIPopoverPresentationController在ipad和iphone两个设备上同时实现popover的效果。找了找网上具体使用的栗子比较少，加上我也是个小菜鸟，所以写着以防我白痴的记忆以后不记得了干着急。如果有不对的地方也请指正~<br>
***
* 如果要再ControllerA中点击某个按钮弹出模态视图显示ControllerB,那么进行主动弹出的controllerA就是presenting view controller，从名字可看出“主动”。而被弹出的controllerB是presented view controller。<br>
 <img src="http://ac-3xs828an.clouddn.com/da212933d457ca2e.PNG" width = "300" height = "500" alt="example" align=center />
 * `UIPopoverPresentationController`是`UIViewController`的一个属性，所以并不需要你特地去建立一个`UIPopoverPresentationController`来进行操作，而应该建立一个`UIViewController`。<br>
* 相关属性<br>
１．`sourceRect`：指定箭头所指区域的矩形框范围，以sourceview的左上角为坐标原点
<br>
２．`permittedArrowDirections`：箭头方向<br>
３．`sourceView`：sourceRect以这个view的左上角为原点<br>
４．`barButtonItem`：若有navigationController,并且从`right/leftBarButtonItem`点击后出现popover,则可以把`right/leftBarButtonItem`看做上面说的sourceView.默认箭头指向up，亲测下来up是最合适的方向，所以在这种情况下可以不设置箭头方向。

***
代码实现的具体效果：点击button出现popover,点击相应的颜色，popover消失，同时背景色会相应改变。<br>
PopoverController:被弹出的Controller<br>
ViewController:主动弹出的Controller<br>
主要代码如下：

```objective-c	
	#import "PopoverViewController.h"

	@implementation PopoverViewController
	- (void)viewDidLoad {
    [super viewDidLoad];
    self.tableView = [[UITableView alloc] initWithFrame:self.view.frame];
    [self.view addSubview:self.tableView];
    self.tableView.dataSource = self;
    self.tableView.delegate = self;
    self.tableView.scrollEnabled = NO;
    self.colorArray = [[NSMutableArray alloc] initWithObjects:@"green",@"gray", @"blue",@"purple", @"yellow", nil];
    }

	 - (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
	 //创建消息，在点击相应的颜色时发送，在ViewController中接受消息并做出相应的处理
    [[NSNotificationCenter defaultCenter] postNotificationName:@"click" object:indexPath];
	}

	//重写preferredContentSize（iOS7之后）来返回最合适的大小，如果不重写，会返回一整个tableview尽管下面一部分cell是没有内容的，重写后只会返回有内容的部分，我这里还修改了宽，让它窄一点。可以尝试注释这一部分的代码来看效果，通过修改返回的size得到你期望的popover的大小。		
	- (CGSize)preferredContentSize { 
	    if (self.presentingViewController && self.tableView != nil) {
    	    CGSize tempSize = self.presentingViewController.view.bounds.size;
        	tempSize.width = 150;
        	CGSize size = [self.tableView sizeThatFits:tempSize];  //sizeThatFits返回的是最合适的尺寸，但不会改变控件的大小
        	return size;
    	}else {
        	return [super preferredContentSize];
   		 }
	}	

	- (void)setPreferredContentSize:(CGSize)preferredContentSize{
    super.preferredContentSize = preferredContentSize;
	}
```
	

```objective-c
	#import "ViewController.h"
	#import "PopoverViewController.h"
	
	@interface ViewController ()
	@property (strong, nonatomic) UIButton *button;
	@property (strong, nonatomic) PopoverViewController *firstPopVC;
	@property (strong, nonatomic) PopoverViewController *secondPopVC;
	@property (strong, nonatomic) NSString *currentPop;
	@end
	
	@implementation ViewController
	
	- (void)viewDidLoad {
	    [super viewDidLoad];
	    self.navigationItem.rightBarButtonItem = [[UIBarButtonItem alloc] initWithTitle:@"item" style:UIBarButtonItemStylePlain target:self action:@selector(rightItemClick)];
	    
	    self.view.backgroundColor = [UIColor whiteColor];
	    _button = [[UIButton alloc] initWithFrame:CGRectMake(20, 100, 100, 40)];
	    [_button setTitle:@"button" forState:UIControlStateNormal];
	    [_button setTitleColor:[UIColor redColor] forState:UIControlStateNormal];
	    [self.view addSubview:_button];
	    [_button addTarget:self action:@selector(buttonClick:) forControlEvents:UIControlEventTouchUpInside];
	    
	    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(tableDidSelected:) name:@"click" object:nil];
	}
	
	- (void)rightItemClick{
    self.itemPopVC = [[PopoverViewController alloc] init];
    self.itemPopVC.modalPresentationStyle = UIModalPresentationPopover;
    self.itemPopVC.popoverPresentationController.barButtonItem = self.navigationItem.rightBarButtonItem;  //rect参数是以view的左上角为坐标原点（0，0）
    self.itemPopVC.popoverPresentationController.permittedArrowDirections = UIPopoverArrowDirectionUnknown; //箭头方向,如果是baritem不设置方向，会默认up，up的效果也是最理想的
    self.itemPopVC.popoverPresentationController.delegate = self;
    [self presentViewController:self.itemPopVC animated:YES completion:nil];
    } 

	//处理popover上的talbe的cell点击
	- (void)tableDidSelected:(NSNotification *)notification {
	    NSIndexPath *indexpath = (NSIndexPath *)notification.object;
	    switch (indexpath.row) {
	        case 0:
	            self.view.backgroundColor = [UIColor greenColor];
	            break;
	        case 1:
	            self.view.backgroundColor = [UIColor grayColor];
	            break;
	        case 2:
	            self.view.backgroundColor = [UIColor blueColor];
	            break;
	        case 3:
	            self.view.backgroundColor = [UIColor purpleColor];
	            break;
	        case 4:
	            self.view.backgroundColor = [UIColor yellowColor];
	            break;
	    }
	    if (self.buttonPopVC) {
	        [self.buttonPopVC dismissViewControllerAnimated:YES completion:nil];    //我暂时使用这个方法让popover消失，但我觉得应该有更好的方法，因为这个方法并不会调用popover消失的时候会执行的回调。
	        self.buttonPopVC = nil;
	        
	    }else{
	        [self.itemPopVC dismissViewControllerAnimated:YES completion:nil];
	        self.itemPopVC = nil;
	    }
	}
	
	- (void)buttonClick:(UIButton *)sender{
	    self.buttonPopVC = [[PopoverViewController alloc] init];
	    self.buttonPopVC.modalPresentationStyle = UIModalPresentationPopover;
	    self.buttonPopVC.popoverPresentationController.sourceView = _button;  //rect参数是以view的左上角为坐标原点（0，0）
	    self.buttonPopVC.popoverPresentationController.sourceRect = _button.bounds; //指定箭头所指区域的矩形框范围（位置和尺寸），以view的左上角为坐标原点
	    self.buttonPopVC.popoverPresentationController.permittedArrowDirections = UIPopoverArrowDirectionUp; //箭头方向
	    self.buttonPopVC.popoverPresentationController.delegate = self;
	    [self presentViewController:self.buttonPopVC animated:YES completion:nil];
	    }


	- (UIModalPresentationStyle)adaptivePresentationStyleForPresentationController:(UIPresentationController *)controller{
	    return UIModalPresentationNone;  //UIPopoverPresentationControllerDelegate,只有返回UIModalPresentationNone才可以让popover在手机上按照我们在preferredContentSize中返回的size显示。这是一个枚举，可以尝试换成其他的值尝试。
	}
	
	- (BOOL)popoverPresentationControllerShouldDismissPopover:(UIPopoverPresentationController *)popoverPresentationController{
	    return NO;   //no点击蒙版popover不消失， 默认yes
	}
```
PS：如果知道哪里有更好的实现，一定要告诉我呀！！！！

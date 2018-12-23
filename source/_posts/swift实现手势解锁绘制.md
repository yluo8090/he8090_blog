---
title: iOS实现手势解锁绘制
date: 2017-06-05 17:52:30
tags:
  - 滑动锁屏
  - 绘制按钮
categories: 技术分享
---
> 近期增加了object-c的具体应用，包括手势绘制，验证，更新，以及处理了一些手势绘制过程中Bug。详情见object-c实现。


- 一、swift实现

使用swift实现iOS手势锁屏，虽然在iOS客户端很少使用到滑动手势，但是有时候为了和安卓应用保持用户交互的一致性，所以有的时候还是很有必要的。
iOS客户端解锁建议使用touch ID。

swift与object-c的CGContextRef不一样，在swift中统一使用CGContext进行管理和使用。

本示例采用9*button进行绘制，关闭button交互事件，通过touchesBegan系列方法对滑动路径进行跟踪和绘制（imageView）。

以下为全部代码：
<!-- more --> 
```
import UIKit

  class TouchPasswordViewController: UIViewController {

    let screenWidth : CGFloat = UIScreen.main.bounds.size.width;
    let screenHeight : CGFloat = UIScreen.main.bounds.size.height;

    var btnArray : NSMutableArray = NSMutableArray.init(); //btn数组
    var selectToArray : NSMutableArray = NSMutableArray.init();//选中的btn数组
    var startPoint : CGPoint = CGPoint.init();//起点
    var endPoint : CGPoint = CGPoint.init();//终点
    var imageView : UIImageView = UIImageView.init();//绘制的背景



    override func viewDidLoad() {
        super.viewDidLoad()
        self.view.backgroundColor = UIColor.white;

        imageView = UIImageView.init(frame: CGRect.init(x: 0, y: 0, width: screenWidth, height: screenHeight));
        self.view.addSubview(imageView);

        for i in 0...2 {
            for j in 0...2 {
                let btn : UIButton = UIButton.init(type: UIButtonType.custom);
                btn.frame = CGRect.init(x: screenWidth / 12 + screenWidth / 3 * CGFloat(j), y: screenHeight / 3 + screenWidth / 3 * CGFloat(i), width: screenWidth / 6, height: screenWidth / 6);

                btn.setImage(UIImage.init(named: "尼罗河-4"), for: UIControlState.normal);
                btn.setImage(UIImage.init(named: "尼罗河"), for: UIControlState.selected);
                btn.isUserInteractionEnabled = false;//取消button的交互事件，否则touch会被截断。
                btnArray.add(btn);
                imageView.addSubview(btn);


            }
        }

    }


    func drawLine() -> (UIImage){
        var img : UIImage = UIImage.init();
        let color : UIColor = UIColor.init(red: 1, green: 0, blue: 0, alpha: 1);

        //绘制线条路径
        UIGraphicsBeginImageContext(imageView.frame.size);
        let context = UIGraphicsGetCurrentContext();
        context?.setLineWidth(5);
        context?.setStrokeColor(color.cgColor);
        context?.move(to: startPoint);

        for b in selectToArray {
            let btn : UIButton = b as! UIButton;
            let btnPit = btn.center;

            context?.addLine(to: btnPit);
            context?.move(to: btnPit);
        }

        context?.addLine(to: endPoint);
        context?.strokePath();

        img = UIGraphicsGetImageFromCurrentImageContext()!;
        UIGraphicsEndImageContext();

        return img;
    }


    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        let touch : UITouch = touches.first!;
        for b in btnArray {
            let btn : UIButton = b as! UIButton;
            let btnPit : CGPoint = touch.location(in: btn);
            if btn.point(inside: btnPit, with: nil) {//判断当前touch是否btn范围内，在则存入selectArray 并改变button状态
                selectToArray.add(btn);
                btn.isHighlighted = true;
                startPoint = btn.center;
            }
        }
    }


    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
        let touch : UITouch = touches.first!;
        endPoint = touch.location(in: imageView);

        for b in btnArray {
            let btn : UIButton = b as! UIButton;
            let po : CGPoint = touch.location(in: btn);

            if btn.point(inside: po, with: nil) {
                var isAdd : Bool = true;

                for b in selectToArray {
                    let selectBtn : UIButton = b as! UIButton;
                    if selectBtn == btn {
                        isAdd = false;
                        break;
                    }
                }

                if isAdd {
                    selectToArray.add(btn);
                    btn.isHighlighted = true;
                }
            }

        }
        imageView.image = self.drawLine();
    }

    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        imageView.image = nil;
        selectToArray.removeAllObjects();

        for b in btnArray {
            let btn : UIButton = b as! UIButton;
            btn.isHighlighted = false;

        }
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
    }
```

- 二、object-c实现
> object-c实现与swift逻辑一致，只是加入在实际使用过程中遇到的问题，使其更有参考性和实际应用价值。
注：代码中有部分资源是中英文适配使用的。如：NSLocalizedString

在object-c中考虑到屏幕适配会导致button图片拉伸，所以使用UIView代替UIButton。自定义了一个UIView.
1、RoundRectView.h

```
#import <UIKit/UIKit.h>

@interface RoundRectView : UIView
@property (nonatomic, strong) UIView *selectView;

@end
```

2、RoundRectView.m

```
#import "RoundRectView.h"
#import "SysConfig.h"

@interface RoundRectView ()
{
    CGFloat _width;
}

@end

@implementation RoundRectView

- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        _width = frame.size.width / 4.0;
        [self initUI];
    }
    return self;
}

- (void)initUI{
    self.backgroundColor = [UIColor whiteColor];
    self.layer.masksToBounds = YES;
    self.layer.cornerRadius = self.frame.size.width/ 2.0;
    self.layer.borderWidth = 2.0;
    self.layer.borderColor = [UIColor hexString:@"#3fb8c8"].CGColor;

    self.selectView = [[UIView alloc] initWithFrame:CGRectMake((self.frame.size.width - _width) / 2, (self.frame.size.width - _width) / 2, _width, _width)];
    self.selectView.backgroundColor = [UIColor hexString:@"#3fb8c8"];
    self.selectView.layer.masksToBounds = YES;
    self.selectView.layer.cornerRadius = _width / 2.0;
    self.selectView.hidden = YES;
    [self addSubview:self.selectView];  
}
@end
```

3、TouchUpViewController.h

```
#import <UIKit/UIKit.h>
#import "LXBaseViewController.h"

typedef enum : NSUInteger {
    TouchUnlockCreatePwd,//绘制
    TouchUnlockValidatePwd,//验证
    TouchUnlockUpdatePwd,//修改手势
} UnlockType;

@interface TouchUpViewController : LXBaseViewController

- (instancetype)initWithUnlockType:(UnlockType)type;

@end
```

4、TouchUpViewController.m

```
#import "TouchUpViewController.h"
#import "RoundRectView.h"
#import "SysConfig.h"
#import "AppDelegate.h"
#import "LXNetworkRequest.h"
#import "LXNetworkRequest+LogWithEmail.h"

#define APPDELEGATEREAL (AppDelegate *)[UIApplication sharedApplication].delegate
#define TouchUnlockPwdKey  @"touchUnlockPwdKey"

@interface TouchUpViewController ()
{
    NSMutableArray *_nomarlArray,*_selectArray;
    CGFloat _width,_height;
    UIImageView *_imageView;
    CGPoint _startPoint,_endPoint;
    NSMutableString *_password;
    NSString *_oldPassword;
    UnlockType _unlockType;
    NSInteger _errorCount;

    UIImageView *_headerImageView;
    UILabel *_nickNameLabel,*_tipLabel;
    UIButton *_loginBtn;

    BOOL isUpdatePwd;

    NSInteger _steps;//1 验证密码 2：创建第一遍   3：创建第二遍
}
@end

@implementation TouchUpViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.title = NSLocalizedString(@"Draw Pattern", nil);
    self.view.backgroundColor = [UIColor whiteColor];
    [self initUI];
}

- (void)comeBack:(id)sender
{
    [self.navigationController popViewControllerAnimated:YES];
}

- (instancetype)initWithUnlockType:(UnlockType)type{

    self = [super init];
    if (self) {
        _unlockType = type;
    }
    return self;
}

- (void)initUI{

    NSUserDefaults *def = [NSUserDefaults standardUserDefaults];
    NSString *savePwd = [def objectForKey:TouchUnlockPwdKey];

    _width = [UIScreen mainScreen].bounds.size.width;
    _height = [UIScreen mainScreen].bounds.size.height;

    _password = [NSMutableString string];
    _nomarlArray = [NSMutableArray array];
    _selectArray = [NSMutableArray array];

    _imageView = [[UIImageView alloc] initWithFrame:CGRectMake(0, 0, _width, _height)];
    [self.view addSubview:_imageView];


    CGFloat leftSpace = fitScreenWidth(40);
    CGFloat wd = (_width - leftSpace * 2) /8.0;

    NSInteger index = 1;
    for (int i = 0; i < 3; i ++) {
        for (int j = 0; j < 3; j ++) {

            CGRect rect = CGRectMake(leftSpace + wd * 3 * j, _height / 3.0 + wd * 3 * i, wd * 2, wd * 2);
            RoundRectView *ve = [[RoundRectView alloc] initWithFrame:rect];

            ve.tag = index ++;

            [_nomarlArray addObject:ve];
            [_imageView addSubview:ve];
        }
    }


    //    -   -  -  - -  header - - - - - - -  - -
    _headerImageView = [[UIImageView alloc] initWithFrame:CGRectMake((_width - fitScreenWidth(50)) / 2, fitScreenWidth(60), fitScreenWidth(50), fitScreenWidth(50))];
    _headerImageView.backgroundColor = [UIColor whiteColor];
    [self.view addSubview:_headerImageView];

    _nickNameLabel = [[UILabel alloc] initWithFrame:CGRectMake((_width - fitScreenWidth(100)) / 2, CGRectGetMaxY(_headerImageView.frame), fitScreenWidth(100), fitScreenWidth(25))];
    _nickNameLabel.font = [UIFont systemFontOfSize:fitScreenWidth(12.0)];
    _nickNameLabel.hidden = YES;
    _nickNameLabel.textAlignment = NSTextAlignmentCenter;
    [self.view addSubview:_nickNameLabel];

    _tipLabel = [[UILabel alloc] initWithFrame:CGRectMake(20, CGRectGetMaxY(_nickNameLabel.frame), _width - 20 * 2, fitScreenWidth(25))];
    _tipLabel.font = [UIFont systemFontOfSize:fitScreenWidth(13.0)];
    _tipLabel.textAlignment = NSTextAlignmentCenter;
    _tipLabel.textColor = [UIColor lightGrayColor];
    [self.view addSubview:_tipLabel];


    _loginBtn = [[UIButton alloc] initWithFrame:CGRectMake((_width - fitScreenWidth(150)) / 2 , _height - fitScreenWidth(55), fitScreenWidth(150), 30)];
    _loginBtn.titleLabel.font = [UIFont systemFontOfSize:fitScreenWidth(13.0)];

    [_loginBtn setTitle:NSLocalizedString(@"Use login password", nil) forState:UIControlStateNormal];
    [_loginBtn setTitleColor:[UIColor hexString:@"#3fb8c8"] forState:UIControlStateNormal];
    [_loginBtn setHidden:YES];
    [_loginBtn addTarget:self action:@selector(loginWithPwd) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:_loginBtn];


    UIBarButtonItem *resetBtn = [[UIBarButtonItem alloc] initWithTitle:NSLocalizedString(@"Reset", nil) style:UIBarButtonItemStylePlain target:self action:@selector(resetBtnClick)];
    self.navigationItem.rightBarButtonItem = resetBtn;

    if (_unlockType == TouchUnlockCreatePwd) {
        //绘制
        _steps = 2;
        _tipLabel.text = NSLocalizedString(@"Draw Your Pattern", nil);
    }
    else if (_unlockType == TouchUnlockUpdatePwd){
        //更新密码不验证原密码 直接新增 !savePwd || savePwd.length == 0
        _unlockType = TouchUnlockCreatePwd;
        _tipLabel.text = NSLocalizedString(@"Draw new pattern", nil);
        _steps = 2;
    }
    else if (_unlockType == TouchUnlockValidatePwd){
        //验证密码
        _steps = 1;
        _errorCount = 5;
        [_loginBtn setHidden:NO];
        _tipLabel.text = NSLocalizedString(@"Draw Your Pattern", nil);
        _headerImageView.layer.masksToBounds = YES;
        _headerImageView.layer.cornerRadius = fitScreenWidth(50) / 2;
        [_headerImageView setNewImageUrl:APPDELEGATE.userModel.avatar placeHolder:[UIImage imageNamed:@"login_content_Avatar@2x"]];

        _nickNameLabel.hidden = NO;
        _nickNameLabel.text = APPDELEGATE.userModel.nickname;
    }

    if ([savePwd isEqualToString:@"TouchPwdError"]) {
        //密码失效，直接弹出提示框
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            [self alertPwdError];
        });
    }
}

- (void)changeUI:(UnlockType)type{
    //切换头部视图
}

- (void)changeTip:(NSInteger)index{
    //改变警告文字  1:最少四个点，请重新绘制  2:与上次绘制不一致，请重新绘制 3：密码错误还可以输入5次
    _tipLabel.textColor = [UIColor redColor];
    NSString *text = nil;
    switch (index) {
        case 1:
        {
            text = NSLocalizedString(@"At least 4 points required", nil);
        }
            break;

        case 2:
        {
            text = NSLocalizedString(@"Not match. Please draw it again", nil);

        }
            break;

        case 3:
        {
            text = _errorCount == 10000 ? [NSString stringWithFormat:NSLocalizedString(@"Invalid password. Please draw it again", nil)] : [NSLocalizedString(@"Invalid password.*** attempt(s) left", nil) stringByReplacingOccurrencesOfString:@"***" withString:[NSString stringWithFormat:@"%ld",_errorCount -- > 0 ? _errorCount : 0]];
        }
            break;


        default:
            break;
    }

    _tipLabel.text = text;

    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{

        if (_unlockType == TouchUnlockCreatePwd) {
            if (_steps == 2) {
                [self changeTipForNormal:1];
            }
            else if (_steps == 3){
                [self changeTipForNormal:3];
            }
            return;
        }
        [self changeTipForNormal:1];
    });
}

- (void)changeTipForNormal:(NSInteger)index{
    _tipLabel.textColor = [UIColor lightGrayColor];
    NSString *text = @"";
    switch (index) {
        case 1:
            text = NSLocalizedString(@"Draw Your Pattern", nil);
            break;
        case 2:
            text = @"请输入原手势密码";
            break;
        case 3:
            text = NSLocalizedString(@"Draw pattern again", nil);
            break;
        case 4:
            text = NSLocalizedString(@"Draw Your Pattern", nil);
            break;

        default:
            break;
    }
    _tipLabel.text = text;
}


- (UIImage *)drawline{

    if (_selectArray.count == 0) {
        return nil;
    }
    RoundRectView *ve = _selectArray.firstObject;
    CGPoint vePoint = ve.center;
    _startPoint = vePoint;

    UIImage *img = [[UIImage alloc] init];
    UIColor *color = [UIColor hexString:@"#3fb8c8"];

    UIGraphicsBeginImageContext(_imageView.frame.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
//    CGContextSetAllowsAntialiasing(context,true);
//    CGContextSetShouldAntialias(context, true);

    CGContextSetLineWidth(context, 2);
    CGContextSetStrokeColorWithColor(context, color.CGColor);
    CGContextMoveToPoint(context, _startPoint.x, _startPoint.y);


    if (_selectArray.count == 0) {
        return nil;
    }

    for (RoundRectView *ve in _selectArray) {
        CGPoint po = ve.center;
        CGContextAddLineToPoint(context, po.x, po.y);
        CGContextMoveToPoint(context, po.x, po.y);
    }

    CGContextAddLineToPoint(context, _endPoint.x, _endPoint.y);
    CGContextStrokePath(context);

    img = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();

    return img;
}


- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{

    UITouch *touch = [touches anyObject];
    if (touch) {
        for (RoundRectView *ve in _nomarlArray) {
            CGPoint point = [touch locationInView:ve];

            if ([ve pointInside:point withEvent:nil]) {
                [_selectArray addObject:ve];
                [self savePassWord:ve.tag];
                ve.selectView.hidden = NO;
                _startPoint = ve.center;
            }
        }
    }
}


- (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    UITouch *touch = [touches anyObject];
    if (touch) {
        _endPoint = [touch locationInView:_imageView];

        for (RoundRectView *ve in _nomarlArray) {
            CGPoint point = [touch locationInView:ve];

            if ([ve pointInside:point withEvent:nil]) {

                BOOL isAdd = YES;
                for (RoundRectView *vv in _selectArray) {
                    if (vv == ve) {
                        isAdd = NO;
                        break;
                    }
                }

                if (isAdd) {
                    [_selectArray addObject:ve];
                    [self savePassWord:ve.tag];
                    ve.selectView.hidden = NO;
                }
            }
        }
    }
    _imageView.image = [self drawline];
    _imageView.layer.contentsScale = [[UIScreen mainScreen] scale];

}


- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{

    UITouch *touch = [touches anyObject];
    if (touch) {
        for (RoundRectView *ve in _nomarlArray) {
            CGPoint po = [touch locationInView:ve];
            if (![ve pointInside:po withEvent:nil]) {
                RoundRectView *seVe = _selectArray.lastObject;
                _endPoint = seVe.center;
                _imageView.image = [self drawline];
            }
        }
    }
    [self cheakPassword];

    _password = [NSMutableString string];

    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{

        _imageView.image = nil;
        [_selectArray removeAllObjects];

        for (RoundRectView *ve in _nomarlArray) {
            ve.selectView.hidden = YES;
        }
    });
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
}


- (void)savePassWord:(NSInteger)tag{
    [_password appendFormat:@"%ld",tag];
}


- (void)cheakPassword{

    if (_password.length < 4) {
        [self changeTip:1];
        return; //密码长度不够
    }

    if (_unlockType == TouchUnlockCreatePwd) {

        if (_steps == 2) {
            _oldPassword = _password;
            [self changeTipForNormal:3];
            _steps = 3;
            return;
        }

        if (_steps == 3) {
            if (![_password isEqualToString:_oldPassword]) {
                [self changeTip:2];
                return; //前后不一致
            }

            if ([_oldPassword isEqualToString:_password]) {
                //两次密码一致 存本地
                NSUserDefaults *def = [NSUserDefaults standardUserDefaults];
                [def setObject:_password forKey:TouchUnlockPwdKey];
                [MessageCenter openAlertViewWithMessage:NSLocalizedString(@"Success", nil) duringtime:1];
                [self.navigationController popViewControllerAnimated:YES];
            }
        }
    }

    else if (_unlockType == TouchUnlockValidatePwd || _unlockType == TouchUnlockUpdatePwd){
        //获取保存的密码
        NSUserDefaults *def = [NSUserDefaults standardUserDefaults];
        NSString *savePwd = [def objectForKey:TouchUnlockPwdKey];

        if (![savePwd isEqualToString:_password] && _steps == 1) {
            //原密码错误
            _steps = 1;
            [self changeTip:3];

            if (_errorCount <= 0) {
                //错误次数太多
                [self alertPwdError];
                return;
            }

            return;
        }
        //登陆成功
        if (_unlockType == TouchUnlockValidatePwd) {

            _errorCount = 5;
//            [self dismissViewControllerAnimated:YES completion:nil];

            if (![APPDELEGATEREAL lockScreenWindow].hidden) {
                [APPDELEGATEREAL lockScreenWindow].hidden = YES;
            }
            return;
        }

        if (_steps == 1) {
            [self changeTipForNormal:1];
            _steps = 2;
            return;
        }

        if (_steps == 2) {
            _oldPassword = _password;
            [self changeTipForNormal:3];
            _steps = 3;
            return;
        }

        if (_steps == 3) {
            if (![_password isEqualToString:_oldPassword]) {
                 [self changeTip:2];
                return;
            }

            if ([_oldPassword isEqualToString:_password]) {
                //两次密码一致 存本地
                NSUserDefaults *def = [NSUserDefaults standardUserDefaults];
                [def setObject:_password forKey:TouchUnlockPwdKey];

                NSLog(@"密码更新成功:%@",_password);
                [MessageCenter openAlertViewWithMessage:NSLocalizedString(@"Success", nil) duringtime:1];
                dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                    [self.navigationController popViewControllerAnimated:YES];
                });
            }
        }

    }

}


- (void)alertPwdError{
    //手势密码失效
    [[NSUserDefaults standardUserDefaults] setObject:@"TouchPwdError" forKey:TouchUnlockPwdKey];

    UIAlertController *alert = [UIAlertController alertControllerWithTitle:NSLocalizedString(@"Pattern is expired", nil) message:NSLocalizedString(@"Please login with password", nil) preferredStyle:UIAlertControllerStyleAlert];

    NSMutableAttributedString *alertControllerMessageStr = [[NSMutableAttributedString alloc] initWithString:NSLocalizedString(@"Please login with password", nil)];
    [alertControllerMessageStr addAttribute:NSForegroundColorAttributeName value:[UIColor hexString:@"#999999"] range:NSMakeRange(0, alertControllerMessageStr.length)];
    [alertControllerMessageStr addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:12] range:NSMakeRange(0, alertControllerMessageStr.length)];
    [alert setValue:alertControllerMessageStr forKey:@"attributedMessage"];

    UIAlertAction *toAction = [UIAlertAction actionWithTitle:NSLocalizedString(@"Login again", nil) style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        [self loginWithPwd];
    }];
    [alert addAction:toAction];
    [self presentViewController:alert animated:YES completion:nil];
}


- (void)loginWithPwd{
    LX_LoginAndForgetViewController *loginVc = [[LX_LoginAndForgetViewController alloc] init];
    loginVc.isTouchUnlockIn = YES;
    [self presentViewController:loginVc animated:YES completion:nil];
}

//重设
- (void)resetBtnClick{
    _oldPassword = nil;
    _steps = 2;
    [self changeTipForNormal:1];
}

@end
```
5、AppDelegate中新增UIwindow属性用于遮罩

```
//didFinishLaunchingWithOptions方法加入
self.lockScreenWindow = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
 self.lockScreenWindow.windowLevel= UIWindowLevelAlert+1;//最高级等级窗口
 self.lockScreenWindow.backgroundColor = [UIColor whiteColor];
 TouchUpViewController *lockvc  =[[TouchUpViewController alloc]initWithUnlockType:TouchUnlockValidatePwd];
 self.lockScreenWindow.rootViewController = lockvc;
 self.lockScreenWindow.hidden = YES;
```
在合适的时候对lockScreenWindow.hidden属性进行操作。


通过上述两个类，初步建立了手势密码的逻辑体系。

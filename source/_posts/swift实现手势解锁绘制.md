---
title: swift实现手势解锁绘制
date: 2017-06-05 17:52:30
tags:
  - 滑动锁屏
  - 绘制按钮
categories: 技术分享
---

使用swift实现iOS手势锁屏，虽然在iOS客户端很少使用到滑动手势，但是有时候为了和安卓应用保持用户交互的一致性，所以有的时候还是很有必要的。
iOS客户端解锁建议使用touch ID。

swift与object-c的CGContextRef不一样，在swift中统一使用CGContext进行管理和使用。

本示例采用9*button进行绘制，关闭button交互事件，通过touchesBegan系列方法对滑动路径进行跟踪和绘制（imageView）。

以下为全部代码：
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
